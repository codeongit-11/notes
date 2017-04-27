
# Algorithm On Graphs

## Week1

### Representing Graphs

 - Edge List
    - egdes: (A, B), (A, C), (A,D), (C,D) 
    - A,B,C,D are vertices
 - Adjacency Matrix
    - Entries 1 if there is an edge, 0 if there is not.
 - Adjacency List
    - For each vertex, a list of adjacent vertices.
    - A adjacent to B, C,D
    - B adjacent to A
    - C adjacent to A,D
    - D adjacent to A, C
    - can be implemented by a dictionary 
 
 - Defferent operations are faster in defferent representations
 - for many problems , want adjacency list .

Op | Is Edge? | List Edge | List Nbrs 
--- | --- | --- | ---
Adj. Matrix | Θ(1) |  Θ( &#124;V&#124;²) |  Θ(&#124;V&#124;)
Edge List | Θ(&#124;E&#124;) | Θ(&#124;E&#124;) |  Θ(&#124;E&#124;)
Adj. List | Θ(deg) | Θ(&#124;E&#124;) | Θ(deg)


### Exploring Graphs

 - Visit Markers
    - To keep track of vertices found: Give each vertex boolean visited(v).
 - Unprocessed Vertices
    - Keep a list of vertices with edges left to check.
    - This will end up getting hidden in the program stack
 - Depth First Ordering
    - We will explore new edges in Depth First order. 

#### Explore , starting from a Node

 - Need adjacency list representation!

```python
def Explore(v):
    visited(v) ← true
    for (v, w) ∈ E:
        if not visited(w):
            Explore(w)
```

#### DFS

```python
def DFS(G):
    for all v ∈ V : mark v unvisited
    for v ∈ V :
        if not visited(v):
            Explore(v)
```

---

### Connectivity

#### Connected Components

 - Explore(v) finds the connected component of v
 - Just need to repeat to find other components.
 - Modify DFS to do this.
 - Modify goal to label connected components

```python
def Explore(v):
    visited(v) ← true
    CCnum(v) ← cc
    for (v, w) ∈ E:
        if not visited(w):
            Explore(w)

def DFS(G):
    for all v ∈ V mark v unvisited
    cc ← 1
    for v ∈ V :
        if not visited(v):
            Explore(v)
            cc ← cc + 1
```

---

### Previsit and Postvisit Orderings

 - Need to Record Data
    - Plain DFS just marks all vertices as visited.
    - Need to keep track of other data to be useful.
    - Augment functions to store additional information

#### Previsit and Postvisit Functions


```python
def Explore(v):
    visited(v) ← true
    previsit(v)
    for (v, w) ∈ E:
        if not visited(w):
            Explore(w)
    postvisit(v)
```

 - Clock
    - Keep track of order of visits.
    - Clock ticks at each pre-/post- visit.
    - Records previsit and postvisit times for each v.

![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/algr_on_graph_previst_clock_example.png)

 - Computing Pre- and Post- Numbers
 - Initialize clock to 1.

```python
def previsit(v):
    pre(v) ← clock
    clock ← clock + 1
def postvisit(v):
    post(v) ← clock
    clock ← clock + 1
```

 - Previsit and Postvisit numbers tell us about the execution of DFS.
 - Lemma
    - For any vertices u, v the intervals pre(u), post(u)] and [pre(v), post(v)] are either **nested** or **disjoint**.
        - nested: eg.  (1,8) , (2,5)
        - disjoint: eg.  (1,8) , (9,12)
    - that is , Interleaved (not possible) 
        - eg. ( 1, 8 ) , ( 5, 9  )

---

