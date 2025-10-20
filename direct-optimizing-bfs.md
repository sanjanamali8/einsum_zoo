# Scott Beamer Paper - Direction-Optimizing BFS

# Top down algorithm -

## English Description -

Given a graph G with E edges and V vertices, discover all vertices that are reachable from a source node s. The algorithm builds a Breadth-First Search (BFS) tree, which records for each vertex the parent vertex that first discovered it during traversal.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

## Steps - 

1. Initialize the frontier with the source node s.
2. Find the neighbors of each node in the frontier.
3. If a neighbor’s parent is not yet recorded, assign the current node as its parent.
4. Add all newly discovered neighbors to the frontier.
5. Repeat from Step 2 while the frontier is not empty.

## Algorithm (Step-by-Step EDGE) -

Tensors -

$$
\begin{array}{l}
\triangleright \text{Tensors} \\
G^{S\equiv|V|,D\equiv|V|} \rightarrow \text{Boolean},\ \text{empty}=false\\
F^{I,S\equiv|V|} \rightarrow \text{Boolean},\ \text{empty}=false\\
\text{Tree}^{I,S\equiv|V|,D\equiv|V|} \rightarrow \text{Boolean},\ \text{empty}=false
\end{array}
$$


1. Create a 2D tensor G (Graph) with ranks S and D, where S represents the source and D represents destination. The initial values are either false if s is not connected to d by an edge and true if s is connected to d by an edge.
    - $G^{S,D} → \text{boolean}, \text{empty}= false$
      
2. To keep track of the active vertices, another data structure is required, called Frontier F, where each element in F stores either true or false. We create a Tensor F which stores 1 if the corresponding vertex is active. To keep the track for every iteration, we add another dimension i. Thus F now stores true if the corresponding vertex is active at iteration i.
    - $F^{I, S} → \text{boolean}, \text{empty}=false$
      
3. Initialize the frontier with the source node n by setting its entry to true for iteration 0.
    - $F_{0, n} = true$
      
4. To find the neighbors of each node in the frontier. We store the neighbours in tensor N.
    - $N_{i, d} = G_{s, d} ⋅ F_{i, s} :: ⋀ \text{AND} (∩) ⋁ \text{OR}(∪)$
      
5. Initialize the Tree tensor for the source node such that the source node is its own parent. This prevents infinite loops in case the source has a self-edge.
    - $\text{Tree}_{0, n, n} = true$
      
6. To check which vertices have parents (i.e., have been visited), reduce the Tree tensor over the parent rank p. A given P-fiber (that is, fix "c" to some coordinate) represents the set of parents for that "c" node. We use the OR operator to reduce over the $P$ rank, which returns true if at least ONE parent exists for "c". Thus we use an OR operator here. The intermediate result is saved in a tensor HP (Have Parents)
    - $HP_{i,c} = Tree_{i,p,c} :: \bigvee \text{OR}(\cup)$

7. To identify nodes without parents, take the complement of tensor HP. HP marks vertices that already have parents in the tree; its complement gives the set of unvisited nodes, nodes that do NOT have Parents (NP)
   - $NP_{i, c} = ¬\text{HP}_{i, c}$
     
8. Intersecting NP with N (Neighbors of frontier) yields the set of Neighbors of Frontier nodes that are not Visited (NFV). Thus, returns NFV - Neighbors of Frontier not Visited, the set of nodes that are neighbors of vertices in the frontier **and** do not have parents.
    - $NFV_{i,d} = N_{i,d} ⋅ \text{NP}_{i,d} :: ⋀ \text{AND}(∩)$

9. Add the neighbors without parents (unvisited nodes) to the new frontier.
    - $F_{i+1, d} = \text{NFV}_{i, d}$
      
10. Update the Tree to mark visited neighbors. Create a temporary tree by intersecting the Graph (to confirm the parent–child edge), the Frontier (active vertices), and NP (newly discovered children).
    - $TTree_{i,s,d} = (G_{s,d} \cdot F_{i,s})_{i,s,d} ⋅ NFV_{i,d} :: \bigwedge AND (∩) ⋀ AND(∩)$

11. If multiple parents discover the same child, select only one as the parent. Use the populate operator with a defined coordinate operator pick-parent to enforce this. To achieve this, we use a 'populate' operator to populate another temporary tensor by applying a defined coordinate operator 'pick-parent'    
    - $Temp_{i, s*, d} = TTree_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent})$
  
12. Merge the temporary tree with the existing Tree to preserve all previously visited nodes.
    - $Tree_{i+1,s,d} = Tree_{i,s,d} ⋅ \text{TTree}_{i,s,d} :: ⋀ OR$
      
13. When the newly created frontier becomes empty, stop the recursion.
    - $◇ : ||F_{i+1}|| ≡ false$
   
## Full EDGE 

$$
\begin{array}{l}
\triangleright \textbf{Tensors} \\
G^{S≡|V|,D≡|V|} → \text{Boolean}, \text{empty}=false\\
F^{I, S≡|V|} →\text{Boolean}, \text{empty}=false\\
\text{Tree}^{I, S≡|V|, D≡|V|} →\text{Boolean}, \text{empty}=false \\
\\
\\
\triangleright \textbf{Initializations} \\
F_{i,s} = false \\
Tree_{i,s,d} = false \\
G_{s,d} → <user-specified> \\
F_{0, n} = true \\
\text{Tree}_{0, n, n} = true \\
\\
\\
\triangleright \textbf{Extended Einsums} \\
N_{i, d} = G_{s, d} ⋅ F_{i, s} :: ⋀ \text{AND} (∩) ⋁ \text{OR}(∪) &\text{...gather neighbors of nodes in frontier}\\ 
\text{HP}_{i,c} = \text{Tree}_{i,p,c} :: ⋁ \text{OR}(∪) &\text{...find visited nodes (Have Parents - HP)}\\
\text{NP}_{i,c} = ¬\text{HP}_{i,c} &\text{...find unvisited nodes (do Not have Parents - NP)}\\
\text{NFV}_{i,d} = N_{i,d} ⋅ \text{NP}_{i,d} :: ⋀ \text{AND}(∩) &\text{...check if neighbors were already visited (Neighbors of Frontier Visited - NFV)} \\
F_{i+1, d} = \text{NFV}_{i, d} &\text{...update frontier for next iteration}\\
\text{TTree}_{i,s,d} = ( G_{s,d} ⋅ F_{i,s} )_{i,s,d} ⋅ NFV_{i,d} :: \bigwedge AND (∩) ⋀ AND(∩) &\text{...record parents of newly discovered nodes}\\
\text{Temp}_{i, s*, d} = \text{TTree}_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent}) &\text{...pick just one parent for each node}\\
\text{Tree}_{i+1,s,d} = \text{Tree}_{i,s,d} ⋅ \text{TTree}_{i,s,d} :: ⋀ OR &\text{...update Tree for next iteration}\\
◇ : ||F_{i+1}|| ≡ false &\text{...terminate when new frontier is empty}
\end{array}
$$


# Bottom up algorithm -

## English Description -

Given a graph G with E edges and V vertices, discover all vertices that are reachable from a source node s. The algorithm builds a Breadth-First Search (BFS) tree, which records for each vertex the parent vertex that first discovered it during traversal.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

## Steps -

1. Initialize the frontier with the source node.
2. Add the source to the tree as its own parent.
3. Check which nodes already have parents (to find unvisited nodes).
4. Find the neighbors of all unvisited nodes.
5. If any neighbor is in the frontier, add that node to the new frontier.

## Algorithm (Step-by-Step-EDGE) -

Tensors - 

$$
\begin{array}{l}
→Tensors \\
G^{S,D} → \text{boolean}, \text{empty}=false\\
F^{I,S} → \text{boolean}, \text{empty}=false\\
\text{Tree}^{I,S,D} →\text{boolean}, \text{empty}=false \\
\text{NNP}^{I,S,D} →\text{boolean}, \text{empty}=false \\
\text{InF}^{I,S,D} →\text{boolean}, \text{empty}=false \\
\end{array}
$$


1. Create a 2D tensor G with ranks S and D, where S represents source and D represents destination. The initial values are either false if s is not connected to d by an edge and true if s is connected to d by an edge.
    - $G^{S,D} → \text{boolean}, \text{empty}=false$
      
2. To keep track of active vertices, create another data structure called Frontier F, where each element stores true if the vertex is active and false otherwise. We create a Tensor F which stores True if the corresponding vertex is active. To keep the track for every iteration, we add another dimension i. Thus F now stores True if the corresponding vertex is active at iteration i (and False otherwise).
    - $F^{I, S} → \text{boolean}, \text{empty}=false$
    
3. Initialize the frontier with the source node n by setting its value to true at iteration 0.
    - $F_{0, n} = true$
      
4. Initialize the Tree tensor so that the source node is its own parent. This prevents infinite loops if the source has a self-edge.
    - $Tree_{0, n, n} = True$
      
5. To check which vertices have parents (i.e., have been visited), reduce the Tree tensor over the parent rank p. This returns a tensor where each entry is true if any parent exists for that vertex. One rank is iteration i and another is 'c' giving us all the nodes that have true values if any of the value in parent rank 'p' was true. Thus we use an OR operator here. The intermediate result is saved in a tensor HP (Have Parents)
    - $HP_{i,c}= \text{Tree}_{𝑖,𝑝,𝑐}:: \bigvee \text{OR}(\cup)$
      
6. To identify nodes without parents, take the complement of tensor HP. HP marks vertices that already have parents in the tree; its complement gives the set of unvisited nodes, nodes that do NOT have Parents (NP) 
    - $NP_{i,c} = ¬\text{HP}_{i,c}$
      
7. Find the neighbors (potential children) of nodes that do not have parents.
For each unvisited child d, check all possible parent neighbors s.
Create a 3D tensor NNP with ranks I, S, D, where:
– I represents the iteration, S represents the parent vertices and D represents the child vertices. Each entry in NNP is true if, at iteration i, child d is unvisited and an edge exists from s to d. We store the intermediate result in tensor NNP (Neighbors of unvisited nodes(NP))
    - $NNP_{i,s,d} = G_{s,d} ⋅ \text{NP}_{i,d} :: ⋀ \text{AND}$
      
8. Check if any parents are currently in the frontier.
For each unvisited child d, determine whether any of its neighbors s are active in the frontier. Create a tensor InF (In Frontier) with ranks I, S, D where each entry is true if, at iteration i, an edge exists from s to d, d is unvisited, and s is in the frontier.
    - $InF_{i,s,d} = NNP_{i,s,d} ⋅ F_{i,s} :: ⋀ \text{AND}(∩)$
      
9. If multiple parents can reach the same child, select only one parent.
Use the populate operator with a defined coordinate operator pick-parent to enforce this. To achieve this, we use a 'populate' operator to populate another temporary tensor by applying a defined coordinate operator 'pick-parent'
    - $Temp_{i,s*,d} = InF_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent})$
      
10. Update the Tree to record all newly visited nodes discovered in this iteration.
    - $Tree_{i+1,s,d} = Temp_{i,s,d} ⋅ \text{Tree}_{i,s,d} :: ⋀ \text{OR}(∪)$
      
11. Update the frontier for the next iteration with nodes that were visited in the current step.
    - $F_{i+1, d} = \text{InF}_{i,s,d} :: \bigvee \text{OR}(∪)$
      
12. When the new frontier becomes empty, stop the recursion.
    - $◇ : ||F_{i+1} || ≡ false$


## Full Edge

$$
\begin{array}{l}
\triangleright \textbf{Tensors} \\
G^{S≡|V|,D≡|V|} → \text{Boolean}, \text{empty}=false\\
F^{I, S≡|V|} →\text{Boolean}, \text{empty}=false\\
\text{Tree}^{I, S≡|V|, D≡|V|} →\text{Boolean}, \text{empty}=false \\
\text{NNP}^{I, D, S} →\text{boolean}, \text{empty}=false \\
\text{InF}^{I, D, S} →\text{boolean}, \text{empty}=false \\
\\
\\
\triangleright \textbf{Initializations} \\
F_{i,s} = false \\
\text{Tree}_{i,s,d} = false \\
G_{s,d} → <user-specified> \\
\text{NNP}_{i,d,s} = false\\
\text{InF}_{i,d,s} = false\\
F_{0, n} = true \\
Tree_{0, n, n} = true \\
\\
\\
\triangleright \textbf{Extended Einsums} \\
\text{HP}_{i,c} = \text{Tree}_{𝑖,𝑝,𝑐}:: \bigvee \text{OR}(\cup) &\text{...find visited nodes (Have Parents - HP)}\\ 
\text{NP}_{i,c} = ¬\text{HP}_{i, c} &\text{...find unvisited nodes (do Not have Parents - NP)}\\
\text{NNP}_{i,s,d} = G_{s,d} ⋅ \text{NP}_{i,d} :: ⋀ \text{AND} &\text{...find Neighbors of unvisited nodes(NP) - NNP}\\
\text{InF}_{i,s,d} = \text{NNP}_{i,s,d} ⋅ F_{i,s} :: ⋀ \text{AND}(∩) &\text{...check if neighbors of unvisited nodes(NNP) are In Frontier - InF}\\
\text{Temp}_{i,s*,d} = \text{InF}_{i,s,d} \lll_{s*} \mathbb{1}(\text{pick-parent}) &\text{...pick only one parent for each node}\\
\text{Tree}_{i+1,s,d} = \text{Temp}_{i,s,d} ⋅ \text{Tree}_{i,s,d} :: ⋀ \text{OR}(∪) &\text{...update Tree to record newly discovered nodes}\\
F_{i+1, d} = \text{InF}_{i,s,d} :: \bigvee \text{OR}(∪) &\text{...update frontier for next iteration}\\
◇ : ||F_{i+1}|| ≡ false &\text{...terminate when new frontier is empty}\\
\end{array}
$$

# Hybrid -

## English Description -

Given a graph G with E edges and V vertices, discover all vertices that are reachable from a source node s. The algorithm builds a Breadth-First Search (BFS) tree, which records for each vertex the parent vertex that first discovered it during traversal.
This algorithm dynamically switches between the top-down and bottom-up approaches — using top-down when the frontier is small and bottom-up when the frontier becomes large.

- Input: An unweighted graph G with |V| vertices |E| edges.
- Input: Source node s ∈ V
- Input: A frontier initialized with the source node s.
- Output: The set of vertices reachable from s.
- Output: The parent of each vertex in the BFS tree.

### We are working on correctly formulating the switching algorithm in EDGE. 

