---
title: "Quantum computing - Grovers Algorithm"
date: 2023-04-08T11:59:39+02:00
math: true
draft: true
---


As this is a very abstract statement, let us go through this using the famous [Grover algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm) which is a quantum search algorithm. Assume we have a database with $N$ entries, where we know one entry is the correct answer for our problem. We have a search operator $U_\omega$, which is a $N \times N$ diagonal matrix, i.e. with $N$ non-zero entries, which are all $1$, except the one entry for the correct answer, which is $-1$. Note that the algorithm can be extended to multiple correct answers, which is out of scope for now.

This algorithm itself then consists of the following steps, which are probably not understandable at first, but we will go through them using an example afterwards:
- Initialize a system of $N$ qubits to a equal superposition $|s \rangle$ over all states, i.e. $|s \rangle = \frac{1}{\sqrt{N}} \sum\_{x = 0}^{N-1} |x \rangle$
- Perform the Grover iteration $\frac{\pi}{4} \sqrt{N}$ times:
    - Apply the search operator $U_\omega$
    - Apply the Grover diffusion operator $U_s = 2|s\rangle \langle s| - \mathbb{I}_N$
- Measure the state $|s \rangle$, it should yield the correct answer

The first important thing to notice here is, that $U_s$, contains the [outer product](https://en.wikipedia.org/wiki/Outer_product) of our current state $|s\rangle$, which is an $N \times N$ matrix, which has not only non-zero entries on the diagonal axis.

For illustration, let us go through this algorithm using the very simple example, where we the database has only four entries, i.e. $N = 4$ which we can represent using two qubits, using the basis as initially defined. Furthermore let us assume that the correct answer is the first entry, i.e. the state $|00 \rangle$, therefore the Grover search operator is given by
$$
    U\_\omega = \begin{pmatrix} -1 & 0 & 0 & 0 \\\\ 0 & 1 & 0 & 0 \\\\ 0 & 0 & 1 & 0 \\\\ 0 & 0 & 0 & 1 \end{pmatrix}
$$
To create the equal superposition for our state vector $|s \rangle$ we start with two qubits in the $|0 \rangle$ state and apply the Hadamard gate $H$ on both of them, and create the product state of them, which yields $|s \rangle = \frac{1}{2}(|00 \rangle + |01 \rangle + |10 \rangle + |11 \rangle)$.

Having the equal superposition initial state, we apply the search operator on this state and obtain
$$
    |s' \rangle = U\_\omega |s \rangle = \frac{1}{\sqrt{2}} \begin{pmatrix} -1 & 0 & 0 & 0 \\\\ 0 & 1 & 0 & 0 \\\\ 0 & 0 & 1 & 0 \\\\ 0 & 0 & 0 & 1 \end{pmatrix} \begin{pmatrix} 1 \\\\ 1 \\\\ 1 \\\\ 1 \end{pmatrix} = \frac{1}{\sqrt{2}} \begin{pmatrix} -1 \\\\ 1 \\\\ 1 \\\\ 1 \end{pmatrix}
$$
where the sign of the "correct" answer has been flipped, and the others have been left unchanged. Now we form the outer product of the state at the beginning of the operation, i.e.
$$
    |s \rangle \langle s| = \frac{1}{2} \begin{pmatrix} -1 \\\\ 1 \\\\ 1 \\\\ 1 \end{pmatrix} \begin{pmatrix} -1 & 1 & 1 & 1 \end{pmatrix} = \frac{1}{2} \begin{pmatrix} 1 & -1 & -1 & -1 \\\\ -1 & 1 & 1 & 1 \\\\ -1 & 1 & 1 & 1 \\\\ -1 & 1 & 1 & 1 \end{pmatrix}
$$
such that the diffusion operator is given by
$$
    U_s = 2|s\rangle \langle s| - \mathbb{I}_4 = \begin{pmatrix} 0 & -1 & -1 & -1 \\\\ -1 & 0 & 1 & 1 \\\\ -1 & 1 & 0 & 1 \\\\ -1 & 1 & 1 & 0 \end{pmatrix}
$$
after applying this operation to the state $|s'\rangle$ we obtain
$$
    U_s |s'\rangle = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix} \begin{pmatrix} -\frac{1}{\sqrt{2}} \\\\ \frac{1}{\sqrt{2}} \end{pmatrix} = \begin{pmatrix} \frac{1}{\sqrt{2}} \\\\ -\frac{1}{\sqrt{2}} \end{pmatrix}
$$

... Finalize explanation of Grover's algorithm ...

