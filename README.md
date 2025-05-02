# Percolation
![pexels-sanlad-11894781](https://github.com/user-attachments/assets/38155db2-067f-48e5-9442-bdf73333648a)
Photo by Tolga deniz Aran: https://www.pexels.com/photo/barista-preparing-coffee-11894781/

**Percolation** refers to the movement of water through soil or a coffee filter. 
In our university project, we simulate percolation using a grid of 0s and 1s.

This project includes:
- A matrix generator with controllable probability
- A DFS-based algorithm to detect connected clusters
- A function to detect whether a cluster spans the grid (i.e., a “path” exists)
- Building a randomized simulation after the basic structure

## `generate_matrix(n, prob=0.5)`
This function creates a new (n+2) x (n+2) grid filled with 0s and 1s.

### parameters: 
- `n`: The size of the grid (excluding borders).
- `prob`: The probability that each cell (excluding borders) is set to `1`.

### Why n+2?
```python
size = n+2
matrix = np.random.choice([0, 1], size=(size, size), p=[1-prob, prob])
```
The grid size is n+2 because we want to ignore edge case, which affect our later program `dfs()` function. If we don't create n+2×n+2 matrix, we have to consider the special case of edge, for instance about left edge, we have to ignore (x-1,y) cases. In this way, if we set n+2×n+2 grid, we can ignore special cases and less coding. We get such a grid below after implementing `generate_matrix()`.

Example output (for `n=5`):
```bin
[[0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0]
 [0 0 1 0 1 0 0]
 [0 0 0 1 1 0 0]
 [0 0 1 0 1 1 0]
 [0 0 0 0 1 0 0]
 [0 0 0 0 0 0 0]]
```
## `find_clusters(matrix)`
This function uses **Depth-First Search (DFS)** to find all clusters of connected 1s in the matrix.

### What is **Depth-First Search (DFS)**?
>Depth-first search (DFS) is an algorithm for traversing or searching tree or graph data structures. The algorithm starts at the root node (selecting some arbitrary node as the root node in the case of a graph) and explores as far as possible along each branch before backtracking. Extra memory, usually a stack, is needed to keep track of the nodes discovered so far along a specified branch which helps in backtracking of the graph.[^1]

### How it works:
* Only the internal (n x n) region is checked, skipping the 0-padded edges.
* Each unvisited 1-cell becomes a starting point for a DFS.
* A cluster is a group of 1s that are connected in any 4 directions (up, down, left, right).
### Key code
```python
for y in range(1,n-1):
  for x in range(1,n-1):
    current_cluster = []
    dfs(y,x,current_cluster)
      if current_cluster:
        clusters.append(current_cluster)
```

### parameters:
* `n`: the size of matrix, n dimension
* `visited`: to check if the cell has already been visited. We will add `x` and `y` coordinate after running dfs function.
* `clusters`: the list of all possible clusters.
* `directions`: it express the direction of each cell. We say it as distance of the cell between current cell and neighbors as well.

We want to consider every cells without edge cells, so the range of for loops is (1,n-1). The list of `current_cluster` is for saving a piece of cluster for each cell. If the cell have a lot of connection in its neighbors, the `current_cluster` will be bigger, while if the cell is 0 or has already been visited, the `current_cluster` is empty (That's why I added if sentence to ignore empty lists). To find all connections, we made `dfs()` function. Also, we executed it here because we want to consider the each case of the cell. 

### `dfs(y,x,current_cluster)`
This function is based on depth-first search method. I inspired from this site.
[Depth First Search or DFS for a Graph](https://www.geeksforgeeks.org/depth-first-search-or-dfs-for-a-graph/)

```python
if (y,x) in visited or matrix[y][x] == 0:
  return
```
Skip the cell if it's either already visited or it's 0.
```python
visited.add((y,x))
current_cluster.append((y,x))
```
Mark the cell as visited and add it to the current cluster.
```python
for dy,dx in directions:
    if matrix[y+dy][x+dx] == 1: 
        dfs(y+dy,x+dx,current_cluster)
```
This checks the 4-directional neighbors:
* For example, if (y, x) = (1, 2), it checks: (0,2), (2,2), (1,1), (1,3).
* Also, we will check if each cell is 1 or not, and it will continue until the edge of the cluster by calling `dfs()` function because I assigned the parameters are for neighbor.

## `find_path(matrix, clusters)`
From `find_cluster()`, we got the list of possible paths(clusters).
So this function checks whether any cluster **spans the grid** — either horizontally or vertically.
### Logic:
- For each cluster, extract all x and y coordinates.
- If all values from 1 to n are present in **either** x-axis or y-axis, the cluster spans the grid.

```python
expected = set(i for i in range(1, len(matrix)-1)) 
x_axis = set()  
y_axis = set()
for x, y in cluster:
    if x not in x_axis:
        x_axis.add(x)
    if y not in y_axis:
        y_axis.add(y)
if x_axis == expected or y_axis == expected:
    return True
```

[^1]: [Depth-first search in Wikipedia](https://en.wikipedia.org/wiki/Depth-first_search)
