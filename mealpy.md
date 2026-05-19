### `mealpy` Variable Types 

| Variable Type | Nature of Bounds | Required / Core Arguments | Example |
| :--- | :--- | :--- | :--- |
| **`FloatVar`** | Continuous floating-point numbers between a minimum and maximum limit. | **`lb`**: Lower bounds (list/array).<br>**`ub`**: Upper bounds (list/array).<br>**`name`**: (Optional) String name. | `FloatVar(lb=[-5., -5.], ub=[5., 5.])` |
| **`IntegerVar`** | Discrete whole numbers between a minimum and maximum limit. | **`lb`**: Lower bounds (list/array of ints).<br>**`ub`**: Upper bounds (list/array of ints).<br>**`name`**: (Optional) String name. | `IntegerVar(lb=[0, 0], ub=[10, 10])` |
| **`BinaryVar`** | Strictly discrete numbers constrained to only `0` or `1`. | **`n_vars`**: Integer specifying how many variables are in the array.<br>**`name`**: (Optional) String name. | `BinaryVar(n_vars=10)` |
| **`BoolVar`** | Strictly discrete boolean logic constrained to `True` or `False`. | **`n_vars`**: Integer specifying how many variables are in the array.<br>**`name`**: (Optional) String name. | `BoolVar(n_vars=5)` |
| **`PermutationVar`** | A fixed set of unique items that the algorithm rearranges into different sequences (orders). | **`valid_set`**: A single list or tuple of the items to be shuffled.<br>**`name`**: (Optional) String name. | `PermutationVar(valid_set=[1, 2, 3, 4])` |
| **`StringVar`** | Categorical variables restricted to specific string values. | **`valid_sets`**: A list of lists/tuples, defining the allowable strings for each variable position.<br>**`name`**: (Optional) String name. | `StringVar(valid_sets=[("red", "blue"), ("A", "B")])` |
| **`MixedSetVar`** | Custom discrete choices that can mix integers, floats, strings, etc. | **`valid_sets`**: A list of lists/tuples, defining the exact allowed choices for each variable.<br>**`name`**: (Optional) String name. | `MixedSetVar(valid_sets=[(1.5, 3.2), ("yes", "no")])` |

---

### Important Notes on Usage

* **Dimensions:** For `FloatVar` and `IntegerVar`, the length of the list you pass to `lb` and `ub` dictates the dimensions of your problem. If you pass a list of 5 elements to `lb` and 5 elements to `ub`, the algorithm knows `n_vars = 5`.
* **Mixing:** You can group different types together by passing a standard Python list of these objects to the `bounds` key in your problem dictionary (e.g., `"bounds": [FloatVar(...), IntegerVar(...)]`).
In `mealpy`, no matter how complex your bounds are, the `solution` parameter passed into your objective function is always a **1D numpy array**. The algorithm stitches all your variables together in the exact order you defined them.
---
To access your variables, you simply use standard Python array **indexing** and **slicing**.

### 1. Accessing a Single Variable Type

If you defined a single variable type (for example, a `FloatVar` of 3 elements), the `solution` array simply contains those 3 elements. You can access them by their index or unpack them directly.

```python
from mealpy import FloatVar

def simple_objective(solution):
    # Method A: Direct Indexing
    x1 = solution[0]
    x2 = solution[1]
    x3 = solution[2]
    
    # Method B: Python Unpacking (cleaner for small arrays)
    x1, x2, x3 = solution
    
    return (x1 ** 2) + (x2 ** 2) + (x3 ** 2)

problem = {
    "obj_func": simple_objective,
    "bounds": FloatVar(lb=[-5, -5, -5], ub=[5, 5, 5]),
    "minmax": "min"
}

```

### 2. Accessing Mixed Variable Types

When you pass a list of different variable types to your bounds, `mealpy` concatenates them. You must use **array slicing** based on the sizes of the variables to extract them.

Let's say you have a problem with **3 binary variables** followed by **2 floating-point variables**. The `solution` array will have exactly 5 elements.

```python
import numpy as np
from mealpy import BinaryVar, FloatVar

def mixed_objective(solution):
    # 1. Slice the first 3 elements for the binary array
    # solution[0:3] grabs index 0, 1, and 2
    binary_part = solution[0:3] 
    
    # 2. Slice the remaining 2 elements for the float array
    # solution[3:5] grabs index 3 and 4
    float_part = solution[3:5]  
    
    # Example logic using the extracted arrays
    if np.sum(binary_part) > 1:
        return np.sum(float_part ** 2)
    else:
        return np.sum(float_part)

# Define the mixed bounds
my_bounds = [
    BinaryVar(n_vars=3),                         # Takes indices 0, 1, 2
    FloatVar(lb=[-10.0, -10.0], ub=[10.0, 10.0]) # Takes indices 3, 4
]

problem = {
    "obj_func": mixed_objective,
    "bounds": my_bounds,
    "minmax": "min"
}

```

### 3. Accessing Values in a 2D Matrix (Flattened)

If you created a 1D array to represent a 2D matrix (as discussed previously), the `solution` array contains the flattened items. You reshape it first, and then access it using row/column coordinates.

```python
from mealpy import FloatVar
import numpy as np

def matrix_objective(solution):
    # Reshape the flat 1D array back into 3x3
    matrix = solution.reshape((3, 3))
    
    # Access specific variables using 2D coordinates [row, column]
    top_left_value = matrix[0, 0]
    center_value = matrix[1, 1]
    bottom_right_value = matrix[2, 2]
    
    return top_left_value + center_value + bottom_right_value

# 3x3 matrix = 9 elements total
problem = {
    "obj_func": matrix_objective,
    "bounds": FloatVar(lb=[0]*9, ub=[10]*9),
    "minmax": "min"
}

```
---
In `mealpy`, the **Genetic Algorithm (GA)** is housed under the `evolutionary_based` module. It is a classic optimization algorithm inspired by the process of natural selection, utilizing mechanisms such as reproduction, crossover, and mutation to evolve a population of candidate solutions toward a global optimum.

Here is a comprehensive summary of how GA is structured in `mealpy`, its parameters, and how to use it.

### 1. Importing the Algorithm

Depending on the version of `mealpy` you are using (especially versions 3.0.0 and newer), the primary class for the standard Genetic Algorithm is usually `OriginalGA`.

```python
from mealpy.evolutionary_based import GA
# The model is accessed via GA.OriginalGA

```

### 2. Core Parameters (Hyper-parameters)

When initializing the GA model, you pass in algorithm-specific hyper-parameters. Here are the core arguments you can tune:

| Parameter | Type | Default | Description | Recommended Range |
| --- | --- | --- | --- | --- |
| **`epoch`** | `int` | `10000` | The maximum number of iterations (or generations) the algorithm will run. | `[100, 10000]` |
| **`pop_size`** | `int` | `100` | The number of candidate solutions (agents) in the population. | `[20, 200]` |
| **`pc`** | `float` | `0.95` | **Crossover Probability:** The likelihood that two parents will swap parts of their solutions to create offspring. | `[0.7, 0.95]` |
| **`pm`** | `float` | `0.025` | **Mutation Probability:** The chance that an offspring will randomly alter a part of its solution (helps avoid getting stuck in local minima). | `[0.01, 0.2]` |

### 3. Advanced Parameters (Strategy Selection)

Under the hood, `mealpy`'s GA allows you to customize the exact evolutionary operators via `kwargs`. If you don't define these, it defaults to standard strategies.

* **`selection` (`str`):** Determines how parents are chosen. Defaults to `"tournament"` (where random agents compete and the best is selected). You can also set it to `"roulette"` or `"random"`.
* **`crossover` (`str`):** Determines how genes are mixed. Defaults to `"uniform"`.
* **`mutation` (`str`):** Determines how genes are randomly altered. Defaults to `"flip"`.

---

### 4. Code Example: Putting it all together

Here is how you initialize the GA with its parameters and solve a problem using the variable bounds we discussed earlier:

```python
import numpy as np
from mealpy import FloatVar
from mealpy.evolutionary_based import GA

# 1. Define the objective function
def sphere_function(solution):
    return np.sum(solution**2)

# 2. Define the problem dictionary
problem_dict = {
    "obj_func": sphere_function,
    "bounds": FloatVar(lb=[-10.0] * 5, ub=[10.0] * 5),
    "minmax": "min"
}

# 3. Initialize the Genetic Algorithm with custom parameters
model = GA.OriginalGA(
    epoch=100,          # Stop after 100 generations
    pop_size=50,        # 50 solutions exploring the space
    pc=0.85,            # 85% chance of crossover
    pm=0.05,            # 5% chance of mutation
    selection="tournament" # Use tournament parent selection
)

# 4. Solve the problem
model.solve(problem_dict)

# 5. Extract results
print(f"Best Fitness Value: {model.g_best.target.fitness}")
print(f"Best Position (Solution): {model.g_best.solution}")

```
