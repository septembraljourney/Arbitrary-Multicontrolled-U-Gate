# Construction of Multi-Controlled U Gate

This note summarizes how to build an **n-controlled single-qubit unitary gate** $(C^n U)$ using standard quantum gates (1-qubit rotations, CNOT, Toffoli, and a phase).

---

## 1. Controlled-U from Decomposition  $U = e^{i\alpha} A X B X C$

If a given single-qubit unitary $U \in U(2)$ can be decomposed as
$$U = e^{i\alpha} A X B X C$$
where $A, B, C$ are single-qubit unitaries and $X$ is the Pauli-X gate, then a **controlled-U** (control = first qubit, target = second) can be implemented as:
$$CU = (P(\alpha)\otimes I)\; (I \otimes A)\; CX\; (I \otimes B)\; CX\; (I \otimes C)$$
where $P(\alpha) = \mathrm{diag}(1, e^{i\alpha})$ is a phase gate on the control qubit. This is easily observed by the following circuit diagram:
<pre style="word-wrap: normal;white-space: pre;background: #fff0;line-height: 1.1;font-family: &quot;Courier New&quot;,Courier,monospace">                         ┌──────┐
q_0: ───────■─────────■──┤ P(α) ├
     ┌───┐┌─┴─┐┌───┐┌─┴─┐└┬───┬─┘
q_1: ┤ C ├┤ X ├┤ B ├┤ X ├─┤ A ├──
     └───┘└───┘└───┘└───┘ └───┘  </pre>

This circuit uses **two CNOTs** and a **conditional phase** on the control.

---

## 2. Decomposition of Any  $U \in U(2)$

Every one-qubit unitary has the following decomposition:
$$U = e^{i\alpha} R_z(\phi) R_y(\theta) R_z(\lambda)$$
where
$$R_z(\gamma) = e^{-i\gamma Z/2}, \qquad R_y(\beta) = e^{-i\beta Y/2}.$$

Define
$$A = R_z(\phi) R_y(\tfrac{\theta}{2}), \quad
B = R_y(-\tfrac{\theta}{2}) R_z(-\tfrac{\phi+\lambda}{2}), \quad
C = R_z(\tfrac{\lambda - \phi}{2}).$$

Using the fact that $\;X^2 = I\;$ and $\;X R_*(\eta)X = R(-\eta)\;$, one can verify algebraically that $R_z(\phi) R_y(\theta) R_z(\lambda) = A X B X C$, which gives the desired form $U = e^{i\alpha} A X B X C$.

**In Qiskit:**

The one-qubit unitary $U(\theta,\phi,\lambda) = R_z(\phi)R_y(\theta)R_z(\lambda)$

is implemented by the built-in gate `U(θ, φ, λ)`.

Combining these facts gives a constructive **Controlled-U** circuit using only `u` (1-qubit), `cx`, and `p` (phase) gates.

---

## 3. Multi-Controlled U Gate  $C^n U$

Once $CU$ is available, higher controls can be added recursively:

1. Use ancilla qubits to store the logical AND of all control qubits.
This is achieved with a ladder of **Toffoli gates** $C^2 X$.
2. Use that ancilla as a **single control** for the $CU$ built above.
3. Uncompute the ancilla chain (apply the Toffolis in reverse).

For example, when $n=5$, we can construct the following circuit:
<pre style="word-wrap: normal;white-space: pre;background: #fff0;line-height: 1.1;font-family: &quot;Courier New&quot;,Courier,monospace">                          ░       ░                     
c_0: ──■──────────────────░───────░──────────────────■──
       │                  ░       ░                  │  
c_1: ──■──────────────────░───────░──────────────────■──
       │                  ░       ░                  │  
c_2: ──┼────■─────────────░───────░─────────────■────┼──
       │    │             ░       ░             │    │  
c_3: ──┼────┼────■────────░───────░────────■────┼────┼──
       │    │    │        ░       ░        │    │    │  
c_4: ──┼────┼────┼────■───░───────░───■────┼────┼────┼──
     ┌─┴─┐  │    │    │   ░       ░   │    │    │  ┌─┴─┐
w_0: ┤ X ├──■────┼────┼───░───────░───┼────┼────■──┤ X ├
     └───┘┌─┴─┐  │    │   ░       ░   │    │  ┌─┴─┐└───┘
w_1: ─────┤ X ├──■────┼───░───────░───┼────■──┤ X ├─────
          └───┘┌─┴─┐  │   ░       ░   │  ┌─┴─┐└───┘     
w_2: ──────────┤ X ├──■───░───────░───■──┤ X ├──────────
               └───┘┌─┴─┐ ░       ░ ┌─┴─┐└───┘          
w_3: ───────────────┤ X ├─░───■───░─┤ X ├───────────────
                    └───┘ ░ ┌─┴─┐ ░ └───┘               
  t: ─────────────────────░─┤ U ├─░─────────────────────
                          ░ └───┘ ░                     </pre>

The resulting circuit acts on computational basis as

$C^n U\,\ket{x_1,\dots,x_n}\ket{y}
= \ket{x_1,\dots,x_n} U^{x_1\dots x_n} \ket{y}$

Thus, with:

- 1-qubit `U(θ, φ, λ)` gates for rotations,
- 2-qubit `cx` gates,
- 3-qubit `ccx` (Toffoli) gates,
- and a single control phase `p(α)`,

we can efficiently construct **multi-controlled U gates** $C^n U$ for any $U \in U(2)$.

---

### References

- M. A. Nielsen & I. L. Chuang, *Quantum Computation and Quantum Information*, §4.3.1
- A. Barenco *et al.*, *Elementary gates for quantum computation*, Phys. Rev. A 52 (1995) 3457, §IV.A