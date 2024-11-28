
# CTF Writeup: "Uncovering the Hidden Message Using Multidimensional Scaling"

## Challenge Overview
- **Category**: Miscellaneous (Misc)
- **Challenge Name**: Uncovering the Hidden Message Using Multidimensional Scaling
- **Points**: 500
- **Difficulty**: Medium
- **Creator**: SKRCTF
- **Description**:  
  In this challenge, we were provided with a distance matrix stored in a `matrix.npy` file. Our goal was to use Multidimensional Scaling (MDS) to reconstruct the original coordinates from the distance matrix. By plotting these coordinates, we would reveal a hidden message — the flag.

---

## Understanding the Problem

### Distance Matrix
- A **distance matrix** is a square matrix that contains the distances between each pair of points in a set. 
- It's symmetric, and the diagonal elements are zero (distance from a point to itself).

### Multidimensional Scaling (MDS)
- **MDS** is a statistical technique for visualizing the level of similarity (or dissimilarity) between individual cases of a dataset. 
- It tries to place each object in N-dimensional space such that the between-object distances are preserved as well as possible.
- We applied MDS to reconstruct the coordinates from the provided distance matrix and then visualized the resulting points to uncover the hidden message.

---

## Solution Approach

### 1. Load the Distance Matrix
We begin by loading the distance matrix from the provided `.npy` file using **NumPy**.

### 2. Apply Multidimensional Scaling (MDS)
We use the **MDS** algorithm from **scikit-learn** to reconstruct the coordinates from the distance matrix.

### 3. Visualize the Coordinates
Once we have the coordinates, we plot them using **Matplotlib** to see if they form any recognizable pattern or message.

### 4. Extract the Flag
From the visualization, we read the hidden message formed by the plotted points, which is the flag.

---

## Step-by-Step Solution

### Step 1: Load the Distance Matrix
First, we load the distance matrix from the provided `matrix.npy` file:

```python
import numpy as np

# Load the distance matrix
distance_matrix = np.load('matrix.npy')
```

**Explanation**:  
- We use `np.load()` to load the NumPy array from the file, which contains the pairwise distances between the points.

---

### Step 2: Apply Multidimensional Scaling (MDS)
Next, we use the MDS algorithm to reconstruct the coordinates. Since we have a precomputed distance matrix, we set `dissimilarity='precomputed'`.

```python
from sklearn.manifold import MDS

# Initialize MDS with dissimilarity='precomputed' because we have a distance matrix
mds = MDS(n_components=2, dissimilarity='precomputed', random_state=42)

# Fit the model and transform the distance matrix into coordinates
coords = mds.fit_transform(distance_matrix)
```

**Explanation**:  
- We use `MDS(n_components=2)` to reduce the data to 2 dimensions, which is ideal for visualizing the points.
- The `random_state=42` ensures that the results are reproducible.
- The `fit_transform()` method computes the positions of the points in 2D space.

---

### Step 3: Visualize the Coordinates
Now, we plot the reconstructed coordinates to check for any recognizable patterns or hidden messages.

```python
import matplotlib.pyplot as plt

# Create a scatter plot of the coordinates
plt.figure(figsize=(8, 8))
plt.scatter(coords[:, 0], coords[:, 1], s=5)

# Invert the y-axis if necessary for correct orientation
plt.gca().invert_yaxis()

# Remove axes for a cleaner look
plt.axis('off')

# Display the plot
plt.show()
```

**Explanation**:  
- We create a scatter plot of the 2D coordinates using `plt.scatter()`.
- The `invert_yaxis()` ensures that the orientation is correct.
- `plt.axis('off')` removes the axis ticks and labels for a cleaner visualization.
- `plt.show()` displays the plot.

---

### Visualization Result:
Upon executing the code, a scatter plot appears that forms the text:  
**`SHCTF24{Intr0_t0_ML}`**

This is the hidden flag encoded in the plot.

---

### Step 4: Extract the Flag
From the visualization, we can now read the hidden message:  
**Flag**: `SHCTF24{Intr0_t0_ML}`
![image](https://github.com/user-attachments/assets/65e3c799-0fe3-438a-9877-400f8e92abd9)

![image](https://github.com/user-attachments/assets/4530cc49-912d-42a0-ab81-02ecc70f8a2e)


---

## Understanding Why This Works

### MDS Reconstruction
- The **distance matrix** represents the pairwise distances between points that form the hidden message when plotted in 2D space.
- By applying MDS, we effectively reverse-engineer the spatial arrangement of these points.

### Reverse Engineering
- The distance matrix contains information about how far apart the points are from each other. By applying MDS, we reconstruct their positions in a 2D space that maintains the relative distances between them.

### Visualization
- Plotting these points in 2D space reveals the structure intended by the challenge creators—the hidden flag is displayed in the form of a message.

---

## Complete Code
Here's the complete script that accomplishes all the steps:

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.manifold import MDS

# Step 1: Load the distance matrix
distance_matrix = np.load('matrix.npy')

# Step 2: Apply MDS
mds = MDS(n_components=2, dissimilarity='precomputed', random_state=42)
coords = mds.fit_transform(distance_matrix)

# Step 3: Plot the coordinates
plt.figure(figsize=(8, 8))
plt.scatter(coords[:, 0], coords[:, 1], s=5)
plt.gca().invert_yaxis()
plt.axis('off')
plt.show()
```

---

## Additional Notes

- **Random State**: Setting the `random_state` ensures that the MDS algorithm produces the same results each time, which is important for reproducibility.
- **Dissimilarity**: Since we're working with a precomputed distance matrix, it's crucial to set `dissimilarity='precomputed'` in the MDS model.
- **Orientation**: If the message appears upside down or mirrored, you may need to adjust the axes. Inverting the y-axis often corrects the orientation.

---

## Conclusion
By applying **Multidimensional Scaling** to the provided distance matrix, we successfully reconstructed the original coordinates. Visualizing these coordinates revealed the hidden message — the flag:  
**`SHCTF24{Intr0_t0_ML}`**

This challenge demonstrates how **data visualization** and **dimensionality reduction** techniques can be used to uncover hidden information embedded within datasets.

---

## References

- **Multidimensional Scaling (MDS)**:  
  [Scikit-learn MDS Documentation](https://scikit-learn.org/stable/modules/generated/sklearn.manifold.MDS.html)
  
- **Matplotlib for Visualization**:  
  [Matplotlib Scatter Plot Documentation](https://matplotlib.org/stable/api/pyplot_api.html#matplotlib.pyplot.scatter)

