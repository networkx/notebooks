---
jupytext:
  notebook_metadata_filter: all
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.1
kernelspec:
  display_name: Python 3
  language: python
  name: python3
language_info:
  codemirror_mode:
    name: ipython
    version: 3
  file_extension: .py
  mimetype: text/x-python
  name: python
  nbconvert_exporter: python
  pygments_lexer: ipython3
  version: 3.9.2
---

# Tutorial: Directed Acyclic Graphs

In this tutorial, we will explore the algorithms related to a directed acyclic graph
(or a "dag" as it is sometimes called) implemented in networkx under `networkx/algorithms/dag.py`.

First of all, we need to understand what a directed graph is.

## Directed Graph

### Example

```{code-cell} ipython3
# import some useful libraries
%matplotlib inline
import networkx as nx
import matplotlib.pyplot as plt
```

```{code-cell} ipython3
# create DiGraph
triangle_graph = nx.from_edgelist([(1, 2), (2, 3), (3, 1)], create_using=nx.DiGraph)
```

```{code-cell} ipython3
# draw `triangle_graph`
nx.draw_planar(triangle_graph,
    with_labels=True,
    node_size=1000,
    node_color="#ffff8f",
    width=0.8,
    font_size=14,
)
```

### Definition

In mathematics, and more specifically in graph theory,
a directed graph (or DiGraph) is a graph that is made up of a set of vertices
connected by directed edges often called arcs.

## Directed Acyclic Graph

### Example

```{code-cell} ipython3
# load Professor Bumstead clothing graph
clothing_graph = nx.read_graphml(f"data/clothing_graph.graphml")
```

```{code-cell} ipython3
# set figure size
plt.figure(figsize=(12, 12), dpi=150)

# draw `clothing_graph`
nx.draw_planar(clothing_graph,
    arrowsize=12,
    with_labels=True,
    node_size=8000,
    node_color="#ffff8f",
    linewidths=2.0,
    width=1.5,
    font_size=14,
)
```

Here is an example that arises when Professor Bumstead gets dressed in the morning.
The professor must do certain garments before others (e.g., socks before shoes).
Other items may be put on in any order (e.g., socks and pants).

A directed edge $(u, v)$ in the example indicates that garment $u$
must be donned before garment $v$.

For example, the `clothing_graph` is a DAG.

```{code-cell} ipython3
# check if the `clothing_graph` is DAG
print(nx.is_directed_acyclic_graph(clothing_graph))
```

But, the `triangle_graph` is not a DAG.

```{code-cell} ipython3
# check if the `triangle_graph` is DAG
print(nx.is_directed_acyclic_graph(triangle_graph))
```

`triangle_graph` has a cycle:

```{code-cell} ipython3
# find cycle in the `triangle_graph`
print(nx.find_cycle(triangle_graph))
```

### Applications

Directed acyclic graphs representations of partial orderings have many applications in scheduling
for systems of tasks with ordering constraints.
An important class of problems of this type concern collections of objects that need to be updated,
such as the cells of a spreadsheet after one of the cells has been changed,
or the object files of a piece of computer software after its source code has been changed.
In this context, a dependency graph is a graph that has a vertex for each object to be updated,
and an edge connecting two objects whenever one of them needs to be updated earlier than the other.
A cycle in this graph is called a circular dependency, and is generally not allowed,
because there would be no way to consistently schedule the tasks involved in the cycle.
Dependency graphs without circular dependencies form DAGs.

A directed acyclic graph may be used to represent a network of processing elements.
In this representation, data enters a processing element through its incoming edges
and leaves the element through its outgoing edges.
For instance, in electronic circuit design, static combinational logic blocks
can be represented as an acyclic system of logic gates that computes a function of an input,
where the input and output of the function are represented as individual bits.

### Definition

Directed acyclic graph ("DAG" or "dag") is a directed graph with no directed cycles.
That is, it consists of vertices and edges (also called arcs), with each edge directed from one vertex to another,
such that following those directions will never form a closed loop.

A directed graph is a DAG if and only if it can be topologically ordered
by arranging the vertices as a linear ordering that is consistent with all edge directions.

## Topological sort

### Example

```{code-cell} ipython3
# get a topological sort of a dag
print(list(nx.topological_sort(clothing_graph)))
```

### Applications

The canonical application of topological sorting is in scheduling a sequence of jobs
or tasks based on their dependencies.
The jobs are represented by vertices, and there is an edge from $u$ to $v$
if job $u$ must be completed before job $v$ can be started
(for example, when washing clothes, the washing machine must finish before we put the clothes in the dryer).
Then, a topological sort gives an order in which to perform the jobs.

A closely related application of topological sorting algorithms
was first studied in the early 1960s in the context of the PERT technique for scheduling in project management.
In this application, the vertices of a graph represent the milestones of a project,
and the edges represent tasks that must be performed between one milestone and another.
Topological sorting forms the basis of linear-time algorithms for finding
the critical path of the project, a sequence of milestones and tasks that controls
the length of the overall project schedule.

In computer science, applications of this type arise in instruction scheduling,
ordering of formula cell evaluation when recomputing formula values in spreadsheets,
logic synthesis, determining the order of compilation tasks to perform in makefiles,
data serialization, and resolving symbol dependencies in linkers.
It is also used to decide in which order to load tables with foreign keys in databases.

### Definition

A topological sort of a directed acyclic graph $G = (V, E)$ is a linear ordering of all its vertices
such that if $G$ contains an edge $(u, v)$, then $u$ appears before $v$ in the ordering.

It is worth noting that if the graph contains a cycle, then no linear ordering is possible.

It is useful to view a topological sort of a graph as an ordering of its vertices
along a horizontal line so that all directed edges go from left to right.

### Kahn's algorithm

First, find a list of "start nodes" which have no incoming edges and insert them into a set S;
at least one such node must exist in a non-empty acyclic graph. Then:

```
L <- Empty list that will contain the sorted elements
S <- Set of all nodes with no incoming edge

while S is not empty do
    remove a node n from S
    add n to L
    for each node m with an edge e from n to m do
        remove edge e from the graph
        if m has no other incoming edges then
            insert m into S

if graph has edges then
    return error  # graph has at least one cycle
else 
    return L  # a topologically sorted order
```

### Asymptotics

The usual algorithms for topological sorting have running time linear
in the number of nodes plus the number of edges, asymptotically,
$\mathcal{O}(|V| + |E|)$.