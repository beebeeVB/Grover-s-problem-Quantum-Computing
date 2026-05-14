# Grover's Algorithm for Graph Coloring

Solving the planar graph coloring problem using Grover's quantum search algorithm, implemented in Qiskit.

## What this does

Graph coloring is the problem of assigning colors to nodes in a graph such that no two adjacent nodes share the same color. This is NP-hard classically. This project encodes the coloring constraint as a quantum oracle and uses Grover's algorithm to search the solution space with quadratic speedup.

Each node is encoded as 2 qubits, allowing up to 4 colors (`00` = red, `01` = blue, `10` = green, `11` = yellow). Edge qubits and a single ancilla qubit are used to check and invert the oracle. The diffusion operator amplifies the probability of valid colorings.

## Files

| File | Description |
|------|-------------|
| `main.ipynb` | Generalized solver with adaptive iteration loop — runs until a valid coloring is found |
| `Three_coloring_problem.ipynb` | Fixed 3-node triangle graph, fixed iteration count, good starting point |
| `coloring_node_using_grovers.ipynb` | 4-color version with graph visualization using NetworkX |
| `Generalized_circuit.ipynb` | Optimized oracle circuit with cleaned-up color-check gate construction |
| `Solving_the_Planar_Graph_Coloring_Problem...pdf` | Full write-up of the approach and results |

## Qubit layout

For a graph with `n` nodes and `e` edges:

- **Node qubits**: `2n` qubits (2 per node, encoding one of 4 colors)
- **Edge qubits**: `e` qubits (one per edge, used by the oracle to flag color conflicts)
- **Ancilla**: 1 qubit (phase kickback target, initialized to `|−⟩`)

Total: `2n + e + 1` qubits

## How to run

### Prerequisites

```bash
pip install qiskit qiskit-ibm-provider qiskit-aer networkx matplotlib numpy
```

### Basic usage

Open `main.ipynb` and set your graph at the top:

```python
number_of_nodes = 4
edge_connections = [[0,1],[0,2],[0,3],[1,2],[1,3],[2,3]]  # complete graph K4
shots = 1
```

The loop runs Grover iterations, incrementing the iteration count by a factor of `1.2` each round until a valid coloring is found. Output includes the iteration count, the measurement result, and a NetworkX visualization of the colored graph.

### Example graphs to try

```python
# Triangle
number_of_nodes = 3
edge_connections = [[0,1],[0,2],[1,2]]

# Pentagon
number_of_nodes = 5
edge_connections = [[0,1],[1,2],[2,3],[3,4],[0,4]]

# 7-node graph
number_of_nodes = 7
edge_connections = [[0,1],[0,4],[0,5],[1,2],[1,5],[1,6],[2,3],[2,6],[3,4],[3,5],[3,6],[4,5],[5,6]]
```

## Oracle design

The oracle uses an optimized `color_check` gate to flag edges where both endpoints share a color. For each edge `(u, v)`, the gate checks all cases where node `u` and node `v` have the same 2-bit color encoding and flips the corresponding edge qubit if a conflict exists.

The inverse (`color_check_inverse`) uncomputes the edge qubits after the ancilla phase kickback, keeping the circuit clean for repeated oracle calls.

The diffusion operator (Grover inversion about the mean) then amplifies the amplitudes of valid colorings.

## Running on IBM hardware

IBM backend calls are present in the notebooks but commented out. To use:

```python
from qiskit_ibm_provider import IBMProvider
provider = IBMProvider()
backend = provider.get_backend('ibmq_qasm_simulator')
job = execute(qc, backend=backend, shots=10000)
```

Replace `'ibmq_qasm_simulator'` with a real device name from your IBM Quantum account.

## Notes

- The simulator backend (`BasicAer` or `Aer`) is used by default
- `shots = 1` in the adaptive loop — the circuit re-runs until the single measurement lands on a valid coloring
- For large graphs, qubit count grows as `2n + e + 1` — 7 nodes with 7 edges already requires 22 qubits
- The optimized color-check gate uses fewer MCT (multi-controlled Toffoli) gates than the naive version; both implementations are in the codebase with the naive version commented out
