# Tehran Smart Route Planner
### Graph-Based Pathfinding & Traffic Simulation Using OpenStreetMap

A Python-based pathfinding project built on **Tehran's real-world road network**, combining graph theory, search algorithms, geospatial data processing, and simulated traffic conditions.

The system extracts road network data from OpenStreetMap, constructs a navigable graph, simulates traffic intensity, and compares two pathfinding algorithms: **Depth-First Search (DFS)** and **traffic-aware A* Search**.

It also features simulated road closures and interactive map visualization using `ipyleaflet`.

## Project Overview

The main objective is to explore how classical graph search algorithms perform on a large, real-world road network under different routing conditions.

The project allows users to:

- Load and process Tehran's road network from OpenStreetMap.
- Represent the road network as a directed graph.
- Simulate geographically distributed traffic conditions.
- Block selected road segments, including a designated Zeinoddin Highway node.
- Select starting points and destinations using draggable map markers.
- Find routes using custom DFS and A* implementations.
- Compare route distances and traffic-weighted costs.
- Visualize traffic conditions and calculated routes on interactive maps.

## Technologies & Libraries

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| OpenStreetMap | Geographic and road network data |
| Pyrosm | OSM data extraction and processing |
| NetworkX | Graph representation and traversal |
| OSMnx | Geographic nearest-node lookup |
| Geopy | Geodesic distance calculations |
| ipyleaflet | Interactive map visualization |
| Jupyter Notebook | Development and execution environment |

## System Workflow

### 1. OpenStreetMap Data Extraction

Tehran's road network is loaded from an OSM `.osm.pbf` file using Pyrosm.

```python
from pyrosm import OSM

osm = OSM("Tehran_2.osm.pbf")

nodes, roads = osm.get_network(
    nodes=True,
    network_type="driving"
)

Graph = osm.to_graph(nodes, roads, "networkx")
```

The extracted road network is represented as a NetworkX `MultiDiGraph`.

Each node contains geographic coordinates, while road edges include attributes such as:

- Road length
- Highway classification
- One-way information
- Additional OpenStreetMap metadata

### 2. Gaussian-Based Traffic Simulation

To simulate spatial variations in traffic intensity, a Gaussian distribution is centered around Enghelab Square in Tehran.

The traffic model considers the geographic midpoint of each road segment and its distance from the traffic center.

The intensity is calculated using:

\[
I = e^{-\frac{d^2}{2\sigma^2}} + \epsilon
\]

Where:

- \(d\) represents the coordinate-based distance from Enghelab Square.
- \(\sigma\) controls the spatial distribution of traffic.
- \(\epsilon\) represents random Gaussian noise.

The generated intensity is clipped to the range [0, 1].

Traffic is classified into three categories:

| Category | Intensity |
|---|---|
| High | Greater than 0.68 |
| Medium | Greater than 0.38 |
| Low | Otherwise |

The simulated traffic is assigned directly to road edges and later used by the A* algorithm.

For visualization, approximately 1% of the high- and medium-traffic edges are randomly sampled and displayed as map markers.

**Note:** Traffic conditions are simulated and are not retrieved from live traffic services.

### 3. Dynamic Road Closures

The project simulates a road closure using a designated OpenStreetMap node ID representing the Zeinoddin Highway area.

The corresponding outgoing edges are marked as inaccessible:

```python
d["access"] = "no"
d["traffic"] = "blocked"
```

Both DFS and A* check edge accessibility and avoid blocked connections during route discovery.

This allows the system to explore alternative paths when selected road segments become unavailable.

### 4. Interactive Start and Destination Selection

The project uses `ipyleaflet` to create an interactive map with two draggable markers:

- **Origin:** Starting location, initially near Enghelab Square.
- **Destination:** Target location, initially near the university parking area.

Both markers can be moved to different geographic locations.

Their coordinates are then mapped to the nearest nodes in the road network.

The marker positions must be reprocessed by executing the corresponding notebook cells before running the pathfinding algorithms again.

### 5. Geographic Node Mapping

The geographic coordinates of each marker are converted into graph node IDs using OSMnx:

```python
nearest = ox.nearest_nodes(
    Graph,
    X=lon,
    Y=lat
)
```

These node IDs are used as the source and destination for the search algorithms.

## Pathfinding Algorithms

### 1. Depth-First Search (DFS)

A custom DFS implementation is developed using pure Python without external pathfinding libraries.

The algorithm uses a stack-based traversal strategy to explore the road network and discover a path between two nodes.

**Implementation details:**

- Stack-based LIFO exploration.
- Visited-node tracking.
- Blocked-edge filtering.
- Path reconstruction.
- Calculation of the resulting route's geographic distance.

DFS does not consider road length or traffic intensity when selecting its path.

As a result, it may discover a valid route that is significantly longer than necessary.

**Visualization:** Red polyline.

### 2. Traffic-Aware A* Search

A custom A* implementation is developed using Python's `heapq` priority queue.

The algorithm combines accumulated movement cost with a geographic heuristic:

\[
f(n) = g(n) + h(n)
\]

Where:

- \(g(n)\) is the accumulated traffic-weighted cost.
- \(h(n)\) estimates the remaining geographic distance to the destination.

#### Traffic-Weighted Edge Cost

Each road segment receives a cost based on its length and simulated traffic category:

\[
w(e) = L(e) \times P(e)
\]

Traffic penalties:

| Traffic Level | Cost Multiplier |
|---|---:|
| Low | ×1 |
| Medium | ×2 |
| High | ×4 |
| Blocked | Excluded |

For parallel edges between two nodes, the implementation selects the shortest accessible edge.

#### Geographic Heuristic

The heuristic uses geodesic distance calculated with `geopy`:

```python
from geopy.distance import geodesic

def h(node_a, node_b):
    ya, xa = Graph.nodes[node_a]["y"], Graph.nodes[node_a]["x"]
    yb, xb = Graph.nodes[node_b]["y"], Graph.nodes[node_b]["x"]

    return geodesic((ya, xa), (yb, xb)).km
```

The heuristic guides the search toward the destination while the accumulated cost accounts for road length and simulated traffic.

**Visualization:** Lime-green polyline.

## Route Evaluation

The project calculates two distance-related metrics for A* routes.

### Real Geographic Distance

The sum of the original road segment lengths:

\[
D_{\text{real}} = \sum_{e \in P} L(e)
\]

### Traffic-Weighted Cost

The accumulated road length after applying traffic penalties:

\[
D_{\text{weighted}} =
\sum_{e \in P} L(e) \times P(e)
\]

The second metric represents a traffic-weighted distance equivalent rather than actual travel distance or estimated travel time.

These measurements help illustrate the effect of traffic penalties on route selection.

## Interactive Map Visualization

The project uses `ipyleaflet` to visualize different aspects of the routing system.

The generated maps include:

- Simulated traffic intensity markers.
- DFS path visualization.
- Traffic-aware A* path visualization.
- Interactive origin and destination markers.
- Alternative routing behavior after road closures.

The routes are displayed using different colors:

| Element | Color |
|---|---|
| High Traffic | Red |
| Medium Traffic | Orange |
| DFS Route | Red |
| A* Route | Lime Green |

## Project Structure

```text
tehran-smart-route-planner/
│
├── README.md
├── university-distance.ipynb
├── requirements.txt
│
├── images/
│   ├── traffic-map.png
│   ├── dfs-route.png
│   └── astar-route.png
│
└── data/
    └── Tehran_2.osm.pbf  (not included)
```

The directory structure above describes the recommended repository organization.

The original OSM dataset is not included because of its potentially large file size. Users should obtain the dataset separately.

## Installation & Usage

### Prerequisites

- Python 3.x
- Jupyter Notebook
- Tehran OpenStreetMap `.osm.pbf` dataset

### 1. Clone the Repository

```bash
git clone https://github.com/VahhabBM/tehran-smart-route-planner.git

cd tehran-smart-route-planner
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Prepare the OSM Dataset

Obtain a Tehran road network dataset in `.osm.pbf` format and place it in the project directory.

Update the dataset path in the notebook if necessary.

### 4. Run the Notebook

```bash
jupyter notebook
```

Open `university-distance.ipynb` and execute the cells sequentially.

The notebook performs road network extraction, traffic simulation, road closure configuration, interactive marker initialization, and pathfinding.

## Limitations & Considerations

- Traffic conditions are synthetically generated and do not represent actual traffic measurements.
- The Gaussian model approximates spatial traffic intensity around a single geographic center.
- DFS does not optimize route length or traffic exposure.
- Interactive marker movement requires rerunning the relevant notebook cells.
- The current A* implementation uses different units for its heuristic (kilometers) and accumulated edge costs (meters), which should be standardized for a properly scaled heuristic.
- The current road closure implementation marks outgoing edges of the selected directed graph node.
- The current implementation is an academic prototype rather than a production-ready navigation system.

## Potential Applications

The techniques implemented in this project can be extended to:

- Traffic-aware navigation systems.
- Urban mobility analysis.
- Emergency vehicle routing.
- Delivery route optimization.
- Geographic network analysis.
- Transportation simulation.

## Contributors

- **Vahhab Bolandmanesh**
- **Mohammadmehdi Kazemi**

**University:** K. N. Toosi University of Technology

**Project Type:** Academic Team Project

**Geographic Focus:** Tehran, Iran

---

*Exploring the intersection of graph algorithms, geographic data, and intelligent route planning.*
