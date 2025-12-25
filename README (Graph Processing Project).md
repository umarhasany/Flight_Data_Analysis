# Flight Graph Analysis (Spark + GraphFrames) — DPA Project 3

Course: Data Processing and Analytics (DPA) — International Master DISS UCBL 25/26  
Project: Graph Processing Assignment (Flights)

This repository runs a flight-network graph analysis inside a Dockerized Spark + Jupyter environment.  
Core required computations are implemented using **native Spark DataFrame operations**. GraphFrames is used **only for optional validation/comparison**, per the assignment note.

---

## Repository Structure (recommended)

```
DPA-Project-3/
├─ container/
│  ├─ Dockerfile
│  ├─ compose.yml
│  └─ mnt/
│     ├─ flight_analysis.ipynb
│     └─ Graphframes.ipynb
├─ .data/
│  ├─ 2009.csv
│  ├─ 2010.csv
│  ├─ ...
│  └─ 2018.csv
└─ README.md
```

### Where files should live (important)

- Docker files in `container/`:
  - `container/compose.yml`
  - `container/Dockerfile`

- Notebooks in `container/mnt/` so they appear inside Jupyter:
  - `container/mnt/flight_analysis.ipynb`
  - `container/mnt/Graphframes.ipynb` (optional)

- Large CSV datasets in `.data/` (recommended):
  - `.data/2009.csv ... .data/2018.csv`

---

## How mounts work (what the container sees)

A common `compose.yml` mounts setup is:

- `container/mnt/` → `/home/jovyan/` inside the container
- `.data/`         → `/home/jovyan/input/` inside the container

So your CSV path in the notebook should be one of:

- If CSVs are in `.data/`:
  - `/home/jovyan/input/2009.csv`

- If CSVs are in `container/mnt/`:
  - `/home/jovyan/2009.csv`

Recommended: use `.data/` → `/home/jovyan/input/`.

---

## Requirements

- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- Enough RAM for Spark (10–12 GB recommended if available)

---

## Launching the environment

Open a terminal **in the `container/` folder** (important: mounts in `compose.yml` are relative to this folder):

```bash
cd container
docker compose up --build
```

Jupyter will be available in your browser (the URL/token is printed in the container logs).  
Commonly:

- `http://localhost:8888`
- or `http://localhost:8888/lab?token=...`

Stop the environment:

```bash
CTRL+C
docker compose down
```

---

## Project Tasks Covered

### 1) Build the flight graph
- Nodes: airports
- Directed edges: routes `src -> dst`
- Edge weight: number of flights on that route

### 2) Degree metrics (native Spark)
- In-degree, out-degree, total degree via `groupBy` + aggregations.

### 3) Triangle counting (native Spark)
- Treat graph as **undirected** for triangles.
- Triangles via wedge creation + wedge closing joins (no GraphFrames triangleCount).

### 4) Centrality (native Spark)
- Eigenvector centrality via **power iteration** with joins + aggregations.

### 5) PageRank (native Spark)
- Iterative PageRank implemented with Spark DataFrames.
- Weighted edges and dangling nodes handled.
- No GraphFrames PageRank used for the core implementation.

### 6) Group of most connected airports (native Spark)
- Finds a densely connected airport group using a native Spark method (dense core / dense subgraph logic).
- Reports:
  - the group
  - most connected airports **within the group** by internal weighted degree
  - strongest internal links inside the group

### 7) Compare metrics
- Join results into one comparison table:
  - degree
  - triangle participation
  - eigenvector centrality
  - PageRank

### 8) Flight Traffic Heatmap
- Draws a heatmap (origin × destination airport matrix) showing flight volumes between airports.
- Cell intensity represents the number of flights (or total edge weight) on that route.
- Makes the busiest routes and overall traffic patterns easy to identify at a glance.
- GraphFrames may be used **only for optional sanity-check comparison**, not for the main computation.

---

## Typical data paths inside the notebook

If datasets are stored in `.data/`:

```python
path = "/home/jovyan/input/2009.csv"
```

If datasets are stored in `container/mnt/`:

```python
path = "/home/jovyan/2009.csv"
```

---

## Troubleshooting

### Kernel busy forever / Spark crashes (Py4J errors, JVM resets)
Usually memory pressure during heavy joins or iterative steps.

Try:
1) Reduce shuffle partitions:
```python
spark.conf.set("spark.sql.shuffle.partitions", "16")
spark.conf.set("spark.default.parallelism", "16")
```

2) Validate on a single year CSV first, then scale up.

3) Ensure Docker/WSL has enough memory allocated.

### Windows + WSL2 memory cap
If using Docker Desktop on Windows with WSL2, WSL can cap memory for Docker.
You can set WSL memory in `C:\Users\<you>\.wslconfig`, then restart WSL + Docker.

---

## Files

- `container/mnt/flight_analysis.ipynb`  
  Main analysis notebook

- `container/mnt/Graphframes.ipynb`  
  GraphFrames checks (optional validation only)

- `container/compose.yml`, `container/Dockerfile`  
  Environment definition and dependencies
