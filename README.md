# Critical Path Method (CPM)

See Youtube for an overview:
* https://www.youtube.com/playlist?list=PLOAuB8dR35oeyKU0ojIxD8Muf6Mwc8ugW

-----

### Step 1 : Importing the library


```python
import networkx as nx
```

[NetworkX](https://networkx.github.io/) is a Python package for the creation, manipulation, and study of the structure, dynamics, and functions of complex networks.

### Step 2: Defining the CPM
By redefining the DiGraph class to suit our purpose


```python
class CPM(nx.DiGraph):
    def __init__(self):
        super().__init__()
        self._dirty = True
        self._critical_path_length = -1
        self._criticalPath = None

    def add_node(self, *args, **kwargs):
        self._dirty = True
        super().add_node(*args, **kwargs)

    def add_nodes_from(self, *args, **kwargs):
        self._dirty = True
        super().add_nodes_from(*args, **kwargs)

    def add_edge(self, *args):
        self._dirty = True
        super().add_edge(*args)

    def add_edges_from(self, *args, **kwargs):
        self._dirty = True
        super().add_edges_from(*args, **kwargs)

    def remove_node(self, *args, **kwargs):
        self._dirty = True
        super().remove_node(*args, **kwargs)

    def remove_nodes_from(self, *args, **kwargs):
        self._dirty = True
        super().remove_nodes_from(*args, **kwargs)

    def remove_edge(self, *args):
        self._dirty = True
        super().remove_edge(*args)

    def remove_edges_from(self, *args, **kwargs):
        self._dirty = True
        super().remove_edges_from(*args, **kwargs)

    def _forward(self):
        for n in nx.topological_sort(self):
            es = max([self.nodes[j]["EF"] for j in self.predecessors(n)], default=0)
            self.add_node(n, ES=es, EF=es + self.nodes[n]["duration"])

    def _backward(self):
        for n in reversed(list(nx.topological_sort(self))):
            lf = min(
                [self.nodes[j]["LS"] for j in self.successors(n)],
                default=self._critical_path_length,
            )
            self.add_node(n, LS=lf - self.nodes[n]["duration"], LF=lf)

    def _compute_critical_path(self):
        graph = set()
        for n in self:
            if self.nodes[n]["EF"] == self.nodes[n]["LF"]:
                graph.add(n)
        self._criticalPath = self.subgraph(graph)

    @property
    def critical_path_length(self):
        if self._dirty:
            self._update()
        return self._critical_path_length

    @property
    def critical_path(self):
        if self._dirty:
            self._update()
        return sorted(self._criticalPath, key=lambda x: self.nodes[x]["ES"])

    def _update(self):
        self._forward()
        self._critical_path_length = max(nx.get_node_attributes(self, "EF").values())
        self._backward()
        self._compute_critical_path()
        self._dirty = False
```

### Step 3: Defining the nodes


```python
G = CPM()
G.add_node("A", duration=5)
G.add_node("B", duration=2)
G.add_node("C", duration=4)
G.add_node("D", duration=4)
G.add_node("E", duration=3)
G.add_node("F", duration=7)
G.add_node("G", duration=3)
G.add_node("H", duration=2)
G.add_node("I", duration=4)
```

### Step 4 : Defining the edges connecting the nodes


```python
G.add_edges_from(
    [
        ('A', 'C'),
        ('A', 'D'),
        ('B', 'E'),
        ('B', 'F'),
        ('D', 'G'), ('E', 'G'),
        ('F', 'H'),
        ('C', 'I'), ('G', 'I'), ('H', 'I')
    ]
)
```

### Step 5 : Solving the critical path


```python
print("Critical path:")
print(G.critical_path_length, G.critical_path)
```

    Critical path:
    16 ['A', 'D', 'G', 'I']
    

#### Editing an existing node


```python
G.add_node("D", duration=2)
```

... to find out how it impacts the critical path


```python
print("Crushing D, from 4 to 2, yields:")
print(G.critical_path_length, G.critical_path)
```

    
    Crushing D, from 4 to 2, yields:
    15 ['B', 'F', 'H', 'I']
    
