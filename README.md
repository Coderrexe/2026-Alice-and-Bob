# Optimized Quantum Cat Reed-Solomon and Repetition Stablizers - A Simplified Tornado

The main code for building our novel method for quantum Reed-Solomon and tornado is in `main.ipynb`.

Our solutions to the close-ended problems 1 to 4 are in `1-challenge.ipynb`. The original code for the challenge is in `challenge/`.

## Classical RS Error Correction
In classical RS error correction, we encode a message 

$$m=\begin{bmatrix} m_0 & m_1 & m_2 & m_3 \end{bmatrix}$$

 into a function $f(x)=m_0 x^0 + m_1 x^1 + m_2 x^2 + m_3 x^3$. We then sample redundant points on the function so that even when some function values, which are passed as the code, are lost, the function, and hence the message bits, could still be recovered.

### Finite Galois Field
We apply the arithmetic logic of $GF(2^m)$ to extract the polynomial from our input message.
For example, $GF(8)$ is a field with elements $$\{0,1,2,3,4,5,6,7\}$$ each representing a polynomial of degree $\text{deg}\leq 2$ and operations similar to polynomials with $f(x)=m_0 x^0 + m_1 x^1 + m_2 x^2 + m_3 x^3 $.

For example, $3+3$ in $GF(8)$ should be interpreted as the addition of polynomials, $$(x+1)+(x+1)=(1+1)x+(1+1)=0x+0=0.$$ Multiplication works similarly. The identity $$x^3=x+1$$ ensures that this forms a field under multiplication, where the resulting polynomial may have a degree larger than 2. 

### Generators

$2$ is a primitive element which, when self-multiplied, genereates every non-zero element of the field. We use this to generate $x^1$ and then calculate the other powers, which are combined into the symmtrical generator $G_{sym}$. In the case of the $[[21,12,4]]$ code, which we should use implicitly in this README,

$$ 
G_{sym} = \begin{bmatrix}
1 & 1 & 1 & 1 & 1 & 1 & 1 \\
1 & 2 & 4 & 3 & 6 & 7 & 5 \\
1 & 4 & 6 & 5 & 2 & 3 & 7 \\
1 & 3 & 5 & 4 & 7 & 2 & 6
\end{bmatrix} = \begin{bmatrix} x^0 \\ x^1 \\ x^2 \\ x^3 \end{bmatrix}^T.
$$

Then, we compute the code with
$$m\cdot G_{sym}=c$$
in other words, $G_{sym}$ transforms the 4-digit $m$ into the 7-digit codeword $c$.

## Quantum RS Error Correction
The goal is to implement $G_{sym}$ in a quantum computer. We first transform each entry into $G_{bin}$, a $3\times3$ matrix of binaries. Then we find the encoder $G_s=\left[ I \mid P \right]$, the Row Reduced Echelon Form of such a matrix. Here, $P$ is the parity matrix. The $1$'s in $P$ gives us the locations to insert the $CX$ gates. We may also construct the decoder $H=\left[ P^T \mid I \right]$, which gives the syndrome. Then we use a pre-calculated lookup table to determine the position of errors corresponding to the syndrome.

Note that the [existing](http://arxiv.org/pdf/quant-ph/9910059) quantum implementation of the quantum RS code requires discrete fourier transform. But this is not implementable with Stim as it required non-Clifford gates. This is also redundant for cat-qubits because it also corrects for phase-flip errors, which is suppressed. Even such an approach is implemented, it does not leverage the advantage of cat-qubits.

## Tornado Codes
We took inspiration from classical Tornado codes, which is multiple layeres of LDPC and a final layer of RS. Then, we apply a simplified idea of that in quantum circuits, namely one layer of repetition code plus one layer of RS.

## Final Plot
The graph below represents the culmination of this research. We compare our three contenders across a range of physical error rates ($p$).

### Methods to compare
1.  **Repetition Code (Blue):** The standard "safe" baseline.
    * **Strategy:** Brute force redundancy.
    * **Result:** Reliable, but expensive (low information density).
2.  **Reed-Solomon Code (Orange):** The "High-Density" solution.
    * **Strategy:** Algebraic efficiency.
    * **Result:** Excellent storage density, but higher error rates than Repetition because it lacks "redundancy padding."
3.  **Tornado Code (Green):** The "Hybrid" solution.
    * **Strategy:** Concatenation (Layered Defense).
    * **Result:** **The Winner.**

**Conclusion:**
By combining the **robustness** of the Repetition code with the **algebraic power** of the RS code, the Tornado architecture achieves a logical error rate orders of magnitude lower than either parent code could achieve alone.

<img src="images/plot.png"/>
