# DAY 10 — Detailed Routing, DRC and TritonRoute

> **RTL-to-GDS | Maze Routing | Design Rule Checking | Global Routing | Detailed Routing | PDN**

Day 10 focuses on turning the global routing plan into actual interconnect geometry. It introduces Lee's maze-routing algorithm, explains why design-rule checking is essential, and describes how TritonRoute uses route guides, access points and routing topology to connect nets while respecting manufacturing constraints.

---

## Contents

1. [Maze Routing with Lee's Algorithm](#1-maze-routing-with-lees-algorithm)
2. [Lee's Algorithm: Strengths and Limitations](#2-lees-algorithm-strengths-and-limitations)
3. [Design Rule Checking](#3-design-rule-checking)
4. [Global Routing, Detailed Routing and TritonRoute](#4-global-routing-detailed-routing-and-tritonroute)
5. [TritonRoute and Preprocessed Route Guides](#5-tritonroute-and-preprocessed-route-guides)
6. [Inter-guide Connectivity and Layer Routing](#6-inter-guide-connectivity-and-layer-routing)
7. [Connectivity Handling and Access Points](#7-connectivity-handling-and-access-points)
8. [Routing Topology Optimization](#8-routing-topology-optimization)
9. [Lab: Power Distribution Network and Routing](#9-lab-power-distribution-network-and-routing)
10. [Key Takeaways](#10-key-takeaways)

---

## 1. Maze Routing with Lee's Algorithm

Lee's algorithm is a grid-based path-search method used to find a connection between two terminals while avoiding blocked regions. It is commonly explained as a breadth-first search that expands outward from the source.

### How the algorithm works

1. Represent the routing area as a grid.
2. Start at the source pin and label reachable grid cells with increasing distance values.
3. Continue expanding the wavefront until it reaches the destination or no further cells can be visited.
4. Trace backward from the destination by following cells with decreasing labels to recover the path.

```text
Source pin
    |
    v
Expand wavefront across available grid cells
    |
    v
Reach destination (if a path exists)
    |
    v
Trace decreasing labels back to source
    |
    v
Shortest grid path
```

The search avoids obstacles by expanding only through available cells. If a route exists on the grid, the algorithm finds a path with the minimum number of grid steps.

---

## 2. Lee's Algorithm: Strengths and Limitations

Lee's algorithm is useful because it is systematic and finds an optimal grid path when a valid path exists. Its main drawback is the amount of computation and memory required for large routing grids.

- **Optimality:** finds the shortest route measured in grid steps.
- **Obstacle handling:** naturally avoids blocked cells.
- **Scalability:** can become expensive as the number of grid cells grows; its work and memory requirements scale with the explored grid.
- **Modern use:** practical routers often use more advanced search methods and optimization strategies, such as A*-style search, multi-terminal search, and rip-up-and-reroute.

The central idea remains important: explore legal routing space, identify a connection, and improve the route while satisfying physical constraints.

---

## 3. Design Rule Checking

Design Rule Checking (DRC) verifies that layout geometry complies with the manufacturing constraints defined by the process technology. A net can be logically connected and still be unsuitable for fabrication if its shapes violate these rules.

### Common routing-related rules

- Minimum metal width
- Minimum spacing between wires on the same layer
- Via dimensions and via-to-via spacing
- Required via enclosure by surrounding metal
- Cut-size restrictions
- End-of-line spacing, notch and antenna rules

Detailed routing must create connections that satisfy these geometric requirements. A DRC-clean result is therefore a key goal of the routing stage, not merely a final cosmetic check.

---

## 4. Global Routing, Detailed Routing and TritonRoute

Global routing and detailed routing solve different parts of the interconnect problem.

### Global routing

The global router assigns each net a coarse path through global routing cells (GCells). These paths are represented as route guides. Global routing estimates wirelength and manages congestion, but it does not create the final wire and via geometry.

### Detailed routing

The detailed router converts the guides into actual metal segments and vias. It must connect the required pins and terminals while observing track locations, layer restrictions and DRC rules.

### TritonRoute

TritonRoute is OpenROAD's detailed-routing engine. Its work includes interpreting global-route guides, finding legal connections, assigning wires to routing resources, and resolving local conflicts through cleanup and rerouting.

Configuration and routing considerations include:

- The allowed routing-layer range
- Track patterns and preferred routing directions
- Legal via choices and via-generation rules
- Congestion and rip-up-and-reroute handling
- DRC checking and cleanup

```text
Global Routing
  └── Coarse paths / route guides
          |
          v
Detailed Routing with TritonRoute
  └── Metal segments and vias
          |
          v
Connectivity and DRC checks
```

---

## 5. TritonRoute and Preprocessed Route Guides

TritonRoute uses the route guides created by global routing as guidance for detailed routing. The guides help keep the detailed solution aligned with the global plan, while the router may still need local flexibility to complete connections and resolve conflicts.

The guide-preprocessing sequence described in this module is:

```text
Initial route guides
        ↓
Guide splitting
        ↓
Guide merging
        ↓
Guide bridging
        ↓
Preprocessed route guides
```

The preprocessed guides are expected to have unit width and to follow the preferred direction for the relevant routing layer. For example, one layer may prefer vertical routing while another prefers horizontal routing.

Guide preprocessing prepares the coarse routing plan for use by the detailed-routing engine.

---

## 6. Inter-guide Connectivity and Layer Routing

### Inter-guide connectivity

Route guides belonging to a net must form a connected structure. Two guides are considered connected when either:

- They lie on the same metal layer and their edges touch, or
- They lie on adjacent metal layers and have a non-zero vertical overlap area.

Each unconnected terminal, such as a standard-cell pin, must also overlap a route guide. This ensures that the detailed router has a guided region through which it can reach the terminal.

### Panel-routing strategy

The routing method described for TritonRoute uses a mixed strategy:

- **Intra-layer parallel routing:** panels on the same metal layer can be processed in parallel.
- **Inter-layer sequential routing:** layers are processed one after another, with even- and odd-indexed panels handled in sequence.

This organization allows parallel work within a layer while preserving the ordering needed to manage routing interactions between layers.

---

## 7. Connectivity Handling and Access Points

The routing problem takes LEF, DEF and preprocessed route guides as inputs. The intended output is a detailed routing solution that connects the terminals while optimizing wirelength and via count, subject to guide, connectivity and design-rule constraints.

### Access Point (AP)

An Access Point is an on-grid location on a route-guide metal layer. It provides a possible connection point to a pin, an I/O port, or a routing segment on the same or another layer.

### Access Point Cluster (APC)

An Access Point Cluster groups access points derived from the same underlying object, such as a lower-layer segment, an upper-layer guide, a pin shape or an I/O port.

```text
Pin / I/O / lower-layer segment / upper-layer guide
                         |
                         v
                 Candidate access points
                         |
                         v
                 Access Point Cluster
                         |
                         v
             Build connectivity between clusters
```

Access points describe where a route can legally connect to a terminal or transition between routing resources. Grouping related access points into APCs gives the topology stage a set of connection targets.

---

## 8. Routing Topology Optimization

After the access point clusters for a net have been identified, TritonRoute determines a connection topology between them. The method described in this module uses pairwise costs and a minimum spanning tree (MST).

### Algorithm

1. For every pair of access point clusters `i` and `j`, calculate the connection cost:

   ```text
   cost[i,j] = dist(APC_i, APC_j)
   ```

2. Construct a minimum spanning tree from the clusters and their pairwise costs:

   ```text
   T = MST(APCs, COSTs)
   ```

3. Use the edges of the tree as the selected routing topology.

The MST provides a low-cost connection skeleton between the access point clusters. The detailed-routing engine then realizes those connections with legal track, wire and via geometry.

---

## 9. Lab: Power Distribution Network and Routing

The practical section covers construction of the Power Distribution Network (PDN) and the routing stages.

The lab topics listed for this module are:

1. Building the power distribution network
2. Connecting power straps to standard-cell power rails
3. Running routing

The PDN distributes power through the physical design. The lab sequence progresses from the larger power structures toward the rails that connect to standard cells, followed by routing work.

**Progress note:** The source notes that the last two labs of Module 4 are still in progress. Their step-by-step commands and results can be added once completed.

---

## 10. Key Takeaways

| Topic | Main point |
|---|---|
| Lee's algorithm | Uses wavefront expansion to find a shortest grid path around obstacles. |
| DRC | Checks layout shapes against manufacturing geometry rules. |
| Global routing | Creates coarse paths and route guides while considering congestion. |
| Detailed routing | Turns route guides into real wire and via geometry. |
| TritonRoute | Performs detailed routing while managing guide, connectivity and DRC constraints. |
| Preprocessed guides | Organize the global plan into guide regions suitable for detailed routing. |
| Access points | Identify legal locations for connecting pins and routing resources. |
| Access point clusters | Group related access points for topology planning. |
| MST topology | Supplies a low-cost connection structure between clusters. |
| PDN lab | Covers power rings/straps, standard-cell rails and routing activities. |

---

## Practical Flow Summary

```text
Understand maze routing
        ↓
Review manufacturing design rules
        ↓
Generate global-route guides
        ↓
Preprocess route guides
        ↓
Identify access points and clusters
        ↓
Optimize routing topology
        ↓
Perform detailed routing
        ↓
Check connectivity and DRC
        ↓
Build and route the PDN
```

---

## Conclusion

Day 10 connects routing theory with physical implementation. Lee's algorithm explains the basic search process behind maze routing, while DRC defines the geometric conditions a manufacturable layout must meet. Global routing supplies coarse guides, and TritonRoute uses those guides, access points, connectivity checks and topology optimization to build detailed interconnects.

The module also introduces the PDN lab, which extends the physical-design work from signal interconnects to the power structures and standard-cell rails. The remaining lab steps and results should be documented after those exercises are completed.
