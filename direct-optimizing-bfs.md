# Scott Beamer Paper - Direction-Optimizing BFS

## Top down algorithm -

### English Description -

Given a graph G with E edges and V vertices, find the minimum number of hops required to reach all vertices from a source node s, and record the parent of each vertex to form a BFS tree.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

## Bottom up algorithm -

### English Description -

Given a graph G with E edges and V vertices, find the minimum number of hops required to reach all vertices from a source node s, and record the parent of each vertex to form a BFS tree.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

## Hybrid -

### English Description -

Given a graph G with E edges and V vertices, find the minimum number of hops required to reach all vertices from a source node s, and record the parent of each vertex to form a BFS tree.
This algorithm dynamically switches between the top-down and bottom-up approaches — using top-down when the frontier is small and bottom-up when the frontier becomes large.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

# Top Down

## Algorithm (Step-by-Step EDGE)

Steps -

1. Initialize the frontier with the source node s.
2. Find the neighbors of each node in the frontier.
3. If a neighbor’s parent is not yet recorded, assign the current node as its parent.
4. Add all newly discovered neighbors to the frontier.
5. Repeat from Step 2 while the frontier is not empty.

Tensors -

$$
&\triangleright \text{Tensors} \\
G^{S≡|V|,D≡|V|} &→ \text{Boolean}, \text{empty}=false\\
F^{I, S≡|V|} &→\text{Boolean}, \text{empty}=false\\
Tree^{I, S≡|V|, D≡|V|} &→\text{Boolean}, \text{empty}=false \\
$$


1. Create a 2D tensor G with ranks S and D, where S represents the source and D represents destination. The initial values are either false if s is not connected to d by an edge and true if s is connected to d by an edge.
    - In EDGE: $G^{S,D} → \text{boolean}, \text{empty}= false$
2. To keep track of the active vertices, another data structure is required, called Frontier F, where each element in F stores either true or false. We create a Tensor F which stores 1 if the corresponding vertex is active. To keep the track for every iteration, we add another dimension i. Thus F now stores true if the corresponding vertex is active at iteration i.
    - In EDGE: $F^{I, S} → \text{boolean}, \text{empty}=false$
3. Initialize the frontier with the source node n by setting its entry to true for iteration 0.
    - $F_{0, n} = true$
4. To find the neighbors of each node in the frontier:
    - $T_{i, d} = G_{s, d} ⋅ F_{i, s} :: ⋀ AND (∩) ⋁ OR(∪)$
5. Initialize the Tree tensor for the source node such that the source node is its own parent. This prevents infinite loops in case the source has a self-edge.
    - $Tree_{0, n, n} = True$
6. To check which vertices have parents (i.e., have been visited), reduce the Tree tensor over the parent rank p. A given P-fiber (that is, fix "c" to some coordinate) represents the set of parents for that "c" node. We use the OR operator to reduce over the $P$ rank, which returns true if at least ONE parent exists for "c". Thus we use an OR operator here.
    - $HP_{i,c} = Tree_{i, p, c} :: ⋁ OR(∪)$

7. To identify neighbors without parents, take the complement of tensor HP. HP marks vertices that already have parents in the tree; its complement gives the set of unvisited nodes. Intersecting this with T yields NP, the set of neighbors of frontier vertices that are not yet visited. Then we intersect this tensor with T which has given us the neighbors of nodes in Frontier. This returns $NP$, the set of nodes that are neighbors of vertices in the frontier **and** do not have parents.
    - $NP_{i,d} = T_{i,d} ⋅ ¬ HP_{i,d} :: ⋀ AND(∩)$

8. Add the neighbors without parents (unvisited nodes) to the new frontier.
    - $F_{i+1, d} = NP_{i, d}$
9. Update the Tree to mark visited neighbors. Create a temporary tree by intersecting the Graph (to confirm the parent–child edge), the Frontier (active vertices), and NP (newly discovered children).
  - $TTree_{i,s,d} = (G_{s,d} ⋅ F_{i,s})_{i,s,d} ⋅ NP_{i,d} :: \bigwedge AND (∩) ⋀ AND(∩)$
10. If multiple parents discover the same child, select only one as the parent. Use the populate operator with a defined coordinate operator pick-parent to enforce this. To achieve this, we use a 'populate' operator to populate another temporary tensor by applying a defined coordinate operator 'pick-parent'    
  - $Temp_{i, s*, d} = TTree_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent})$
  
11. Merge the temporary tree with the existing Tree to preserve all previously visited nodes.
    - $Tree_{i+1,s,d} = Tree_{i,s,d} ⋅ TTree_{i,s,d} :: ⋀ OR$
12. When the newly created frontier becomes empty, stop the recursion.
    - $◇ : ||F_{i+1}|| ≡ false$



# Bottom up

## Steps -

1. Initialize the frontier with the source node.
2. Add the source to the tree as its own parent.
3. Check which nodes already have parents (to find unvisited nodes).
4. Find the neighbors of all unvisited nodes.
5. If any neighbor is in the frontier, add that node to the new frontier.

## Algorithm (Step-by-Step-EDGE) -

$$
→Tensors \\
G^{S,D} → \text{boolean}, \text{empty}=false\\
F^{I, S} → \text{boolean}, \text{empty}=false\\
Tree^{I, S, D} →\text{boolean}, \text{empty}=false \\
NNP^{I, D, S} →\text{boolean}, \text{empty}=false \\
InF^{I, D, S} →\text{boolean}, \text{empty}=false \\
$$


1. Create a 2D tensor G with ranks S and D, where S represents source and D represents destination. The initial values are either false if s is not connected to d by an edge and true if s is connected to d by an edge.
    - In EDGE: $G^{S,D} → \text{boolean}, \text{empty}=false$
2. To keep track of active vertices, create another data structure called Frontier F, where each element stores true if the vertex is active and false otherwise. We create a Tensor F which stores True if the corresponding vertex is active. To keep the track for every iteration, we add another dimension i. Thus F now stores True if the corresponding vertex is active at iteration i (and False otherwise).
  - In EDGE: $F^{I, S} → \text{boolean}, \text{empty}=false$
3. Initialize the frontier with the source node n by setting its value to true at iteration 0.
    - $F_{0, n} = true$
4. Initialize the Tree tensor so that the source node is its own parent. This prevents infinite loops if the source has a self-edge.
    - $Tree_{0, n, n} = True$
5. To check which vertices have parents (i.e., have been visited), reduce the Tree tensor over the parent rank p. This returns a tensor where each entry is true if any parent exists for that vertex. One rank is iteration i and another is 'c' giving us all the nodes that have true values if any of the value in parent rank 'p' was true. Thus we use an OR operator here.
    - $HP_{i,c}= Tree_{𝑖,𝑝,𝑐}:: \bigvee OR(\cup)$
6. Identify nodes without parents by taking the complement of HP, which gives all unvisited nodes.
    - $NP_{i, c} = ¬HP_{i, c}$
7. Find the neighbors (potential children) of nodes that do not have parents.
For each unvisited child d, check all possible parent neighbors s.
Create a 3D tensor NNP with ranks I, D, S, where:
– I represents the iteration, D represents the child vertices, and S represents the parent vertices. Each entry in NNP is true if, at iteration i, child d is unvisited and an edge exists from s to d.
    - $NNP_{i,d,s} = G_{s,d} ⋅ NP_{i,d} :: ⋀ AND$
8. Check if any parents are currently in the frontier.
For each unvisited child d, determine whether any of its neighbors s are active in the frontier. Create a tensor InF with ranks I, D, S where each entry is true if, at iteration i, an edge exists from s to d, d is unvisited, and s is in the frontier.
    - $InF_{i,d,s} = NNP_{i,d,s} ⋅ F_{i,s} :: ⋀ AND(∩)$
9. If multiple parents can reach the same child, select only one parent.
Use the populate operator with a defined coordinate operator pick-parent to enforce this. To achieve this, we use a 'populate' operator to populate another temporary tensor by applying a defined coordinate operator 'pick-parent'
    - $Temp_{i, d, s*} = InF_{i,d,s} \lll_{s*} \mathbb{1}(\text{pick-parent})$
10. Update the Tree to record all newly visited nodes discovered in this iteration.
    - $Tree_{i+1,d,s} = Temp_{i,d,s} ⋅ Tree_{i,d,s} :: ⋀ OR(∪)$
11. Update the frontier for the next iteration with nodes that were visited in the current step.
    - $F_{i+1, d} = InF_{i,d,s} :: \bigvee OR(∪)$
12. When the new frontier becomes empty, stop the recursion.
    - $◇ : ||F_{i+1} || ≡ false$

***

# Full EDGE 

## Top Down


$$
&\triangleright \text{Tensors} \\
G^{S≡|V|,D≡|V|} &→ \text{Boolean}, \text{empty}=false\\
F^{I, S≡|V|} &→\text{Boolean}, \text{empty}=false\\
Tree^{I, S≡|V|, D≡|V|} &→\text{Boolean}, \text{empty}=false \\
\\
&\triangleright \text{Initializations} \\
F_{i,s} &= false \\
Tree_{i,s,d} &= false \\
G_{s,d}&→<user-specified> \\
F_{0, n} &= true \\
Tree_{0, n, n} &= true \\
\\
&\triangleright \text{Extended Einsums} \\
HP_{i,c} &= Tree_{i, p, c} :: ⋁ OR(∪) \\
NP_{i,d} &= T_{i,d} ⋅ ¬ HP_{i,d} :: ⋀ AND(∩)\\
T_{i, d} &= G_{s, d} ⋅ F_{i, s} :: ⋀ AND (∩) ⋁ OR(∪) \\
F_{i+1, d} &= NP_{i, d} \\
TTree_{i,s,d} &= (G_{s,d} ⋅ F_{i,s})_{i,s,d} ⋅ NP_{i,d} :: \bigwedge AND (∩) ⋀ AND(∩) \\
Temp_{i, s*, d} &= TTree_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent})\\
Tree_{i+1,s,d} &= Tree_{i,s,d} ⋅ Temp_{i,s,d} :: ⋀ OR(∪) \\
◇ : ||F_{i+1}|| &≡ false
$$


## Bottom UP


$$
\triangleright \text{Tensors} \\
G^{S≡|V|,D≡|V|} → \text{Boolean}, \text{empty}=false\\
F^{I, S≡|V|} →\text{Boolean}, \text{empty}=false\\
Tree^{I, S≡|V|, D≡|V|} →\text{Boolean}, \text{empty}=false \\
NNP^{I, D, S} →\text{boolean}, \text{empty}=false \\
InF^{I, D, S} →\text{boolean}, \text{empty}=false \\
\\
\triangleright \text{Initializations} \\
F_{i,s} = false \\
Tree_{i,s,d} = false \\
G_{s,d} → <user-specified> \\
NNN_{i,d,s} = false\\
InF_{i,d,s} = false\\
F_{0, n} = true \\
Tree_{0, n, n} = true \\
\\
\triangleright \text{Extended Einsums} \\
HP_{i,c}= Tree_{𝑖,𝑝,𝑐}:: \bigvee OR(\cup) \\
NP_{i, c} = ¬HP_{i, c} \\
NNP_{i,d,s} = G_{s,d} ⋅ NP_{i,d} :: ⋀ AND\\
InF_{i,d,s} = NNP_{i,d,s} ⋅ F_{i,s} :: ⋀ AND(∩)\\
Temp_{i, d, s*} = InF_{i,d,s} \lll_{s*} \mathbb{1}(\text{pick-parent})\\
Tree_{i+1,d,s} = Temp_{i,d,s} ⋅ Tree_{i,d,s} :: ⋀ OR(∪)\\
F_{i+1, d} = InF_{i,d,s} :: \bigvee OR(∪)\\
◇ : ||F_{i+1} || ≡ false\\
$$


