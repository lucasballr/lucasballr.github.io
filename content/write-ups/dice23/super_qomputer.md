---
title: "rev/super_qomputer"
date: 2023-01-27T19:46:10-08:00
draft: false
---

This challenge gave you a qasm file which creates a quantum circuit.
This circuit was designed to just output a 1 or a 0 depending on the position.
All you had to do was read the superposition of the qubits and the resulting
string would be a bunch of 1's and 0's that translate to the ascii version of
the flag. It helped to run these things one at a time cause some stuff took extra
time so I did it in a jupyter notebook. Here's the script:

```python
import numpy as np
from qiskit import QuantumCircuit
from qiskit import Aer, transpile, execute, assemble
from qiskit.tools.visualization import plot_histogram, plot_state_city
import qiskit.quantum_info as qi

qc = QuantumCircuit.from_qasm_file("./challenge.qasm")
simulator = Aer.get_backend('aer_simulator')
job = execute(qc, simulator)
result = job.result()
print(result)
qc.measure_all()
qobj = assemble(qc)
result = simulator.run(qobj).result()
counts = result.get_counts()
plot_histogram(counts)
print(counts)
```

And I got this back:

```bash
'0000000000000000000000000000000000000000000000000000000000000000000000000
110010001101001011000110110010101111011011000110110110001101001011001100110
011001101111011100100110010000101101011101000110100001100101001011010110001
001101001011001110010110101110001011101010110000101101110011101000111010101
101101001011010110010001101111011001110010110100110001001110010110010100110
011011001100011010101111101'
```

This is just the flag in binary
