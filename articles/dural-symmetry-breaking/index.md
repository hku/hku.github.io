---
title: From Many-body Physics to Neural Network
---

### Intro

 In some sense, machine learning works like a "magic" to everybody.

For a real-world AI problem, such as classifying images, translating articles or playing games like AlphaGo, a learning machine demands thousands or millions of parameters to describe the complexity of the problem. Such a large number of parameters is a serious obstacle for our intuition or reasoning. 

Take neural networks for example. A neural network consist of many neurons. It's not difficult to understand the behavior of a single neuron, because the parameters of a single neuron is quite limited. However, the connectivity (or interaction) between the neurons make the neural network an indivisible entity, while our brains lack capacity to digest such a large chunk of parameters as a whole. Thus, even the parameters are tuned by the optimization algorithm we choose, such as minimizing KL-divergence by SGD (stochastic gradient descent), the neural network still seems like a "magic".

<!-- indivisible entity -->

This situation is also present in many-body physics. It's usually not difficult to describe the dynamics of a single particle. However, tracking a large number of interactive particles is an impossible mission in physics. In fact, even a three-body system may exhibit a chaotic behavior, which makes tracking them hopeless. 

As [P. W. Anderson](https://en.wikipedia.org/wiki/Philip_Warren_Anderson) says ["*more is different*"](http://science.sciencemag.org/content/177/4047/393), though the interaction between a large number of elementary particles is a difficult intellectual challenge for the physicists, it is the fundamental reason for the amazing diversity of our universe. (If the particles has no mutual interaction, no highly organized system would exist, including our human body.) Similarly, the complicated connectivity between the neurons is responsible for the neural network to function properly, though it seems hopeless to get an intuitive feeling of so many "interactive" neurons. 

A good news from many-body physics is that, though it is impossible to track every particles of a system, we may understand the collective behaviors ( or macroscopic properties) of the system in a statistical way. Thus, if we could establish a clear connection between many-body physics and machine learning,  we may borrow weapons from physics to understand machine learning and vice versa. For this purpose, let's first look at the phase transition theory in physics. 

### Phase Transition and Ising Model

(In this section, the terminology in physics might be introduced quite casually. The readers may refer to the links for more information if they are not explained in depth here. For those who have a physics background are also encouraged to glance over this section to get familiar with some discussion context. )

Phase transition is perhaps the most important object of study in many-body physics, which is also closely related to other interdisciplinary research. In several recent papers, it is shown that machine learning methods can be used to discover phase transitions in physics. In contrast, this online paper works on a reverse scenario in some way. __To be specific, this article would like to visually give the evidence that "phase transition" is also present in the training of neural networks, and it is possible to identify the central concepts of phase transition theory, such as "symmetry breaking" or "critical point", in machine learning.__

Phase transition theory is based on [statistical physics](https://en.wikipedia.org/wiki/Statistical_physics), of which the basic principle states that:

In thermal equilibrium, a given system at temperature $T$, the probability $p$ of its microscopic configuration $x$ is given by,

$$p(x) = {e^{-\beta H(x)}\over Z},\qquad \beta\equiv {1\over k_BT},\; Z\equiv\sum_{x^\prime} e^{-\beta H(x^\prime)}\ \label{eq1}$$

where $k_B$ is the Boltzmann constant, $H(\cdot)$ is the energy function, or called "Hamiltonian". $Z$ is a normalization factor, referred as "partition function", where the summation runs over all possible microscopic configurations.

Let's first look at the two limits of Eq. (\ref{eq1}):

* In high temperature limit $T\rightarrow +\infty$ (or $\beta\rightarrow +0$), the probability $p$ tends to be uniform for every configuration. Consider the definition of entropy, $s\equiv\sum\limits_x p(x) \log p(x)$, which measures the complexity of the system. Thus, in this case, the entropy is quite high, and we say the system is in a "thermal disordered phase".

* In low temperature limit $T\rightarrow +0$ (or $\beta\rightarrow +\infty$),  the probability $p$ focuses on the few configurations with lower energy. Consequently, the entropy approaches to zero.

As an example, let's consider the famous [Ising model](https://en.wikipedia.org/wiki/Ising_model), which is an elementary model to describe ferromagnetic/anti-ferromagnetic phase transition ( and also related to ["Boltzmann machine"](https://en.wikipedia.org/wiki/Boltzmann_machine) ). Its Hamiltonian given by,

$$H=-J\sum\limits_{\langle ij\rangle}s_is_j$$

where $J=\pm1$ is the coupling strength, $s_i\in \\{+1=\uparrow,-1=\downarrow \\} $ denotes the spin on site $i$ of a 2D square lattice. $\langle ij\rangle$ indicates a summation over nearest neighbors.  

For ferromagnetic Ising model ($J=1$), the configurations in which adjacent spins tend to be of the same sign have lower energy, thus dominate the system near zero temperature. Similarly, for anti-ferromagnetic Ising model ($J=-1$), the configurations in which adjacent spins tend to have the opposite sign dominate the system near zero temperature.

Though the "ground state" (The state which has the lowest energy.) of ferromagnetic/anti-ferromagnetic Ising model differs widely, their entropy as a function of the temperature $T$, behaves quite the same.  Notably, Ising model is [exactly solvable](https://en.wikipedia.org/wiki/Ising_model#Onsager's_exact_solution), and if we consider a cooling process from above the "critical temperature" $T_c=2/\log(1+\sqrt{2})\approx2.27$ to below $T_c$, there exist a "phase transition", across which the complexity of the system (measured by entropy) changes very fast.

In the following, we focus on the anti-ferromagnetic Ising model ($J=-1$), of which the ground state shows a checkerboard pattern which seems more interesting than the ferromagnetic case.