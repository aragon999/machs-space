---
title: "Quantum Computing Introduction"
date: 2023-02-06T20:59:33+01:00
math: true
draft: true
---

## Introduction

## Mathematical introduction of classical bits

Unfortunately, the shortest language is sometimes math, therefore we will not get around with some basic math. I will start with a basic introduction and definition of the notation but will try to use the formulas as additional information and explain everything in words.

Let us start with the [classical bit](https://en.wikipedia.org/wiki/Bit). A bit has two distinguishable values, which are usually labeled $0$ and $1$. Using a combination of several bits can be used to encode arbitrary information, where the encoding needs to be specified. Take for example the Unicode encoding({{< ref "posts/whats-the-max-valid-length-of-an-emoji#unicode-encoding" >}}), which defines a mapping from characters to bits.

We will introduce the physical [bra-ket notation](https://en.wikipedia.org/wiki/Bra%E2%80%93ket_notation), which makes it easier to express mathematical operations and label the ket values of a bit $|0 \rangle$ and $|1 \rangle$.

The values of a bit describe two distinguishable states, which makes it possible to express them as basis vectors of a two-dimensional vector space. Using the two unit vectors of $\mathbb{R}^2$ we can express them as:
$$
    |1 \rangle = \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} \quad |0 \rangle = \begin{pmatrix} 0 \\\\ 1 \end{pmatrix}
$$
But this is only one representation of these vectors, where the important property is that they are "orthogonal to each other", meaning that the [dot product](https://en.wikipedia.org/wiki/Dot_product) vanishes. We could also use any other representation, that fulfills this condition, but the unit vectors of the $\mathbb{R}^2$ are the simplest choice, so why make life more complicated than it already is? The dot product in the bra-ket notation involves the bra, meaning the complex conjugated and transposed (daggered) of the ket, for example, 
$$
    \langle 1 | = | 1 \rangle^{\dagger} = (| 1 \rangle^\*)^T = \left[ \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} ^\* \right]^T = \begin{pmatrix} 1 & 0 \end{pmatrix}
$$
where the $\cdot^\*$ means that we [complex conjugate](https://en.wikipedia.org/wiki/Complex_conjugate) the ket and $\cdot^T$ means that we [transpose](https://en.wikipedia.org/wiki/Transpose) the vector. Note that in this case we only have real numbers such that the complex conjugation will have no effect, but later, when going from the classical bits to qubits, this will become important.

Using this definition we can write the dot product of two basis vectors as
$$
    \langle 0 | 1 \rangle = \begin{pmatrix} 0 & 1 \end{pmatrix} \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = 0 * 1 + 1 * 0 = 0
$$
and as it is zero (vanishing), we say the two vectors are orthogonal to each other.

Of course we know that with a single bit, we can only represent the state "on" or "off", but the power of computing comes because we combine multiple bits into a larger data unit. For example, the simplest extension is to use two bits, where we can have the following four combinations $|00 \rangle$, $|01 \rangle$, $|10 \rangle$, and $|11 \rangle$. To represent these states as vectors, we would need for example the four unit-vectors of $\mathbb{R}^4$, i.e.
$$
    |00 \rangle = \begin{pmatrix} 1 \\\\ 0 \\\\ 0 \\\\ 0 \end{pmatrix} \quad |01 \rangle = \begin{pmatrix} 0 \\\\ 1 \\\\ 0 \\\\ 0 \end{pmatrix} \quad
    |10 \rangle = \begin{pmatrix} 0 \\\\ 0 \\\\ 1 \\\\ 0 \end{pmatrix} \quad |11 \rangle = \begin{pmatrix} 0 \\\\ 0 \\\\ 0 \\\\ 1 \end{pmatrix}
$$
where the order of which unit vector is assigned to which combination is arbitrary, but it is advantageous to use this representation, as we will later see. This concept can of course be generalized to $n$ bits, representing the $\mathbb{R}^{2^n}$ vector space. Therefore for more than two bits it is already very tedious to write out the representation as the unit vectors, for example a standard byte has eight bits, meaning each state can be expressed as a unit vector of the $\mathbb{R}^{256}$, i.e. a vector with $256$ rows. This is one of the reasons why we use labels, like $|10010010\rangle$.

Using these definitions we can define operations, like the NOT operation as a matrix. For example the NOT operation $N$ is for the two-dimensional case given by the matrix
$$
    N = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix}
$$
since using [matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication) (row times column) yields
$$
    N |1 \rangle = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix} \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = \begin{pmatrix} 0\*1 + 1\*0 \\\\ 1\*1 + 0\*0 \end{pmatrix} = \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = |0 \rangle
$$
$$
    N |0 \rangle = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix} \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = \begin{pmatrix} 0\*0 + 1\*1 \\\\ 1\*0 + 0\*1 \end{pmatrix} = \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = |1 \rangle
$$
the inversion of the bit. This is, of course, a well-known operation in programming, i.e. a standard not operation on a boolean value, `!true = false` and `!false = true`.

## Why Qubits?

Why the mathematical formulation of the last section is needed might be unclear at first, but it will serve as a basis for the extension of classical bits to qubits, which will be explained in this section.

First note that we can also write for example the $|0 \rangle$ and $|1 \rangle$ state as
$$
    |0 \rangle = 1 |0 \rangle + 0 |1 \rangle \quad |1 \rangle = 0 |0 \rangle + 1 |1 \rangle
$$
or in general we can write an arbitrary state $|\Phi \rangle$, again which is just a label, as
$$
    |\Phi \rangle = \alpha |0 \rangle + \beta |1 \rangle
$$
where for a classical bit either $\alpha = 1$ and $\beta = 0$ or vice versa $\alpha = 0$ and $\beta = 1$. Other values are not allowed for classical bits.

Qubits generalize this concept, by allowing the coefficients $\alpha$ and $\beta$ to be an arbitrary complex number. Typically, one imposes the condition $| \alpha |^2 + | \beta |^2 = 1$, which allows us to interpret the coefficients $\alpha$ and $\beta$ as [probability amplitudes](https://en.wikipedia.org/wiki/Probability_amplitude) of the states $|0 \rangle$ and $|1 \rangle$. When measuring the result of this qubit, it will be the value with the largest coefficient. This concept can be of course extended to an arbitrary amount of qubits, for example for the case of two qubits we have
$$
    |\Phi_1 \Phi_2 \rangle = \alpha |00 \rangle + \beta |01 \rangle + \gamma |10 \rangle + \delta |11 \rangle
$$
where the sum of the absolute values of all complex coefficients is normalized to $1$, i.e. $| \alpha |^2 + | \beta |^2 + | \gamma |^2 + | \delta |^2 = 1$. Again when measuring such a qubit, the result will be the state with the largest absolute value of the coefficient.

As the coefficients can take infinitely many different values, already two qubits can have infinitely many possible combinations, this is also usually referred to as superposition. But in the end, as the final answer of the measurement will be the state with the largest probability amplitude. Let us assume that a measurement of a two-qubit system yields the result $|01 \rangle$, then we would write
$$
    |\Phi_1 \Phi_2 \rangle = |01 \rangle = 0 |00 \rangle + 1 |01 \rangle + 0 |10 \rangle + 0 |11 \rangle
$$
meaning $\beta = 1$, being a non-complex real value, and all other coefficients are $0$. There exists no other possible solution, as all basis states are orthogonal to each other. This is commonly referred to as the collapse of the wave function, as the superposition of the states, from before collapses to a single state. After the measurement, there exists no possibility to obtain the values of the coefficients before the measurement.

When we apply an operation as the NOT operation $N$ on the single qubit state $|\Phi \rangle$, we find
$$
    N |\Phi \rangle = N \left( \alpha | 0 \rangle + \beta | 1 \rangle \right) = \alpha | 1 \rangle + \beta | 0 \rangle = \beta | 1 \rangle + \alpha | 0 \rangle = \alpha' | 1 \rangle + \beta' | 0 \rangle = |\Phi' \rangle
$$
meaning that the NOT operation exchanges the probability amplitudes of the two states, or mathematically the new probability amplitudes are given by $\alpha' = \beta$ and $\beta' = \alpha$. For these new coefficients, the condition that the sum of squares of all absolute values is $1$ is still true, such that these can be interpreted as probability amplitudes. In general, when an operation maps to the same space, i.e. we do not need a new state to represent the result, and the sum of the squares of absolute values of the coefficients is preserved, it is called a unitary operation. Meaning when we perform unitary operations on a valid qubit state, we can be sure that the result will be a valid qubit state as well.

A general, valid operation in the context of quantum computing always involves the manipulations of the coefficients in such a way that the result is still a valid qubit state.

## Qubits in more detail - entanglement

The probability properties of qubits alone, as described in the previous section, would not enable us to do advanced calculations. But there exists another very important concept in quantum computing, the entanglement of qubits. Using entanglement one has an advanced way of communicating information, with the important feature, that the information is only determined at the point of measurement.

If not familiar with quantum entanglement this probably sounds very abstract and confusing, but we will go through the details of the statement using an example, the Bell state $| \Phi^+ \rangle$. To put a two-qubit system into this state, the probability amplitudes of the two-qubit system have to be "initialized" (we will speak later about how to do that), such that $\alpha = \delta = 1/\sqrt(2)$ and $\beta = \gamma = 0$, i.e.
$$
    | \Phi^+ \rangle = \frac{1}{\sqrt{2}} \left(|00 \rangle + |11 \rangle\right)
$$
therefore when measuring such a state we obtain either of the two states $|00 \rangle$ or $|11 \rangle$, with equal probability of $| \alpha |^2 = | \delta |^2 = 1/2$. Remember that this state has been built from two qubits, which can be separated and treated independently. When we now measure the value of one qubit independently of the other, we know that the other qubit has to have the same value, as the measured one, since the mixed states $|01 \rangle$ and $|10 \rangle$ are not present in the state and the wave function has collapsed to a single state, i.e. either $|00 \rangle$ or $|11 \rangle$.

## Operations in Quantum computing (gates)

