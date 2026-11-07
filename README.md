# Tehran Smart Route Planner
### Real-World Pathfinding, Traffic Simulation & Route Optimization

A Python-based pathfinding system built on the real road network of **Tehran, Iran**, using OpenStreetMap (OSM) data.

The project combines graph theory, classical search algorithms, geospatial data processing, and traffic simulation to explore and compare different route-finding strategies.

It implements **Depth-First Search (DFS)** and **traffic-aware A\* Search**, visualizing their results on an interactive map with support for dynamic road closures and draggable start/destination markers.

## Overview

The system processes real-world geographic data to construct a navigable road network of Tehran. It then simulates traffic conditions across road segments and evaluates alternative routes using two different search algorithms.

Users can interact with the map, select starting and destination locations, simulate road closures, and visually compare the resulting routes.

### Key Features

- **Real-World Road Network:** Extracts Tehran's driving network from OpenStreetMap data.
- **Graph Construction:** Converts geographic road data into a NetworkX `MultiDiGraph`.
- **Traffic Simulation:** Generates spatially varying traffic conditions using a Gaussian-based model.
- **Dynamic Road Closures:** Simulates blocked roads, including Zeinoddin Highway.
- **Interactive Map:** Supports draggable markers for selecting starting points and destinations.
- **DFS Pathfinding:** Implements Depth-First Search for route discovery.
- **Traffic-Aware A\* Search:** Implements A* with geographic heuristics and traffic-weighted edge costs.
- **Route Comparison:** Evaluates routes using actual distance and traffic-weighted distance.
- **Interactive Visualization:** Displays traffic intensity and both algorithmic routes on Tehran's map.

## Technologies & Libraries

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| OpenStreetMap (OSM) | Real-world geographic and road data |
| Pyrosm | Loading and parsing OSM data |
| NetworkX | Road network graph representation |
| OSMnx | Geographic nearest-node lookup |
| ipyleaflet | Interactive map visualization |
| NumPy | Numerical computations and traffic simulation |
| Geospatial libraries | Geographic distance and coordinate processing |

## System Architecture & Workflow

The project follows a multi-stage workflow, from geographic data extraction to route visualization.

### 1. Loading OpenStreetMap Data

Tehran's road network is extracted from an OSM `.osm.pbf` file using Pyrosm.

```python
from pyrosm import OSM

osm = OSM("Tehran_2.osm.pbf")

nodes, roads = osm.get_network(
    nodes=True,
    network_type="driving"
)

Graph = osm.to_graph(nodes, roads, "networkx")
```

The resulting graph is a NetworkX `MultiDiGraph` containing geographic nodes and directed road edges.

Each road segment includes relevant metadata, such as:

- Geographic length
- Highway classification
- One-way restrictions
- Additional OpenStreetMap attributes

### 2. Gaussian-Based Traffic Simulation

To simulate spatial variations in traffic intensity, a Gaussian-based model is centered around **Enghelab Square** in Tehran.

For each selected road segment:

1. Calculate the segment midpoint.
2. Compute its geographic distance from Enghelab Square.
3. Apply Gaussian decay to model traffic intensity.
4. Introduce random noise to create variation.
5. Assign a traffic category.

Three traffic levels are generated:

| Traffic Level | A* Cost Multiplier |
|---|---:|
| Low | ×1 |
| Medium | ×2 |
| High | ×4 |
| Blocked | Excluded |

Only 1% of sampled road edges are displayed on the traffic visualization map to keep the visualization manageable.

**Note:** Traffic conditions are simulated rather than retrieved from live traffic services.

### 3. Dynamic Road Closures

The project simulates road closures by marking all edges connected to a specific node representing **Zeinoddin Highway** as inaccessible.

```python
d['access'] = 'no'
d['traffic'] = 'blocked'
```

Blocked road segments are excluded from both DFS and A* route searches.

The algorithms must therefore find alternative routes whenever the original route is affected by a closure.

### 4. Interactive Start & Destination Selection

The system uses `ipyleaflet` to provide an interactive map with two draggable markers:

- **Start:** Represents the starting location (Home).
- **Destination:** Represents the target location (Parking).

Moving either marker automatically updates its geographic coordinates and maps its new position to the nearest road network node.

This allows users to experiment with different routes without manually specifying graph node IDs.

### 5. Geographic Node Mapping

The selected coordinates are mapped to their nearest graph nodes using OSMnx:

```python
nearest = ox.nearest_nodes(
    Graph,
    X=lon,
    Y=lat
)
```

The resulting node IDs are passed to the pathfinding algorithms.

## Pathfinding Algorithms

The project implements and compares two graph search algorithms with different optimization strategies.

### 1. Depth-First Search (DFS)

A stack-based Depth-First Search implementation is used to discover a valid path between the selected nodes.

Key characteristics:

- Stack-based graph traversal.
- Blocked-edge filtering.
- Path reconstruction.
- Calculation of the resulting route's actual geographic distance.

DFS does not consider traffic conditions or optimize route length. Consequently, the path it discovers may be significantly longer than an optimized alternative.

**Visualization:** Red polyline.

### 2. A* Search with Traffic-Aware Cost

A* Search is implemented using a traffic-aware edge cost function and a geographic heuristic.

The evaluation function is:

\[
f(n) = g(n) + h(n)
\]

Where:

- **g(n):** Accumulated traffic-weighted movement cost from the starting node.
- **h(n):** Geodesic straight-line distance from the current node to the destination.

#### Traffic-Aware Edge Weights

Each road segment receives a cost based on its geographic length and simulated traffic intensity.

\[
w(e) = L(e) \times P(e)
\]

Where:

- \(L(e)\) is the actual road segment length.
- \(P(e)\) is the traffic penalty.

Traffic penalties are assigned as follows:

- Low traffic: 1
- Medium traffic: 2
- High traffic: 4

Blocked edges are excluded from the search entirely.

#### Advantages

- Considers both accumulated travel cost and estimated remaining distance.
- Penalizes congested road segments.
- Avoids blocked roads.
- Searches for lower-cost alternatives.
- Produces traffic-aware routes rather than arbitrary paths.

**Heuristic:** Geographic straight-line distance, designed to provide an admissible lower-bound estimate under the implemented edge-cost model.

**Visualization:** Lime-green polyline.

## Route Evaluation & Comparison

The project evaluates A* routes using two different distance metrics.

### 1. Real Distance

The sum of the original geometric lengths of all road segments in the selected path.

\[
D_{\text{real}} = \sum_{e \in P} L(e)
\]

### 2. Traffic-Weighted Distance

The sum of road lengths multiplied by their corresponding traffic penalties.

\[
D_{\text{weighted}} =
\sum_{e \in P} L(e) \times P(e)
\]

These metrics provide a way to compare route length against simulated traffic exposure.

They also help evaluate how traffic-aware optimization changes the selected route compared with the DFS result.

## Interactive Visualization

The project uses `ipyleaflet` to display the road network and pathfinding results directly on an interactive map of Tehran.

The visualization includes:

- Simulated traffic heatmap.
- DFS route (Red).
- A* optimized route (Lime Green).
- Draggable starting and destination markers.
- Dynamic route updates based on marker positions.
- Alternative paths resulting from road closures.

The interactive interface makes it possible to observe how different algorithms respond to changes in the road network and traffic conditions.

## Project Structure

The following is a suggested repository structure; filenames can be adjusted to match the actual project files.

```text
tehran-smart-route-planner/
│
├── README.md
│
├── notebooks/
│   └── pathfinding.ipynb
│
├── data/
│   └── Tehran_2.osm.pbf
│
├── images/
│   ├── traffic-map.png
│   ├── dfs-route.png
│   └── astar-route.png
│
├── requirements.txt
│
└── .gitignore
```

**Note:** The original OSM `.pbf` dataset can be large. It does not necessarily need to be committed to GitHub. A download/source reference and setup instructions can be provided instead.

## Installation & Usage

### Prerequisites

- Python 3.x
- Jupyter Notebook
- OpenStreetMap Tehran `.osm.pbf` dataset

### 1. Clone the Repository

```bash
git clone https://github.com/VahhabBM/tehran-smart-route-planner.git

cd tehran-smart-route-planner
```

### 2. Install Dependencies

```bash
pip install pyrosm networkx osmnx ipyleaflet numpy
```

Install any additional dependencies required by the notebook.

### 3. Prepare the OSM Dataset

Place the Tehran `.osm.pbf` dataset in the appropriate data directory and update the dataset path in the notebook if necessary.

### 4. Run the Project

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Open the project notebook and execute the cells to load the road network, simulate traffic, configure road closures, and visualize the resulting routes.

## Key Takeaways

This project demonstrates the practical application of several computer science concepts:

- Graph representation and traversal.
- Classical search algorithms.
- Heuristic-based search and route optimization.
- Weighted graph algorithms.
- Geographic coordinate processing.
- Spatial simulation using Gaussian distributions.
- Interactive data visualization.
- Real-world geographic data integration.

It also highlights the practical differences between uninformed search (DFS) and heuristic-based search (A*) when applied to a large, real-world road network.

## Potential Real-World Applications

The techniques explored in this project can be extended to:

- Urban navigation systems.
- Traffic-aware route planning.
- Urban mobility analysis.
- Emergency vehicle routing.
- Delivery and logistics optimization.
- Road network simulation.

## Contributors

- **Vahhab Bolandmanesh**
- **Mohammadmehdi Kazemi**

K. N. Toosi University of Technology

## Academic Context

**Course:** Data Structures & Algorithms / Algorithm Design (verify the official course title before publishing).

**Project Type:** Academic Team Project

**Geographic Focus:** Tehran, Iran

---

*Built to explore how classical graph algorithms can be applied to real-world geographic networks and dynamic routing problems.*
