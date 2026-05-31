# Module 2 Assignment — Exploratory Network Data Analysis: Facebook Social Graph

> Course: Exploratory Data Analysis  
> Module: 2  
> Dataset: Facebook Social Circles (SNAP)  

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Dataset](#dataset)
3. [Requirements](#requirements)
4. [Installation and Setup](#installation-and-setup)
5. [How to Run](#how-to-run)
6. [Analysis Pipeline](#analysis-pipeline)
   - [Graph Construction](#graph-construction)
   - [Centrality Measures](#centrality-measures)
   - [Network-Level Statistics](#network-level-statistics)
   - [Degree Distribution Summary](#degree-distribution-summary)
   - [Ego Network Extraction and Export](#ego-network-extraction-and-export)
7. [Visualizations](#visualizations)
   - [Histogram: Friendship Distribution](#histogram-friendship-distribution)
   - [Network Graph: Centrality Coloring](#network-graph-centrality-coloring)
   - [Bar Chart: Top 5 Most Connected Users](#bar-chart-top-5-most-connected-users)
8. [Key Metrics and Findings](#key-metrics-and-findings)
9. [GraphML Export for Gephi](#graphml-export-for-gephi)
10. [Limitations and Future Work](#limitations-and-future-work)

---

## Project Overview

This module assignment performs exploratory analysis of the Facebook ego-network dataset from the Stanford Network Analysis Project (SNAP). An undirected graph is constructed from a raw edge list file, three distinct centrality measures are computed to identify the most important nodes in the network, and key structural properties of the network as a whole are examined. The top three most central nodes each have their local ego networks (depth-2 subgraphs) extracted and exported to GraphML format for further visualization in Gephi. Three visualizations summarize the degree distribution, the full network structure, and the top five most connected users.

---

## Dataset

**File:** `facebook_combined.txt`  
**Source:** Stanford Network Analysis Project (SNAP)  
**Format:** Plain text, one edge per line, space-separated node IDs  
**Graph type:** Undirected (friendship relationships are bidirectional)

The dataset represents combined ego networks collected from Facebook via a survey app. Each line in the file contains two integer node IDs representing a mutual friendship connection.

**Citation:**  
J. McAuley and J. Leskovec. *Learning to Discover Social Circles in Ego Networks*. NIPS, 2012.

---

## Requirements

- Python 3.x

```bash
pip install networkx matplotlib
```

| Package      | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `networkx`   | Graph construction, centrality computation, ego graph, export  |
| `matplotlib` | All three visualizations                                       |

---

## Installation and Setup

1. Download `facebook_combined.txt` from the SNAP website or course materials.
2. Place the file in the same directory as `Module2Assignment.ipynb`.
3. Open the notebook in Jupyter Notebook or JupyterLab.

---

## How to Run

Open the notebook and run all cells in order. The first cell loads data, builds the graph, prints centrality rankings, prints network statistics, and extracts ego networks. The second cell produces the three-panel visualization. The third cell extracts and exports ego networks to GraphML.

Runtime note: betweenness centrality is the most computationally expensive step due to the need to enumerate shortest paths. The `k=500` approximation parameter is set to sample only 500 pivot nodes rather than all nodes, substantially reducing computation time at a minor cost to precision.

---

## Analysis Pipeline

### Graph Construction

The file is read line by line using a custom `load_facebook_data()` function. Each non-empty line is stripped of whitespace and split into two parts. If exactly two parts are found, they are converted to integers and appended as a tuple to the edge list. This approach handles any formatting inconsistencies without crashing:

```python
def load_facebook_data(file_path='facebook_combined.txt'):
    edges = []
    with open(file_path, 'r') as file_connection:
        for line in file_connection:
            if line.strip():
                parts = line.strip().split()
                if len(parts) == 2:
                    edges.append((int(parts[0]), int(parts[1])))
    return edges
```

An undirected `networkx.Graph()` is constructed (`nx.Graph()`) because Facebook friendships are mutual — an edge between nodes A and B implies A is friends with B and B is friends with A.

---

### Centrality Measures

Three centrality measures are computed on the full graph and the top 20 nodes are printed for each:

**Degree Centrality** measures the proportion of all other nodes that a given node is directly connected to. A high degree centrality means the user has the most direct friend connections relative to the total network size. Values range from 0.0 to 1.0:

```python
degree_centrality = nx.degree_centrality(g)
```

**Betweenness Centrality** measures how often a node lies on the shortest path between two other nodes. A high betweenness score means the user acts as a bridge between otherwise disconnected parts of the network — removing them would fragment communication flows. Approximated with `k=500` pivot nodes:

```python
betweenness_centrality = nx.betweenness_centrality(g, k=500)
```

**Eigenvector Centrality** measures influence by weighting a node's connections by the centrality of those connections. Being connected to highly central nodes increases your own eigenvector score — it captures the idea of being well-connected to well-connected people:

```python
eigenvector_centrality = nx.eigenvector_centrality(g)
```

All three dictionaries are sorted in descending order using `sorted(..., key=lambda x: x[1], reverse=True)` before printing.

---

### Network-Level Statistics

Three global structural properties are computed:

| Metric                         | Method                               | Interpretation                                                                 |
|-------------------------------|--------------------------------------|--------------------------------------------------------------------------------|
| Number of connected components | `nx.number_connected_components(g)` | Whether the network is fully connected or has isolated subgraphs               |
| Average clustering coefficient | `nx.average_clustering(g)`          | Tendency for friends of friends to also be friends with each other             |
| Network diameter               | `nx.diameter(g)`                    | The longest shortest path — the maximum number of hops between any two users   |

The clustering coefficient is especially relevant for social networks where tight friend groups (cliques) are common. A high coefficient indicates the "small world" phenomenon where people cluster into dense social circles.

---

### Degree Distribution Summary

Raw degree values are extracted using a list comprehension over `g.degree()`:

```python
degrees = [d for n, d in g.degree()]
```

The maximum and minimum degree values are printed to identify the most and least connected nodes in the network.

---

### Ego Network Extraction and Export

Ego networks are extracted for the three nodes that ranked highest across all three centrality measures — nodes **107**, **1684**, and **1912**. For each node, `nx.ego_graph(g, node_id, radius=2)` produces the subgraph containing the target node, all of its direct friends (radius=1), and all of their friends (radius=2). This gives a neighborhood view two hops deep.

Key stats printed per ego network:
- Total nodes in the subgraph
- Total edges in the subgraph
- Direct connection count (degree) of the central node

Each subgraph is then exported to GraphML format using `nx.write_graphml()`:

```python
filename = f"node_{node_id}_ego_graph.graphml"
nx.write_graphml(ego_subgraph, filename)
```

This produces three files (`node_107_ego_graph.graphml`, `node_1684_ego_graph.graphml`, `node_1912_ego_graph.graphml`) that can be loaded directly into Gephi for interactive layout and community detection visualization.

---

## Visualizations

All three visualizations are produced in a single `plt.figure()` with a 1x3 subplot layout (12x4 inches total).

### Histogram: Friendship Distribution

**Position:** Subplot 1  
**Type:** Histogram  
**Bins:** 50  
**Color:** Orange with black edge color  
**Y-axis scale:** Logarithmic (`plt.yscale('log')`)

The degree of every node is plotted as a histogram. The log scale on the y-axis is essential here — without it, the highly skewed distribution (many nodes with few friends, very few nodes with many friends) compresses the interesting tail into an unreadable spike. The log scale reveals the power-law-like shape characteristic of real-world social networks.

---

### Network Graph: Centrality Coloring

**Position:** Subplot 2  
**Layout:** Spring layout (`nx.spring_layout`)  
**Node size:** 10  
**Node color:** Degree value mapped to `hot` colormap  
**Edge color:** Gray at width 0.1 and `alpha=0.5`

The spring layout algorithm positions nodes using a force-directed algorithm where well-connected nodes are pulled toward the center and loosely connected nodes drift to the periphery. Each node's color is mapped to its degree value, making the most connected users visually prominent as bright spots in the dense network core.

---

### Bar Chart: Top 5 Most Connected Users

**Position:** Subplot 3  
**Type:** Vertical bar chart  
**Color:** Blue, `alpha=0.5`

The top 5 node IDs from the sorted degree centrality list are extracted and their centrality scores plotted as bars. X-tick labels display "Node {id}" for each bar, rotated 45 degrees.

---

## Key Metrics and Findings

- The network contains thousands of nodes with a notably skewed degree distribution — the majority of users have relatively few connections, while a small number of high-degree hub nodes connect large portions of the network.
- Nodes 107, 1684, and 1912 consistently rank at the top across all three centrality measures, indicating they are central not just in terms of raw connections (degree) but also as structural bridges (betweenness) and influential connectors (eigenvector).
- The average clustering coefficient is high relative to a random graph of the same size, confirming the "small world" structure typical of social networks — friend groups are tightly interconnected.
- The network diameter quantifies the maximum social distance between any two users in the dataset, giving a concrete measure of how few intermediaries separate the most distant users.

---

## GraphML Export for Gephi

Three GraphML files are written to the working directory upon running the ego network cell:

| File                          | Description                                        |
|-------------------------------|----------------------------------------------------|
| `node_107_ego_graph.graphml`  | Depth-2 ego network centered on node 107           |
| `node_1684_ego_graph.graphml` | Depth-2 ego network centered on node 1684          |
| `node_1912_ego_graph.graphml` | Depth-2 ego network centered on node 1912          |

To visualize in Gephi: File > Open > select `.graphml` > apply a layout algorithm (e.g., ForceAtlas2) > run modularity for community detection coloring.

---

## Limitations and Future Work

- Betweenness centrality is approximated with `k=500`. Using all nodes (`k=None`) would give exact values but requires substantially more compute time on a graph of this size.
- The spring layout in the network visualization places all nodes at once using a random seed, meaning the layout is not deterministic across runs. Setting `seed=42` in `nx.spring_layout()` would ensure reproducibility.
- Community detection algorithms (Louvain, Girvan-Newman) could be applied to formally identify the social circles present in the ego networks beyond the visual impression from the spring layout.
- Node features (profile attributes such as education, work, location) are available in the extended SNAP dataset and could be incorporated to profile the characteristics of high-centrality users beyond topology alone.
