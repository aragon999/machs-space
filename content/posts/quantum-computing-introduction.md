---
title: "Quantum Computing Introduction"
date: 2023-03-23T20:59:33+01:00
math: true
draft: true
---

## Introduction

In this article I try to give an introduction to quantum computing. My goal is to explain some important concepts of quantum computing, in a different way it is mostly introduced. As a disclaimer, I am not a quanum computing expert, as I am not working directly in this field, but rather do theoretical particle physics. So if you find something which might be incorrect or is explained in a bad way, please let me know. Furthermore since I am a theoretical phyisist, I will mostly speak about the theoretical concepts behind quantum computing and not so much about the realization of a quantum computer.

## Mathematical introduction of classical bits

Unfortunately, the shortest language is sometimes math, therefore we will not get around with some basic math. I will start with a basic introduction and definition of the notation but will try to use the formulas as additional information and explain everything in words.

Let us start with the [classical bit](https://en.wikipedia.org/wiki/Bit). A bit has two distinguishable values, which are usually labeled $0$ and $1$. Using a combination of several bits can be used to encode arbitrary information, where the encoding needs to be specified. Take for example the [Unicode encoding]({{< ref "posts/whats-the-max-valid-length-of-an-emoji#unicode-encoding" >}}), which defines a mapping from characters to bit combinations.

We will introduce the physical [bra-ket notation](https://en.wikipedia.org/wiki/Bra%E2%80%93ket_notation), which makes it easier to express mathematical operations and label the ket values of a bit $|0 \rangle$ and $|1 \rangle$.

The values of a bit describe two distinguishable states, which makes it possible to express them as basis vectors of a two-dimensional vector space. Using the two unit vectors of $\mathbb{R}^2$ we can express them as:
$$
    |0 \rangle = \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} \quad |1 \rangle = \begin{pmatrix} 0 \\\\ 1 \end{pmatrix}
$$
But this is only one representation of these states, where the important property, for them to be distinguishable, is that they are "orthogonal to each other", meaning that the [dot product](https://en.wikipedia.org/wiki/Dot_product) vanishes. Although other representations are possible, using unit vectors of $\mathbb{R}^2$ is the simplest choice, so why make life more complicated than it already is? The assignment could also be exchanged, but it is convention to assign the vector with a non-zero first component to the "lowest state" $|0\rangle$ and the vector which has a non-zero last component to the "highest state" $|1\rangle$.

The dot product in the bra-ket notation involves the bra, meaning the complex conjugated and transposed (daggered) of the ket, e.g.,
$$
    \langle 1 | = | 1 \rangle^{\dagger} = (| 1 \rangle^\*)^T = \left[ \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} ^\* \right]^T = \begin{pmatrix} 0 & 1 \end{pmatrix}
$$
where the $\cdot^\*$ means that we [complex conjugate](https://en.wikipedia.org/wiki/Complex_conjugate) the ket and $\cdot^T$ means that we [transpose](https://en.wikipedia.org/wiki/Transpose) the vector. Note that in this case we only have real numbers such that the complex conjugation has no effect, but later, when going from classical bits to qubits, this is important.

Using this definition we can write the dot product of two basis vectors as
$$
    \langle 0 | 1 \rangle = \begin{pmatrix} 1 & 0 \end{pmatrix} \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = 1 * 0 + 0 * 1 = 0
$$
and as it is zero (vanishing), we say the two vectors are orthogonal to each other. As furthermore the dot product of the same basis vectors
$$
    \langle 0 | 0 \rangle = \begin{pmatrix} 1 & 0 \end{pmatrix} \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = 1 * 1 + 0 * 0 = 1
$$
is $1$, the basis vectors are said to be normalized, in general we can write the dot product for two arbitrary states
$$
    \langle a | b \rangle = \delta_{ab}
$$
where $\delta_{ab}$ is the Kroneker delta, which is $1$ if $a$ equals $b$ and zero otherwise.

Of course we know that with a single bit, we can only represent the state "on" or "off", but the power of computing comes because we combine multiple bits into larger data units. For example, the simplest extension is to use two bits, where we can have the following four combinations $|00 \rangle$, $|01 \rangle$, $|10 \rangle$, and $|11 \rangle$. To represent these states as vectors, we would need for example the four unit-vectors of $\mathbb{R}^4$, i.e.
$$
    |00 \rangle = \begin{pmatrix} 1 \\\\ 0 \\\\ 0 \\\\ 0 \end{pmatrix} \quad |01 \rangle = \begin{pmatrix} 0 \\\\ 1 \\\\ 0 \\\\ 0 \end{pmatrix} \quad
    |10 \rangle = \begin{pmatrix} 0 \\\\ 0 \\\\ 1 \\\\ 0 \end{pmatrix} \quad |11 \rangle = \begin{pmatrix} 0 \\\\ 0 \\\\ 0 \\\\ 1 \end{pmatrix}
$$
where the order of which unit vector is assigned to which combination is arbitrary, but it is advantageous to use this representation, as we will later see. This concept can of course be generalized to $n$ bits, representing the $\mathbb{R}^{2^n}$ vector space. Therefore for more than two bits it is already very tedious to write out the representation as the unit vectors, for example a standard byte has eight bits, meaning each state can be expressed as a unit vector of the $\mathbb{R}^{256}$, i.e. a vector with $256$ rows. This is one of the reasons why we prefer labels, like $|10010010\rangle$.

Using these definitions we can define operations, like the NOT operation as a matrix. For example the NOT operation $N$ is for the two-dimensional case given by the matrix
$$
    N = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix}
$$
since using [matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication) (row times column) yields
$$
    N |0 \rangle = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix} \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = \begin{pmatrix} 0\*1 + 1\*0 \\\\ 1\*1 + 0\*0 \end{pmatrix} = \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = |1 \rangle
$$
$$
    N |1 \rangle = \begin{pmatrix} 0 & 1 \\\\ 1 & 0 \end{pmatrix} \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = \begin{pmatrix} 0\*0 + 1\*1 \\\\ 1\*0 + 0\*1 \end{pmatrix} = \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} = |0 \rangle
$$
the inversion of the bit. This is, of course, a well-known operation in programming, i.e. a standard not operation on a boolean value, `!true = false` and `!false = true`.

## From classical bits to qubits

Why the mathematical formulation of the last section is needed might be unclear at first, but it will serve as a basis for the extension of classical bits to qubits, which will be explained in this section.

First note that we can also write for example the $|0 \rangle$ and $|1 \rangle$ state as
$$
    |0 \rangle = 1 |0 \rangle + 0 |1 \rangle = \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} \quad |1 \rangle = 0 |0 \rangle + 1 |1 \rangle = \begin{pmatrix} 0 \\\\ 1 \end{pmatrix}
$$
or in general we can write an arbitrary state $|\Phi \rangle$, again which is just a label, as
$$
    |\Phi \rangle = \alpha |0 \rangle + \beta |1 \rangle = \alpha \begin{pmatrix} 1 \\\\ 0 \end{pmatrix} + \beta \begin{pmatrix} 0 \\\\ 1 \end{pmatrix} = \begin{pmatrix} \alpha \\\\ \beta \end{pmatrix}
$$
where for a classical bit either $\alpha = 1$ and $\beta = 0$ or vice versa $\alpha = 0$ and $\beta = 1$. Other values are not allowed for classical bits.

Qubits generalize this concept, by allowing the coefficients $\alpha$ and $\beta$ to be arbitrary complex numbers. Typically, one imposes the condition $| \alpha |^2 + | \beta |^2 = 1$, which allows us to interpret the coefficients $\alpha$ and $\beta$ as [probability amplitudes](https://en.wikipedia.org/wiki/Probability_amplitude) of the states $|0 \rangle$ and $|1 \rangle$. The probability amplitudes are not real probabilities, but their square value, i.e. $| \alpha |^2$ or $| \beta |^2$, yield the probabilities with which we encounter the corresponding state. When measuring the result of this qubit, it will be the value with the largest coefficient. TODO: Move explanation of wave function here

This concept can be of course extended to an arbitrary amount of qubits, for example for the case of two qubits we have
$$
    |\Phi_1 \Phi_2 \rangle = \alpha |00 \rangle + \beta |01 \rangle + \gamma |10 \rangle + \delta |11 \rangle
$$
where the sum of the absolute values of all complex coefficients is normalized to $1$, i.e. $| \alpha |^2 + | \beta |^2 + | \gamma |^2 + | \delta |^2 = 1$. Again when measuring such a qubit, the result will be the state with the largest absolute value of the coefficient.

As the coefficients can take infinitely many different values, already two qubits can have infinitely many possible combinations, this is also usually referred to as superposition. But in the end, as the final answer of the measurement will be the state with the largest probability amplitude. Let us assume that a measurement of a two-qubit system yields the result $|01 \rangle$, then we would write
$$
    |\Phi_1 \Phi_2 \rangle = |01 \rangle = 0 |00 \rangle + 1 |01 \rangle + 0 |10 \rangle + 0 |11 \rangle
$$
meaning $\beta = 1$, being a non-complex real value, and all other coefficients are $0$. There exists no other possible solution, as all basis states are orthogonal to each other. This is commonly referred to as the collapse of the wave function, as the superposition of the states, from before collapses to a single state. After the measurement, there exists no possibility to obtain the values of the coefficients before the measurement, therefore this is also referred to as a destructive operation.

Let us briefly look, how the NOT operator, as defined for the classical bits, will change the probability amplitudes. To check this we will apply the NOT operation $N$ on the single qubit state $|\Phi \rangle$ and find
$$
    N |\Phi \rangle = N \left( \alpha | 0 \rangle + \beta | 1 \rangle \right) = \alpha | 1 \rangle + \beta | 0 \rangle = \beta | 1 \rangle + \alpha | 0 \rangle = \alpha' | 1 \rangle + \beta' | 0 \rangle = |\Phi' \rangle
$$
meaning that the NOT operation exchanges the probability amplitudes of the two states, or mathematically the new probability amplitudes are given by $\alpha' = \beta$ and $\beta' = \alpha$. For these new coefficients, the condition that the sum of squares of all absolute values is $1$ is still true, such that these can be interpreted as probability amplitudes. In general, when an operation maps to the same space, i.e. we do not need a new state to represent the result, and the sum of the squares of absolute values of the coefficients is preserved, it is called a unitary operation. Meaning when we perform unitary operations on a valid qubit state, we can be sure that the result will be a valid qubit state as well.

A general, valid operation in the context of quantum computing always involves the manipulations of the coefficients in such a way that the result is still a valid qubit state, i.e. a unitary operation.

## Qubits in more detail - entanglement

The probability properties of qubits alone, as described in the previous section, would not enable us to do advanced calculations. But there exists another very important concept in quantum computing, the entanglement of qubits. Using entanglement one has an advanced way of communicating information, with the important feature, that the information is only determined at the point of measurement.

If not familiar with quantum entanglement this probably sounds very abstract and confusing, but we will go through the details of the statement using an example, the Bell state $| \Phi^+ \rangle$. To put a two-qubit system into this state, the probability amplitudes of the two-qubit system have to be "initialized" (we will speak later about how to do that), such that $\alpha = \delta = 1/\sqrt(2)$ and $\beta = \gamma = 0$:
$$
    | \Phi^+ \rangle = \frac{1}{\sqrt{2}} \left(|00 \rangle + |11 \rangle\right)
$$
When measuring such a state we obtain either of the two states $|00 \rangle$ or $|11 \rangle$, with equal probability of $| \alpha |^2 = | \delta |^2 = 1/2$. Remember that this state has been built from two qubits, which can be separated and treated independently. When we now measure the value of one qubit independently of the other, we know that the other qubit has to have the same value, as the measured one, since the mixed states $|01 \rangle$ and $|10 \rangle$ are not present in the state and the wave function has collapsed to a single state, i.e. either $|00 \rangle$ or $|11 \rangle$. If two qubits are entangled, their probability amplitudes are correlated and therefore they are also referred to as "correlated qubits".

Note that sometimes this is interpreted, that a communication faster than light is possible. This is not true, since in order to know which value the other bit has, the measurement needs to be communicated to the other measurement.

## Operations in Quantum computing (gates)

As briefly shown in the section which introduced the vector notation of qubits, operations performed on qubits, will modify the probability amplitudes. We already have seen the NOT gate, exchanging the two probability amplitudes. This gate could be represented as a matrix only using real numbers, but in general, the entries of the matrix could be complex values. Explaining [complex numbers](https://en.wikipedia.org/wiki/Complex_number) in more detail would go out of scope of this blog post. But let us recap the most important feature in this context, that the square of a complex number $z = x + iy$ is given by $| z | = \sqrt{x^2 + y^2}$. A consequence of this is that for example the two distinct probability amplitudes $z\_1 = 0.3 + i 0.4$ and $z\_2 = 0.3 - i 0.4$ yield the same absolute value and therefore in our case the same probabilities, i.e. $| z\_1 | = | z\_2 | = 0.3^2 + 0.4^2 = 0.25$.

TODO: Explain complex conjugation

### Instructions

There of course exists a variety of logical quantum gates. The NOT gate which we have seen is the first [Pauli matrix $\sigma_1$](https://en.wikipedia.org/wiki/Pauli_group). The other two Pauli matrices are also valid quantum computing operations. For an overview refer [to Wikipedia](https://en.wikipedia.org/wiki/Quantum_logic_gate), the discussion of each individual gate would be a blog post on its own. In this blog post we will only discuss the very important [Hadamard gate](https://en.wikipedia.org/wiki/Quantum_logic_gate#Hadamard_gate) and [controlled NOT (CNOT) gate](https://en.wikipedia.org/wiki/Controlled_NOT_gate).

Similarly when we compile a high level language like [C](https://en.wikipedia.org/wiki/C_(programming_language)) to assembler, i.e. the [instruction set](https://en.wikipedia.org/wiki/Instruction_set_architecture) of the CPU, we can also compile complex quantum operations, to a basic instruction set of quantum operations using a "quantum compiler". One possible quantum instruction set, from which we can create all other quantum operations consist of the following five operations:
- **Measurement** When we apply this irreversible operation we measure the qubit, where after the measurement the superposition state has collapsed.
- **Hadamard gate** This gate creates an equal superposition state and will be discussed in the next subsection.
- **Phase gate** and **T-gate** These gates modify the probability amplitude of the $|1 \rangle$ state, by a multiplication of $i$ for the Phase gate and by $e^{i\pi/4}$, which both do not change the probability of the state, but rotate the probability amplitude in the complex plane by $\pi/2 = 90°$ and $\pi/4 = 45°$. These play an important role in many algorithms, but for now we will not discuss them further.
- **CNOT gate** While all other instructions only involved one qubit, this operation involves two qubits, i.e. this is in particular important for the interaction between the qubits. The controlled not gate, flips the second bit, if and only if the first bit is in the $|1\rangle$ state and we will discuss this gate in more detail in the next section.

### Hadamard and CNOT gate

Let us start with the Hadamard gate, which is defined by the following matrix:
$$
    H = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 & 1 \\\\ 1 & -1 \end{pmatrix}
$$
In order to find out how it modifies the amplitude, we apply it to a general qubit state $|\Phi \rangle$ and obtain:
$$
    H | \Phi \rangle = H \left( \alpha | 0 \rangle + \beta | 1 \rangle \right) = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 & 1 \\\\ 1 & -1 \end{pmatrix} \begin{pmatrix} \alpha \\\\ \beta \end{pmatrix} = \frac{1}{\sqrt{2}} \begin{pmatrix} \alpha + \beta \\\\ \alpha - \beta \end{pmatrix} = \frac{1}{\sqrt{2}} (\alpha + \beta) | 0 \rangle + \frac{1}{\sqrt{2}} (\alpha - \beta) | 1 \rangle
$$
This choice might seem random at first, but consider what happens when you apply it to a qubit which is only in the $|0 \rangle$ state, i.e. $\alpha = 1$ and $\beta = 0$, we will obtain a state which looks quite similar to the Bell state $| \Phi^+ \rangle$, i.e. $|+ \rangle := \frac{1}{\sqrt{2}} (|0 \rangle + |1 \rangle)$. This state is commonly named $|+ \rangle$ and referred to as equal superposition state, since when we measure it, we will get one of the two states with equal probability of $1/2 = 50%$.

When we combine this equal superposition state with another qubit in the $|0 \rangle$-state, we obtain
$$
    \frac{1}{\sqrt{2}} (|00 \rangle + |10 \rangle)
$$
where when we compare it to the Bell state $|\Phi^+\rangle$, we have the correct state $|00 \rangle$ for the first combination of qubits, but the second combination is a mixed state $|10 \rangle$.

This is where the CNOT gate comes into play. The CNOT gate acts on a two qubit state, and is defined such that it inverts the second bit if, and only if, the first qubit is $1$. For completes the matrix definition is given by
$$
    C = \begin{pmatrix} 1 & 0 & 0 & 0 \\\\ 0 & 1 & 0 & 0 \\\\ 0 & 0 & 0 & 1  \\\\ 0 & 0 & 1 & 0 \end{pmatrix}
$$
where we will not go through the detailed proof, but it can be similarly shown as for the Hadamard gate.

## Creating Bell superposition states

Therefore we could create a Bell state $|\Phi^+ \rangle$ using a combination of the Hadamard and CNOT gates. A problem is, that we only defined the Hadamard gate, such that it can be applied to a single qubit, but luckily there exists a systematic way to extend each operation, such that we can apply it to a combined state. This is defined as the [tensor product](https://en.wikipedia.org/wiki/Tensor_product). Unfortunately a detailed explanation of the tensor product is out of scope for this blog post, but for now there are two important properties we need know for our context. The first one is, that the tensor product of two vectors is each possible combination, such that for two qubits we can have the states $|00 \rangle$, $|01 \rangle$, $|10 \rangle$, and $|11 \rangle$. For operations it is composed such that the operation acts on the specific qubit.

That means we can always first apply a quantum gate on a single qubit and later combine it with another qubit. Therefore, we can create the Bell $|\Phi^+\rangle$-state using first the Hadamard gate on a single qubit $|0 \rangle$ and combine it with another qubit $|0 \rangle$ and apply the CNOT gate to the combination. quantum gates subsequently on
$$
    |\Phi^+ \rangle = C (H |0 \rangle) |0 \rangle = \frac{1}{\sqrt{2}} \left(|00 \rangle + |11 \rangle\right)
$$
where the Hadamard gate only acts on the first qubit and the CNOT operation on both.

Similarly when we apply this combination of gates to the $| 1 \rangle$ state, we obtain another important entangled state the Bell $|\Psi^-\rangle$-state:
$$
    |\Phi^- \rangle = C (H |1 \rangle) |0 \rangle = \frac{1}{\sqrt{2}} \left(|00 \rangle - |11 \rangle\right)
$$

## The power of quantum computing

Now we have discussed most of the basic ingredients of quantum computing, but you might ask, where is the magic why quantum computers are so powerful. Before discussing this in more detail, we should briefly speak about how quantum algorithms work. As we previously have seen, we can manipulate the probability amplitudes for qubits in a controlled manner. The idea of quantum algorithms is to increase the probability of the correct answer, and reduce all others, using constructive and destructive interference.

The power of quantum computing lies in the fact that the quantum effects can be represented using matrix multiplications, but these multiplications are not performed explicitly, instead they are carried out implicitly and we only read off the result of our quantum computer, which is also the correct answer for the original specified problem, using matrices. When adding a qubit, the matrix size, and therefore the problem size, is doubled. For example, for three qubits, the matrix size would be $8 \times 8$, with $64$ entries, for four qubits already a $16 \times 16$ matrix, with $256$ entries and in general for $N$ qubits $2^N \times 2^N$ matrices, with $2^{2N}$ entries. Already for 64 qubits the matrix would have $2^{128} = 340282366920938463463374607431768211456$ entries, which still can be simulated on a classical computer, but becomes quite challenging.

In general the matrix vector multiplication grows in volume by a factor of two for each qubit, which would be the number of operations a normal computer would need to do, if these matrix operations are performed explicitly.

## Noise and error correction

Some algorithms are more robust to errors than others in particular physical simulations

TODO: Write about constructive and destructive interference
