---
stand_alone: true
title: "Timing Regimes in Quantum Networks and their Physical Underpinnings"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-hajdusek-qirg-timing-physics
submissiontype: IRTF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: IRTF
workgroup: QIRG
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
  ins: M. Hajdušek
  fullname: Michal Hajdušek
  organization: Keio University
  email: michal@sfc.wide.ad.jp
 -
  ins: R. Van Meter
  fullname: Rodney Van Meter
  organization: Keio University
  email: rdv@sfc.wide.ad.jp

normative:

informative:
...
---

--- abstract

TODO Abstract

--- middle

# Prologue

In 1982, Digital Equipment Corporation, Intel, and Xerox published  **The Ethernet: A Local Area Network Data Link Layer and Physical Layer Specifications**. This 120-page document specifies pretty much everything: diameter of the coaxial cable, its impedance, dispersion, maximum cable length, voltages and currents, signal rise times, etc. The types of physical connectors allowed. How a bit is encoded in the signal. How a frame is demarcated. How collisions are detected. The format of messages. Addressing. Multicasting. Polynomials for error correction. It's ALL there.

Equally importantly, it specifies **timing requirements**.  For example, the rise time for a signal on the coaxial cable shall be 25 $$\pm$$ 5 nanoseconds. The total worst-case round-trip delay is calculated in a table to be 46.38 microseconds. How the entries in that table are combined to produce that number is fairly obvious; however, the numerical entries themselves are mostly unjustified in the specification itself, only stated. One exception is the statement, "Rise and fall times meet 10,000 series ECL requirements," referring to a specific series of well-known digital emitter-coupled logic parts, and hence incorporating a great deal of prior knowledge and work by reference.

In the quantum world, we are starting from first principles. Hence, we must begin at the beginning. We want to have specifications like Ethernet's, but first we must describe how the entries in e.g. the physical propagation delay budget are determined. The role of this document is to provide the underpinnings that give a shared understanding of how the basic numbers are determined and how they can be combined in a particular system design.

Thanks, DIX Ethernet creators, for showing the way!

# Introduction

Quantum networks that distribute end-to-end entanglement involve a number of tasks with varying demands on timing precision and jitter. The design of a quantum network will involve a layered protocol architecture where different layers take responsibility for meeting these differing constraints. This document describes the various timing regimes, from most to least stringent, in order to assist the process of making key design decisions.

The range of time scales of interest extends from ensuring the sub-wavelength stability of optical paths up to batch monitoring of the operation of the network itself. Light with a wavelength of 1.5 micrometers (common in communications, including quantum communications) has a frequency of approximately 200 THz (2E+14 Hz), for a cycle time of 5E-15 seconds.  Ranging from sub-wavelength stabilization through background operations such as routing, therefore, covers some 16 or more decimal orders of magnitude.  Add in a 24-hour thermal drift that must be compensated for in many cases, and we reach twenty decimal orders of magnitude from the bottom to the top. Naturally, meeting this range of demands requires the use of a variety of mechanisms.  This document avoids specifying solutions to the problems, and instead presents the functions and how their requirements are calculated (or measured).  Thus, each individual network design should apply the methods introduced here and present a numerical summary of the resulting values, after which corresponding solutions can be proposed and implemented.

Summary of timing regimes:

* **Interferometric stabilization:** photon wavepacket overlap, technology dependent, roughly nanoseconds.
* **Detector timing windoes:** opening and closing of detector timing windows, detector recovery time: nanoseconds to microseconds.
* **Measurement basis selection (if required in BSA):** performance will constrain entanglement attempt rate.
* **Optical switch control:** switching of trains of wave packets.
* **Pre-configured event-driven tasks:** timing-triggered or measurement-triggered execution of quantum circuits, microseconds
* **Urgent but not synchronization-critical tasks:** execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution, milliseconds
* **Host-side application-level tasks:** post-measurement operations, milliseconds
* **Background tasks:** link tomography calculations, routing table updates, seconds to minutes

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.

## Goals

* Identify and provide introduction to the physical principles related to timing regimes in quantum networks.
* Provide justification behind specific design choices discussed in our other documents.
* Serve as a reference for other quantum network specifications.

## Non-Goals

* Detailed physical derivations.
* Exhaustive coverage of all existing quantum platforms and technologies.

# Interferometric Stabilization

Entanglement distribution in quantum networks is performed by entanglement swapping (ES) on photonic qubits.
Central to photonic ES is the [Hong-Ou-Mandel (HOM) interference](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.59.2044), regardless of the photonic qubit encoding or of the particular protocol implementing photonic ES.
We begin by introducing the notation used, giving a brief overview of the effect, as well as discussing how to quantify the effect.
We then continue with a discussion of the requirements that must be satisfied in order to observe the effect.
This section follows quite closely this excellent [tutorial](https://arxiv.org/abs/1711.00080).

## Hong-Ou-Mandel interference

Consider two photons incident on a beamsplitter (BS) with reflectivity $$r$$.
We label the input modes by $$a$$ and $$b$$, and the output modes by $$c$$ and $$d$$.
We are interested in the observed behavior at the output modes of the BS.
There are four possible cases that may occur:

* Case A: photon in mode $$a$$ is reflected, while photon in mode $$b$$ is transmitted.
* Case B: both photons are transmitted.
* Case C: both photons are reflected.
* Case D: photon in mode $$a$$ is transmitted, while photon in mode $$b$$ is reflected.

This figure shows these cases:

<artwork type="svg" src="Figures-BW/HOM-bw.svg"></artwork>

The input state can be expressed as

~~~math
|\psi \rangle _{ab} = a ^{\dagger} _j b ^{\dagger} _k | 0 \rangle _{ab}
~~~

where the daggered operators represent bosonic creation operators, which create a single photon in the corresponding input port of the BS.
The indices $$j$$ and $$k$$ represent other properties of the photons that determine how distinguishable the photons are.
For example, $$j$$ and $$k$$ could represent

* polarizations (for polarization-encoded qubits),
* spectral modes,
* temporal modes (for time-bin encoded qubits),
* arrival time,
* transverse spatial mode.

Action of the BS on the input modes is the following:

~~~math
a ^{\dagger} -> \sqrt{1-r} c ^{\dagger} + \sqrt{r} d ^{\dagger}, \qquad b ^{\dagger} -> \sqrt{r} c ^{\dagger} - \sqrt{1-r} d ^{\dagger}
~~~

The output state of the two photons is

~~~math
|\psi\rangle _{cd} = ( \sqrt{r(1-r)} c ^{\dagger} _{j} c ^{\dagger} _{k} + r c ^{\dagger} _{k} d ^{\dagger} _{j} - (1-r) c ^{\dagger} _{j} d ^{\dagger} _{k} - \sqrt{r(1-r)} d ^{\dagger} _{j} d ^{\dagger} _{k} ) |0\rangle _{cd}
~~~

For a 50:50 BS when $$r=1/2$$:

~~~math
|\psi\rangle _{cd} = \frac{1}{2} \left( c ^{\dagger} _{j} c ^{\dagger} _{k} + c ^{\dagger} _{k} d ^{\dagger} _{j} - c ^{\dagger} _{j} d ^{\dagger} _{k} - d ^{\dagger} _{j} d ^{\dagger} _{k} \right) |0\rangle _{cd}
~~~

From this expression, we can see that when $$j=k$$, in other words when the input photons are indistinguishable, the output state has the following form,

~~~math
|\psi\rangle _{cd} = \frac{1}{\sqrt{2}} ( |2\rangle_c - |2\rangle_d )
~~~

The probability amplitudes for the cases where both input photons are transmitted or both reflected (Cases B and C in the figure above) interfere destructively.
Perfectly indistinguishable input photons always exit the BS in the same ouput mode.
It is this interference effect that is at the heart of quantum networking.

In order to quantify the effect that distinguishability has on HOM interference, we consider the **probability of a coincidence detection**, $$p_c$$, where one photon is detected in the BS output mode $$c$$, and the other photon in output mode $$d$$.
This probability is defined as

~~~math
p _c = \langle \psi | _{cd} P_c \otimes P_d | \psi \rangle _{cd}
~~~

where $$P_i$$, for $$i=c,d$$, are the projection operators representing a detection of a single photon in output mode $$i$$ of the BS.
For completely indistinguishable input photons that undergo the full HOM interference, we have $$p_c=0$$.
On the other hand, for fully distinguishable photons, the probability of a coincidence detection attains its maximum value $$p_c=1/2$$.

An often-used measure that quantifies the degree of HOM interference is the **visibility** $$V$$, defined via the probability of a coincidence detection,

~~~math
V = \frac{p_c ^{\text{max}} - p _c ^{\text{min}}}{p _c^{\text{max}}} = 1 - 2 p _c ^{\text{min}}
~~~

where we used the fact that the maximum probability of a coincidence detection is $$1/2$$.
We observe that the visibility varies from $$V=0$$ for fully distinguishable input photons to $$V=1$$ for perfectly indistinguishable ones.

Visibility $$V$$ plays a useful role when modelling the effects of imperfect HOM interference in the context of entanglement swapping.
Consider the case when the input photons $$a$$, $$b$$ are entangled with auxiliary systems $$s_1$$ and $$s_2$$, respectively.
The BSA performs ES by measuring the input photons, entangling systems $$s_1$$ and $$s_2$$ in the process.
Fidelity of the new entangled pair is directly proportional to the visibility $$V$$ of the HOM interference.
Non-ideal HOM interference can be modelled as a [two-qubit dephasing](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803),

~~~math
\rho _{s _1 s _2} = V \times \rho _{s _1 s _2} ^{\text{no-deph}} + (1 - V) \times \rho _{s _1 s _2} ^{\text{deph}}
~~~

where superscript no-deph denotes a density matrix resulting from an ideal ES at the BSA with unit visibility of the HOM interference, and superscript deph denotes a fully dephased state obtained by setting all off-diagonal elements of the density matrix to zero.

In the following subsections, we address and quantify how distinguishable photons affect the visibility of the HOM interference.

## Polarization

We now consider the case when the input photons differ in their polarization degree of photons.
The maximum probability of a coincidence detection is obtained for orthogonally polarized photons, for example when $$j=H$$ and $$k=V$$.
Here, $$H$$ denotes horizontal polarization and $$V$$ denotes vertical polarization.
The output state of the two photons is

~~~math
| \psi \rangle _{cd} = \frac{1}{2} \left( |1;H\rangle _c |1;V\rangle _c + |1;V\rangle _c |1;H\rangle _d - |1;H\rangle _c |1;V\rangle _d - |1;H\rangle _d |1;V\rangle _d \right)
~~~

We can observe that maximum probbility of coincidence is $$1/2$$.

In general, the two input photons will have polarizations given by two unit vectors, $$j=\epsilon$$ and $$k=\epsilon'$$.
The output state can be written as

~~~math
|\psi\rangle _{cd} = \frac{1}{2} \left( |1;\epsilon\rangle_c |1;\epsilon'\rangle_c + |1;\epsilon'\rangle_c |1;\epsilon\rangle_d - |1;\epsilon\rangle_c |1;\epsilon'\rangle_d - |1;\epsilon\rangle_d |1;\epsilon'\rangle_d \right)
~~~

The projection operators corresponding to a detection even at detector $$i$$ ($i=a,b$) are given by

~~~math
P _i = |1;\epsilon\rangle _i \langle 1;\epsilon| _i + |1;\epsilon'\rangle _i \langle 1;\epsilon'| _i
~~~

Either an $$\epsilon$$-polarized or an $$\epsilon'$$-polarized photon is detected in the output mode $$i$$.
The probability of coincidence is then

~~~math
p _{\text{co}} = \langle\psi| _{cd} P _c \otimes P _d |\psi\rangle _{cd} = \frac{1}{2} \left( 1 - \left| \langle\epsilon'|\epsilon\rangle \right| ^2 \right) = \frac{1}{2} \sin^2\theta
~~~

where the overlap between the polarization unit vectors is parametrized by $$\theta$$, and can be written as

~~~math
\langle\epsilon'|\epsilon\rangle = \cos\theta
~~~

We can define the corresponding visibility as a function of the angle between the two polarization vectors,

~~~math
V(\theta) = 1 - 2 p_{\text{c}} = \cos^2\theta
~~~

When the photons have identical polarization, the visibility reaches its maximum of 1.
On the other hand, when the photons are fully distinguishable and their polarization vectors are orthogonal, visibility vanishes.
The visibility and the probability of a coincidence detection are both displayed in the figure below.

<artwork type="svg" src="Figures-matplotlib/visibility-polarization.svg"></artwork>

Ensuring that the two input photons are indistinguishable in their polarization degree of freedom is critical for proper operation of the BSA.
Care must be therefore taken to characterize the photons just before they are incident onto the BS, as it is possible for the polarization of a photon to **drift** during its transmission and change its state from the one that the photon possessed immediately after emission.
This is often the case in fiber-based quantum networks, where polarization of photons is particularly sensitive to mechanical stresses and temperature gradients affecting the fiber.
This issue may be sidestepped by using polarization-maintaining fibers that are designed to suppress coupling between linearly-polarized orthogonal states of light.
However, these incur prohibitive costs for long-distance quantum communication, and may actually introduce unwanted coupling between linearly and circularly polarized light.

### Interference of photons from two independent EPPS

The preceding discussion was concerned with two independent pure photons of different polarization.
In the context of quantum networking, a much more common scenario is that of two entangled pairs of photons originating from two independent EPPS nodes, where two qubits, one from each pair, are incident onto a BS and undergo HOM interference.
The two pairs are in the following initial state,

~~~math
|\psi\rangle_{a_1b_2} = \frac{1}{\sqrt{2}}\left( |HV\rangle_{a_1a_2} + e^{i\theta_1} |VH\rangle_{a_1a_2} \right), \qquad |\psi\rangle_{b_1b_2} = \frac{1}{\sqrt{2}}\left( |HV\rangle_{b_1b_2} + e^{i\theta_2} |VH\rangle_{b_1b_2} \right)
~~~

where $$\theta_1$$ and $$\theta_2$$ represent the polarization drift induced in the single-mode fiber.
Photons $$a_2$$ and $$b_1$$ are incident onto a BS, where they undergo HOM interference.
Following the same calculation as above, it can be shown that the probability of a coincidence event is $$p_{c} = 1/4$$, regardless of the polarization drift.
This suggests that the visibility is insensitive to the polarization drift.
However, the polarization drift must be tracked regardless because it affects the fidelity of the post-ES state of photons $$a_1$$ and $$b_2$$.
It is therefore important to characterize the polarization drift at the BSA at regular intervals and compensate for it.
This can be done at the nodes generating the photon pairs at the cost of the BSA having to communicate polarization drift results to the ends nodes.
Or it can be compensated for directly at the BSA using waveplates at the cost of increased complexity of the BSA.
In [Krutyanskiy et.al.](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803), polarization drift characterization and compensation at the BSA takes a few minutes and is performed every 20 minutes.

## Spectral distinguishability

Another important source of distinguishability in HOM interference is the spectral property of the input photons.
The photon wave packet of a photon can be represented by its **spectral amplitude function** $$\phi(\omega)$$ that satifies the normalization condition:

~~~math
\int d\omega |\phi(\omega)|^2=1
~~~

Two input photons become distinguishable if their respective spectral amplitude functions are not equal.
We resctrict our discussion to Gaussian spectral amplitude functions but the same methods generalize to arbitrary photons.
The two photons may have different central frequencies or different standard deviations, as shown in the Figure below.

<artwork type="svg" src="Figures-BW/distinguishability-spectral-bw.svg"></artwork>

In this subsection, we analyze the requirements in terms of the photonic spectral amplitude function that lead to high visibility of the HOM interference.

### Pure states

We begin the discussion by focusing on pure states of the input photons first.
Single-photon state with a spectral amplitude function $$\phi(\omega)$$ is a superposition written as

~~~math
|1;\phi\rangle_a = \int d\omega \phi(\omega) a ^{\dagger}(\omega) |0\rangle_a
~~~

where creation operator creates a photon in the BS input mode $$a$$ with frequency $$\omega$$.
Two input photons with arbitrary spectral functions $$\phi$$ and $$\varphi$$ are described by

~~~math
|\psi \rangle _{ab} = |1;\phi\rangle_a |1;\varphi\rangle_b = \int d\omega_1 \phi(\omega_1) a ^{\dagger}(\omega_1) \int d\omega_2 \varphi(\omega_2) b ^{\dagger}(\omega_2) |0 \rangle _{ab}
~~~

We assume that the BS acts on the different frequency modes independently, and that the reflectivity is frequency-independent.
Applying the same transformation rules for the creation operators, the output state of the two photons is

~~~math
|\psi \rangle _{cd} = \frac{1}{2} \int d\omega_1 \phi(\omega_1) \int d\omega_2 \varphi(\omega_2) \left[ c ^{\dagger}(\omega_1) c ^{\dagger}(\omega_2) + c ^{\dagger}(\omega_2) d ^{\dagger}(\omega_1) - c ^{\dagger}(\omega_1) d ^{\dagger}(\omega_2) - d ^{\dagger}(\omega_1) d ^{\dagger}(\omega_2) \right] |0\rangle _{cd}
~~~

The projection operators corresponding to a detection event in output mode $$c$$ and output mode $$d$$ are given by

~~~math
P _c = \int d\omega c ^{\dagger}(\omega) |0\rangle_c\langle0|_c c (\omega),\quad P _d = \int d\omega d ^{\dagger}(\omega) |0\rangle _d\langle0| _d d(\omega)
~~~

The probability of a coincidence detection is then

~~~math
p _{\text{c}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1) \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2)
~~~

The form of this expression is the same as the one in subsection on polarization above, where the probability of a coincidence detection depended on the overlap between the polarization vectors $$\epsilon$$ and $$\epsilon'$$.
Now, $$p_c$$ depends on the overlap between the spectral amplitude functions.
If the input photons are fully distinguishable, their respective spectral amplitude functions $$\phi(\omega)$$ and $$\varphi(\omega)$$ are orthogonal and the integrals vanish, meaning $$p_c=1/2$$.
On the other hand, for completely indistinguishable input photons we have $$\phi(\omega)=\varphi(\omega)$$, and due to the normalization condition we obtain $$p_c=0$$.

### Mixed states

Previous discussion of pure states can be extended to include mixed states of the input photons.
Such states will inevitably arise due to imperfections in the preparation procedure and due to the input photons being entangled with other degrees of freedom.
These can include other photons or quantum memories.

The mixed state of an input photon is described by the following density matrix:

~~~math
\rho_a = \sum_k u_k |1;\phi_k\rangle_a\langle1;\phi_k|_a, \quad \sum_k u_k=1
~~~

where the state of the photon is a mixture of pure single-photon states with spectral amplitude function $$\phi_k(\omega)$$, weighted by probability $$u_k$$.
The two-photon input state can be written as

~~~math
\rho ^{\text{in}} _{ab} = \sum _{kk'} u_k v _{k'} |1;\phi_k\rangle_a |1;\varphi _{k'}\rangle_b \langle 1;\phi_k|_a \langle 1;\varphi _{k'}|_b
~~~

It is not necessary to repeat the entire calculation we did for pure states.
Due to linearity of quantum mechanics, we can immediately write the expression for the probability of coincidence as a sum of pure-state coincidence probabilities weighted by $$u_k$$ and $$v'_k$$:

~~~math
p _{\text{c}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} u _k v _{k'} \int d\omega_1\phi^\ast_k(\omega_1)\varphi _{k'}(\omega_1) \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2)
~~~

### Example 1: Gaussian wave packets

In this example, we consider input photons with Gaussian spectral amplitude functions.
The spectral amplitude functions are given by

~~~math
\phi_i(\omega) = \frac{1}{\pi^{1/4}\sqrt{\sigma_i}} e ^{-\frac{(\omega-\bar{\omega}_i)^2}{2\sigma^2_i}},\quad\text{for } i=a,b
~~~

The probability of a coincidence detection is then

~~~math
p _{\text{c}} = \frac{1}{2} -\frac{\sigma_a\sigma_b}{\sigma_a^2 + \sigma_b^2} e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{\sigma_a^2+\sigma_b^2}}
~~~

**Case A (different central frequencies)**

We assume that the two spectral amplitude functions have the same standard deviation, which simplifies the expression for the probability of a coincidence detection to

~~~math
p _{\text{c}} = \frac{1}{2} \left( 1 - e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{2\sigma^2}} \right)
~~~

We observe that for identical photons, the probability of a coincidence detection vanishes.
For fully distinguishable wave packets, when the difference between central frequencies diverges, the probability approaches 1/2.
The visibility as a function of the difference between the central frequencies is

~~~math
V(\bar{\omega}_a-\bar{\omega}_b) = e^{-\frac{(\bar{\omega}_a-\bar{\omega}_b)^2}{2\sigma^2}}
~~~

**Case B (different standard deviations)**

The spectral amplitude functions have the same central frequencies, which gives the following expression for the probability of coincidence and visibility,

~~~math
p_{\text{c}} = \frac{1}{2} - \frac{\sigma_b/\sigma_a}{1 + (\sigma_b/\sigma_a)^2}, \quad V(\sigma_b/\sigma_a) = \frac{2\sigma_b/\sigma_a}{1 + (\sigma_b/\sigma_a)^2}
~~~

The probability of coincidence and corresponding visibility for both Cases are shown in the Figure below.

<artwork type="svg" src="Figures-BW/visibility-spectral-bw.svg"></artwork>

## Wave Packet Overlap

So far we have assumed that the two input photons arrive at the BS at exactly the same time.
In this subsection, we address this unrealistic assumption and quantify how temporal distinguishability affects the visibility of HOM interference.
Even for photons with identical spectral amplitude functions, different arrival times result in decreased overlap between the photons' wave packets, diminishing the visibility of the HOM interference.
If the times of arrival are too different as shown in the Figure below, the probability of a coincidence detection will reach its maximum value of 1/2, and the visibility will vanish.

<artwork type="svg" src="Figures-BW/distinguishability-temporal-bw.svg"></artwork>

Without loss of generality we assume that photon $$b$$ is delayed by a time $$\tau$$, which transforms its creation operator,

~~~math
b ^{\dagger}(\omega) \rightarrow b ^{\dagger}(\omega) e^{-i\omega\tau}
~~~

Two input photons with arbitrary spectral functions $$\phi$$ and $$\varphi$$, with photon $$b$$ arriving late, are described by

~~~math
|\psi\rangle _{ab} = |1;\phi\rangle_a |1;\varphi\rangle_b = \int d\omega_1 \phi(\omega_1) a ^{\dagger}(\omega_1) \int d\omega_2 \varphi(\omega_2) b ^{\dagger}(\omega_2) e^{-i\omega_2\tau} |0\rangle _{ab}
~~~

We assume that the BS acts on the different frequency modes independently, and that the reflectivity is also frequency-independent.
Applying the same transformation rules for the input creation operators, the output state of the two photons is

~~~math
|\psi\rangle _{cd} = \frac{1}{2} \int d\omega_1 \phi(\omega_1) \int d\omega_2 \varphi(\omega_2) e^{-i\omega_2\tau} \left[ c ^{\dagger}(\omega_1) c ^{\dagger}(\omega_2) + c ^{\dagger}(\omega_2) d ^{\dagger}(\omega_1) - c ^{\dagger}(\omega_1) d ^{\dagger}(\omega_2) - d ^{\dagger}(\omega_1) d ^{\dagger}(\omega_2) \right] |0\rangle _{cd}
~~~

For pure input states, the probability of a coincidence detection is

~~~math
p _{\text{c}} = \frac{1}{2} - \frac{1}{2}\int d\omega_1\phi^\ast(\omega_1)\varphi(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi^\ast(\omega_2) \phi(\omega_2) e^{i\omega_2\tau}
~~~

while for mixed states is can be generalized to the following form,

~~~math
p _{\text{c}} = \frac{1}{2} - \frac{1}{2} \sum _{kk'} u_k v _{k'} \int d\omega_1\phi_k^\ast(\omega_1)\varphi _{k'}(\omega_1)e^{-i\omega_1\tau} \int d\omega_2 \varphi _{k'}^\ast(\omega_2) \phi_k(\omega_2) e^{i\omega_2\tau}
~~~

### Example: Gaussian wave packets

Consider two identical pure Gaussian wavepackets that arrive at the BS with a time difference given by $$\tau$$.
The probability of coincidence and the corresponding visibility are given by

~~~math
p_{\text{c}} = \frac{1}{2} \left( 1 - e^{-\frac{1}{2}\sigma^2\tau^2} \right), \quad V(\tau) = e^{-\frac{1}{2}\sigma ^{2} \tau^{2}}
~~~

The figure below displays the visibility and probability of coincidence for this case.

<artwork type="svg" src="Figures-BW/visibility-temporal-bw.svg"></artwork>

# Detector Timing Windows

In this section, we discuss how properties of single-photon detectors (SPDs) affect the timing regimes in quantum networks.
An ideal SPD generates an electrical signal after absorbing a photon, and generates no signal in the absence of a photon.
This is not always true for [real-world SPDs](https://www.nature.com/articles/nphoton.2009.230).

## Detector basics

Performance of SPDs can be quantified by the following characteristics,

* **Spectral range:** SPDs are sensitive over a limited range of wavelengths. This range depends on the materials used in the fabrication of the detector. Typical spectral ranges are in the near-infrared, around 1550nm, where commercial optical fibers perform best in terms of photon loss rates.
* **Detection efficiency:** The overall probability that an incoming photon registers a count, denoted by $$\eta$$.
This efficiency can be further broken down.
Probability of losing the photon before it reaches the detector is described by the _coupling efficiency_, $$\eta_coupling$$.
The type of material and geometry of the detector determine the photon _absorption efficiency_, $$\eta_absorption$$.
Finally, the probability that an electric signal is generated upon successful absorption of a photon is described by the _registering efficiency_, $$\eta_registering$$.
The overall _system detection efficiency_ is given by the product of these three,

~~~math
\eta_{\text{sde}} = \eta_{\text{coupling}} \times \eta_{\text{absorption}} \times \eta_{\text{registering}}
~~~

The _device detection efficiency_ is given by

~~~math
\eta_{\text{dde}} = \eta_{\text{absorption}} \times \eta_{\text{registering}}
~~~

Detection efficiency affects the rate at which entanglement can be distributed.

* **Recovery time:** Denoted by $$\tau_recovery$$ and also known as 'dead' time. It is the time duration following an absorption of a photon during which the detector is unable to reliably detect another photon. Recovery time affects the maximum detection rate. If the source of photons has low efficiency, the clock rate does not need to be limited by the recovery time, as majority of the trials will not produce a photon. This could also be the case if the probability of losing the photon is high (either due to loss in fiber or due to low system detection efficiency $$\eta_sde$$). On the other hand, if the photon source is highly efficient, it is important to ensure that the separation between the wavepackets is longer than $$\tau_recovery$$ to ensure effcient use of the generated photons.
* **Dark count rate:** SPDs have a finite chance to produce an output electric signal even in the absence of a photon. This may be caused by materials properties of the detector, biasing conditions, or external noise. It is usually given in Hz (counts per second). Dark counts decrease the fidelity of the distributed entangled states.
* **Timing jitter:** Denoted by $$J_timing$$.
Describes the variation in time between the photon being absorbed and the output electric signal being generated. A few example profiles are shown in the Figure below taken from [Hadfield's review](https://www.nature.com/articles/nphoton.2009.230).

<artwork type="svg" src="Figures/timing-jitter.svg"></artwork>

The table below shows the above characteristics for a [SNSPD](https://singlequantum.com/wp-content/uploads/2022/12/SQ-General-Brochure.pdf).


| Wavelength | 800 nm | 1550 nm |
| ---------- | ------ | ------- |
| System detection efficiency | > 90% | > 90% |
| Recovery time | 10 ns | 20 ns |
| Dark count rate | < 1 Hz | < 1 Hz |
| Timing jitter | < 15 ps | < 15 ps |

## Acceptance window

In the previous Section, we used $$\tau$$ to denote the difference between the arrival time of the photons at the BSA.
However, due to emission jitter it is impossible to know the precise time of arrival of a photon.
The only information that is available comes from the electric output signal of a detector.
Time of detection is in general different from the time of photon arrival due to finite time needed to generate the output signal described by the timing jitter J_timing.
Therefore, we will use $$\tau$$ to denote the difference in detection time of the two photons.

Measurement at the BSA is successful when the correct pattern of detector clicks is observed, and the difference in detection times $$\tau$$ is smaller than a given detection **acceptance window**, T_window.
The size of this window affects both the fidelity and the generation rate of the entangled pairs that the link produces.
Large acceptance windows produce high rates but low fidelity, while small acceptance windows result in low rates and high fidelity.
The appropriate size of the acceptance window must be chosen in order to satisfy the demands of the application requesting the entangled states.
Reaching the requested fidelity should take priority over high generation rate.

## Separation in a train of wavepackets

Current experiments on quantum repeaters use single quantum memory per QNIC.
As quantum technologies improve, it is likely that QNICs will be equipped with multiple quantum memories.
This will allow for generation of link-level entanglement in a multiplexed manner, where trains of photons, each originating from a different memory inside the same QNIC, are sent to the BSA.
The photons making up a train must be well separated such that upon a successful BSM, the BSA can uniquely identify which two photons were measured.
We refer to the minimum separation between the photons as the **separation time** T_separation.

The size of the separation time depends on the following:

* **Wave packet shape:** Individual photons cannot have overlapping spatial wavepackets, which may lead to incorrect assignement of entangled qubits following a successful meadurement at the BSA. We will use T_photon to denote the length of a wavepacket in seconds.
* **Detector recovery time:** Spacing the wavepackets too close to each other may result in some of the photons being lost due to the detector recovering following a detection event, leading to inefficient use of initially generated entangled pairs (either memory-photon or photon-photon).
* **Memory emission jitter:** The separation between the wavepackets must take into account the probabilistic nature of photon emission from a quantum memory in order to prevent wavepacket overlap.
* **Detector timing jitter:** Generation of the electric signal following absorption of a photon varies in duration, leading to a variance in timing of the detection event. This may lead to the BSA mislabelling which photons were part of a successful measurement if their wavepackets are spaced too closely.

General (conservative) separation time should therefore be set to

~~~math
T_{\text{separation}} \ge T_{\text{photon}} + J_{\text{emission}} + J_{\text{timing}}
~~~

The above discussion assumes that the photons can be generated nearly on-demand.
This is a fair assumption in the case of quantum memories based on [trapped ions](https://ora.ox.ac.uk/objects/uuid:604c53b9-8df8-4e45-8103-10fd81eb3366).
Here, the memory must be first initialized by cooling it to its ground state, a process which takes <1ms.
The memory is then excited by a laser pulse of approximately 50 microseconds that generates a photon.

In the case of memory-less link architectures, the picture is slightly different.
Here, EPPS nodes utilizing the principle of spontaneous parametric down-conversion (SPDC) generate entangled photon pairs.
Each photon is sent to a different BSA, where they are measured with a photon originating from a different EPPS node.
SPDC is an inefficient process with success probability of around $$10^{-6}$$ per pump photon. In system design, the intensity of the pump laser is adjusted so that the average number of photons is appropriate; generally this must be set below one photon per time window in order to avoid polluting the signal with two-photon states.
This means that most of the time windows given by the separation time will not contain a photon.
However, the separation time should be maintained in order to correcly identify the photons that were part of a successful measurement at the BSA.
The separation time governs the maximum rate at which EPPS attempts to generate the entangled photon pairs, which is given by 1/T_separation.

# Measurement basis selection

We have encountered Bell-state measurements performed by the BSA on photonic qubits that are needed for entanglement swapping.
These measurements were static in the sense that we did not need to change the measurement basis.
Observed detection pattern determined whether the post-measurement state of the remote quantum systems (either quantum memories or other photons) was $$|\Psi^+\rangle$$ or $$|\Psi^-\rangle$$.
Projection onto two of the four Bell-basis states was achieved probabilistically without actively applying any transformations on the photonic qubits.
We will see in this Section that the photonic BSA is a very special case in this regard, and that changing the basis of the measurement is an indispensable part of quantum networking.
Entanglement swapping on stationary qubits stored in quantum memories is not possible without applying appropriate unitaries first that change the basis of the measurement.
There are also cases, where change of basis is required even when dealing with only photonic qubits.
An example of this are the so-called all-photonic quantum repeaters, where measurement basis is conditioned on the outcomes of previous measurements, leading to the requirement of very fast basis switching.

## Measurement basics

We will first discuss quantum measurements in general before discussing concrete implementations and their timing requirements based on their physical implementations.

### Single-qubit measurements

For simplicity, we begin with measurements on a single qubit before generalizing to two qubit measurements.
Consider a general state of the qubit, $$|\psi\rangle = \alpha |0\rangle + \beta |1\rangle$$.
Measurement in an arbitrary basis $$M$$ projects $$|\psi\rangle$$ onto one of the eigenvectors of $$M$$.
Probabilities of the two possible measurement outcomes are given by the squared modula of the overlaps between the initial state $$|\psi\rangle$$ and the eigenvectors of the observable $$M$$.

~~~math
\text{Pr}(|\phi\rangle;|\psi\rangle)=|\langle\phi|\psi\rangle|^2, \quad \text{Pr}(|\phi^{\perp}\rangle;|\psi\rangle)=|\langle\phi^{\perp}|\psi\rangle|^2
~~~

It is often difficult to directly measure the qubit in an arbitrary basis when it comes to real-world implementation.
In such a case, the qubit needs to be pre-rotated by an appropriate unitary operation, and then measured in the $$Z$$ basis, which can usually be implemented in a straightforward way.
This approach greatly simplifies the implementation of arbitrary measurements.

Consider that the observable $$M$$ is related to the Pauli $$Z$$ by unitary $$U$$:

~~~math
M = U Z U ^{\dagger}
~~~

This means the unitary $$U$$ relates the eigenvectors of the two observables,

~~~math
|\phi\rangle = U |0\rangle, \quad\text{and}\quad |\phi^{\perp}\rangle = U |1\rangle
~~~

We can perform measurement in the $$M$$ basis by applying adjoint of $$U$$ to the initial state $$|\psi\rangle$$, and then measuring it in the Pauli $$Z$$ basis.
This can be easily verified by rewriting the above probabilities corresponding to the two measurement outcomes,

~~~math
\text{Pr}(|\phi\rangle;|\psi\rangle) = |\langle\phi|\psi\rangle|^2 = |\langle0| U ^{\dagger}|\psi\rangle|^2 = \text{Pr}(|0\rangle; U ^{\dagger}|\psi\rangle)
~~~

and

~~~math
\text{Pr}(|\phi^{\perp}\rangle;|\psi\rangle) = |\langle\phi^{\perp}|\psi\rangle|^2 = |\langle1| U ^{\dagger}|\psi\rangle|^2 = \text{Pr}(|1\rangle; U ^{\dagger}|\psi\rangle)
~~~

This is pictured in the Figure below.

<artwork type="svg" src="Figures/measurement-1qubit.svg"></artwork>

### Two-qubit measurements

The same principle of changing the measurement basis can be generalized to two qubits.
This time state $$|\psi\rangle$$ represents a general two-qubit state, unitary $$U ^{\dagger}$$ acts on both qubits, which are both finally measured in Pauli $$Z$$ basis.
In the majority of cases, we are interested in performing measurements in the Bell basis.
Required unitary is the Hermitian conjugate of the unitary that creates a Bell pair when the qubits are both initialized in $$|0\rangle$$, as shown in the Figure below, where we have dropped the $$Z$$ label indicating the measurement basis.

<artwork type="svg" src="Figures/measurement-2qubit.svg"></artwork>

## Measurements on quantum memories

In this Section, we discuss various methods of implementing measurements of quantum memories.
These methods vary based on the quantum technology used as the quantum memory, and even within the same technology there are usually variations.
We are mainly concerned with giving an overview of the different measurement methods, and their respective timing regimes.

### Trapped ions

Trapped ions possess [two degrees of freedom](https://journals.aps.org/rmp/abstract/10.1103/RevModPhys.75.281).
The first one is the motional degree of freedom, resulting from the ion oscillating around its equilibrium position in the trap.
The second one is the internal degree of freedom, represented by the ground state $$|g\rangle$$ and the excited state $$|e\rangle$$.
It is the latter degree of freedom which is used to encode a qubit and hence acts as a quantum memory.

Measurement in the **Pauli Z** basis is performed by **electron shelving** via the use of a third atomic level $$|r\rangle$$, with much shorter life time than the [excited state](https://www.amazon.co.jp/Quantum-World-Ultra-Cold-Atoms-Light/dp/1783266163) $$|e\rangle$$, $$\tau_e \gg \tau_r$$.
The figure below demonstrates how this method works.

<artwork type="svg" src="Figures/shelving.svg"></artwork>

The ion is illuminated by light tuned to resonate with the transition $$|g\rangle <->|r\rangle$$, represented by the red straight arrow in the Figure above.
If fluorescence is immediately observed, this corresponds to measuring the ion in the ground state $$|g\rangle$$.
If no fluorescence is observed, the ion is measured in the excited state $$|e\rangle$$.
Hypothetically a single fluorescent photon would be sufficient, however, the fluorescent photons are only rarely captured into the measurement apparatus (typically involving lenses and a camera) and observed, and stray photons are also often captured, so a relatively long **integration time** is used to confirm the fluorescence with high probability.  (Solid-state systems such as quantum dots and superconducting qubits also need relatively long integration times in their measurement processes.)
Combined with laser pulses that apply a single-qubit rotation, measurement of a **single ion in an arbitrary basis** can be performed in [1-2 ms](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803).

The **CNOT gate** can be applied in two different ways.
The original proposal is due to [Cirac and Zoller](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.74.4091), where the ions needed to be cooled to their collective motional ground state first.
This approach was demonstrated experimentally using [calcium ions](https://www.nature.com/articles/nature01494).
Execution of the gate took around 600 microseconds, with the achieved fidelity being less than 0.8.
The second approach is due to [Molmer and Sorensen](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.82.1835), and is more robust against motional excitation.
This led to high-fidelity demonstrations of $$>0.99$$, and gate times of around 50 microseconds.

## Measurements on photonic qubits

Measurement of polarization-encoded photonic qubits can be performed with the aid of a **polarizing beam splitter** (PBS), a **half waveplate** (HWP), a **quarter waveplate** (QWP), and [two detectors](https://link.springer.com/chapter/10.1007/978-3-540-44481-7_4) (one detector is enough in fact but less efficient).
The idea is the same as in the case of measurements performed on stationary qubits discussed above.
Setting the HWP and QWP at particular angles applies the unitary $$U ^{\dagger}$$ that picks the basis of the measurement, while the PBS filters out vertical and horizontal polarizations that then get detected by the detectors placed in the output paths of the PBS.
Horizontal polarization gets transmitted through the PBS, while vertical polarization gets reflected.
This setup is shown in the figure below.

<artwork type="svg" src="Figures/2detector-polarization.svg"></artwork>

General pure state of a polarization-encoded qubit can be written as

~~~math
|\psi\rangle = \cos\left(\frac{\theta}{2}\right)|H\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)|V\rangle
~~~

This is directly equivalent to expressing the qubit state in the computational basis, and can be visualized with the help of the **Poincaré sphere**.
The table below summarizes this equivalence.

<artwork type="svg" src="Figures/polarization-table.svg"></artwork>

The Figure below shows the Poincaré sphere along with the position of the polarization states.

<artwork type="svg" src="Figures/poincare-sphere.svg"></artwork>

Polarization of light is manipulated by waveplates.
Waveplate rotated by an angle $$\alpha$$ (zero is aligned with the horizontal axis) rotates the polarization state around an axis, located at an angle of $$2\alpha$$ with the horizontal state $$|H\rangle$$ in the horizontal plane, as shown in Figure above.
Half waveplate rotates the polarization state by an angle $$\pi$$, while a quarter waveplate rotates by an angle $$\pi/2$$ in the Poincaré sphere.
The action of the waveplates is captured by the corresponding unitary operations:

<artwork type="svg" src="Figures/waveplates-matrix.svg"></artwork>

The idea behind measurements in arbitrary basis

~~~math
\{|\psi\rangle, |\psi^{\perp}\rangle\}
~~~

is to choose the angles for the waveplates such that the following transformation is achieved,

~~~math
U_{HWP}(\alpha_{HWP})U_{QWP}(\alpha_{QWP}) |\psi\rangle \rightarrow |H\rangle, \quad U_{HWP}(\alpha_{HWP})U_{QWP}(\alpha_{QWP}) |\psi^{\perp}\rangle \rightarrow |V\rangle
~~~

Settings for the three Pauli bases are summarized in the table below.

<artwork type="svg" src="Figures/waveplate-angles.svg"></artwork>

Changing the basis of measurement requires mechanical rotation of the waveplates and coordination with the detectors.
The waveplates can be rotated by a motorized rotator device, which can be adjusted at a rate of around 1 degree per 100ms.
Therefore, for a rotation of 45 degrees, the motor requires areound 4.5s.
During the rotation interval, any results obtained from the detectors must be discarded as they correspond to measurements in an undesired basis.

# Optical Switch Control

Optical switches play an essential role in distributed computing and communication systems.
Their job is to guide light from a given input to the desired output.
Optical switches have a number of important characteristics such as _insertion loss_, _crosstalk_, and _size_.
In the context of timing regimes, we will focus on the following characteristics in this section,

* **Switching time:** time required to reconfigure the switch.
* **Propagation time delay:** time required for the photon to travel across the switch.

Two approaches to switching are of relevance to our discussion.
The first approach is the _crossbar switch_ with all-to-all connectivity. Such an $$N\times N$$ switch can be reconfigured to accomodate all possible $$N!$$ permutations of input-output pairs. One usual implementation of a crossbar switch is using **microelectromechanical systems (MEMS)** relying on small movable parts such as popup micromirrors, rotating prisms or [spinning holographic disks](https://onlinelibrary.wiley.com/doi/book/10.1002/0471213748). MEMS have usually low insertion loss and crosstalk, however due to their mechanical nature they suffer from slow switching times, which range from 10 microseconds to 10 miliseconds.

Crossbar switches are important in classical switching networks and are use in classical control systems in some quantum technologies. In the context of quantum networks, it is often not necessary for the switch to be able satisfy all possible $$N!$$ input-output permutations.  For example, the switch can be placed behind a pool of entangled photon pair sources (EPPS) in order to route entangled photons towards end nodes requesting a [connection](https://opg.optica.org/jocn/fulltext.cfm?uri=jocn-8-5-331&id=340335). Or the switch can be placed before a pool of Bell State Analyzers (BSA) and route input pairs of photons to the desired BSA, where they undergo measurement in the [Bell basis](https://ieeexplore.ieee.org/document/10821447). These approaches are pictured in the figure below.

<artwork type="svg" src="Figures/optical-switch.svg"></artwork>

Both of these designs consider a $$2\times 2$$ switch as the basic building block, which is implemented with **integrated photonics** and controlled electro-optically. Applied electric fields are used to alter the refractive index of the material (such as lithium niobate) to change the state of the switch from a BAR state to a CROSS state. Switching times for electro-optical switches are much faster, varying from 10 nanoseconds to 10 microseconds.

The optical switch introduces a **propagation time delay**. For some MEMS switches, this delay can be as low as [25 nanoseconds](https://www.viavisolutions.com/en-us/literature/polatis-series-6000-osm-network-switch-module-data-sheets-en.pdf). In general, this delay time varies with the choice of input-output ports. This variation is probably insignificant in most classical contexts, but as discussed in Section A, any delay between the arrival times of photon pairs at the same BSA may result in decreased visibility further lowering the fidelity of the post-measurement state. The issue of arrival time delay arises in the case of integrated switches used in paired-egress BSA pools. The propagation delay introduced by the switching fabric depends on the design of the switch, as demonstrated in figure below.

<artwork type="svg" src="Figures/time-delay.svg"></artwork>

This triangular switch design was introduced by [Koyama et.al.](https://ieeexplore.ieee.org/document/10821447). Photons entering the switch from different ports need to traverse vastly different number of switching points. For example, photons from input port $$X_0$$ have to traverse at least 5 switching points, while photons from input port $$X_{11}$$ do not have to traverse any at all. Furthermore, if photons from these two input ports are required to undergo Bell-state measurement, both need to be routed to $$BSA_5$$. This requires photons from $$X_0$$ to traverse 10 switching points intoducing the largest possible time delay giventhis design and size of the switch.

The significance of this time delay ultimately depends on the type of photons used. Photons with longer envelopes, such as those emitted from trapped ions, may be more robust to the propagation time delays introduced by the optical switch. Photons with very short envelopes, such as the ones originating from an SPDC source, are expected to be very susceptible to any propagation time delays.

In order to compensate for the propagation time delay, and ensure acceptable visibility at the BSAs, it is neccesary to adjust the path length of the photons with **optical delay lines** (ODLs). For small enough optical switches, it may be possible to characterize the propagation time delays for given photon pairs $$(X_i,X_j)$$ assigned to a particular BSA prior to the opration of the switch. This would allow the ODLs to be set to precomputed configurations based on the connection request patterns. This approach will most likely not scale, at least in its general form, to larger optical switches.

Further complication that arises during the operation of the switch is also related to maintaining indistinguishability of the photons. As the photons traverse the switching points, their **polarization** changes leading to a decrease in the visibility of HOM interference at the BSA. This polarization drift must be characterized and compensated if acceptable levels of visibility are to be maintained. Polarization drift characterization and compensation is a regular step in modern experiments in quantum communications. For example, in the Innsbruck demonstration of remote-entanglement generation over [230m](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803), data acquisition was stopped every 20 minutes in order to correct for the polarizaiton drift. This process took **several minutes**. In the worst case scenario, this process needs to take place after every reconfiguration of the optical switch leading to severely limited multiplexing capabilities.

Finally, given a set of connection requests, the optical switch must compute the state of all switch points to **route** the photons correctly. The reconfigurably non-blocking designs proposed in `[15]` come with efficient routing algorithms that achieve this. Given the need for path-length adjustment with ODLs and polarization drift correction, it is expected that computing the configuration of all switching points will not be the bottleneck during operation of the optical switch.

# Pre-configured Event-driven Tasks

In this section, we discuss synchronization-critical tasks that must be conducted when an event occurs. Most stationary qubits are under the control of a classical analog circuit that includes a local oscillator (LO) coupled to the corresponding frequency of the qubit itself. Avoiding drift between the _understood_ phase of the qubit and the _actual_ phase of the LO is a key part of hardware design for a qubit, but is beyond the scope of this document.

Events can be _local_ to a quantum computer or repeater node, or _remote_, generally implying reception and processing of a classical message.

In some cases, when a memory is used to emit a photon, the ultimate disposition of the qubit in memory might be measurement immediately after the emission of the photon (as in QKD). Alternatively, in systems involving QEC, immediately after emission of the photon, the memory qubit may be encoded into a logical qubit. In general, such events can trigger execution of a local quantum circuit.

For links using HOM-based entanglement generation, inevitably there is a delay between the BSA operation completing successfully and the generation, transmission and reception of the confirmation message. Over distances of a few kilometers, this can require a few microseconds.

# Urgent but Not Synchronization-critical Tasks

Some events trigger a computation, or series of computations, that are too complex to be compiled directly into a form for execution by an ASIC or FPGA. For example, hybrid or adaptive algorithms such as VQE, if executed in a distributed fashion, might require a substantial statistical computation to adjust the parameters used in the creation of the ansatz.

# Host-side Application-level Tasks

The service provided by a quantum network is entangled states, which may be either delivered to applications on quantum computers, held in limited-capability quantum memories for later release and use, or directly measured as creation is completed, corresponding to the capabilities of COMP, STOR and MEAS end node types, respectively.

An application that uses the services of a quantum network passes through several phases:

* **Planning**: selecting application-level tasks and communication partners, including defining application quantum circuits (for distributed computation) or measurement bases and characteristics (for QKD or sensing applications), required distributed quantum states (at the moment, presumed to be Bell pairs), distributed entanglement fidelity, and number or rate of entangled states needed.
* **Computational resource preparation**: for distributed quantum computation, allocation of quantum computing resources attached to the quantum network, compilation of application circuits for processing and consumption of the entangled states delivered by the network service.
* **Connection setup**: the classical process of establishing communication between nodes.  Depending on the network architecture, this may include allocation of resources at repeater nodes.
* **Real-time receipt and disposition of entangled states**: as the network delivers entangled states to the end nodes, they will be consumed by applications, ultimately resulting in classical information which may determine further quantum actions at the level of immediate, result-dependent actions.
* **Near-real time computation**: many quantum algorithms are hybrid classical/quantum computation and may require larger-scale adaptation or recompilation of application quantum circuits.
* **Post-quantum processing**: the classical data generated by the quantum circuits or measurements can be delivered to larger classical computation and communication systems, e.g. for use as cryptographic keys or as part of a much larger computation. At this point, processing is completely decoupled from the quantum network.
* **Connection teardown**: After completion of the quantum network service requested by the application or larger IT service (e.g., encrypted classical connection), resources along the communication path can be recovered; partially complete entangled states are discarded or repurposed, and physical resources reallocated to other connections.
* **Computational resource release**: reserved quantum computational resources are released.
* **Completion**: the end-to-end hybrid quantum+classical service is terminated.

All of the above except those marked real-time and near-real time are almost entirely insensitive to timing issues, except as necessary for the end-to-end service to meet the users' needs. If allocated resources sit unused for extensive periods of time, the service delivery of the network as a whole may be negatively impacted; introduction of proper pricing or admission control may be needed to resolve such issues.

# Background Tasks

Network operations include a number of tasks that monitor and maintain the integrity and performance of the network. In the case of a quantum network, uses of the quantum portion of the network can often be deferred until the network is idle or pre-scheduled time slots arrive, in order to minimize the impact on application requests.  Once the quantum operations are begun, of course, they are subject to all of the constraints listed above, but the accompanying classical calculation and inter-node reconciliation can proceed in the background.

Such tasks include:

* **Link monitoring**: Each link must be monitored continuously in order to inform routing (below) and RuleSet creation during connection setup.  Reconstruction of the link density matrix and entanglement success rates involve classical information sharing between the two nodes at opposite ends of the link. This information must be shared reliably but does not have hard real-time constraints, as so is well suited to transmission over a reliable protocol such as TCP without concern for delays. The required classical information is the outcomes of measurements of the quantum portion of the link. That data can be collected from entangled states specifically assigned to the link monitoring task.  It can also be collected from application-targeted uses of the link, provided that appropriate coordination can be achieved and connection privacy maintained.
* **Routing**: Creation and update of routing tables at each node is an ordinary, distributed classical task that shares the information collected about links as above. The expected completion time of this tasks should be quick enough that the network converges to provide seamless service upon topology changes.  Unless nodes are mobile, propagation and recalculation of such changes at the level of seconds should be acceptable.
* **Malicious use monitoring**: It is known that a hijacked or malfunctioning repeater can be used to impede the overall service of the network or even to partition the network. It is also known that QKD-derived monitoring of the network using randomly selected measurement bases on a portion of the network capacity can serve as a detection mechanism for this malicious behavior.

--- back

# References

## Normative

## Informative

~~~text
* C.K. Hong, Z.Y. Ou, and L. Mandel, Measurement of subpicosecond time intervals between two photons by interference, [_Phys. Rev. Lett._ **59**, 2044 (1987)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.59.2044).
* A. M. Branczyk, Hong-Ou-Mandel Interference, [_arXiv:1711.00080_ (2017)](https://arxiv.org/abs/1711.00080).
* V. Krutyanskiy _et al._, Entanglement of Trapped-Ion Qubits Separated by 230 Meters, [_Phys. Rev. Lett._ **130**, 050803](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.130.050803) (2023).
* R. H. Hadfield, Single-photon detectors for optical quantum information applications, [_Nature Photonics_ **3**, 696](https://www.nature.com/articles/nphoton.2009.230) (2009).
* Single Quantum, [Datasheet](https://singlequantum.com/wp-content/uploads/2022/12/SQ-General-Brochure.pdf).
* D. Nadlinger, Device-independent key distribution between trapped-ion quantum network nodes, DPhil Thesis, [Oxford University (2022)](https://ora.ox.ac.uk/objects/uuid:604c53b9-8df8-4e45-8103-10fd81eb3366).
* D. Leibfried, R. Blatt, C. Monroe, and D. Wineland, Quantum dynamics of single trapped ions, [_Rev. Mod. Phys._ **75**, 281 (2003)](https://journals.aps.org/rmp/abstract/10.1103/RevModPhys.75.281).
* C. Gardiner, and P. Zoller, The Quantum World of Ultra-Cold Atoms and Light: The Physics of Quantum-Optical Devices, [Imperial College Press, (2015)](https://www.amazon.co.jp/Quantum-World-Ultra-Cold-Atoms-Light/dp/1783266163).
* J.I. Cirac, and P. Zoller, Quantum Computations with Cold Trapped Ions, [_Phys. Rev. Lett._ **74**, 4091 (1995)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.74.4091).
* F. Schmidt-Kaler _et. al._, Realization of the Cirac–Zoller controlled-NOT quantum gate, [_Nature_ **422**, 408 (2003)](https://www.nature.com/articles/nature01494).
* K. Molmer, and A. Sorensen, Multiparticle Entanglement of Hot Trapped Ions, [_Phys. Rev. Lett._ **82**, 1835 (1999)](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.82.1835).
* J. Benhelm _et al._, Towards fault-tolerant quantum computing with trapped ions, [_Nature Physics_ **4** 463 (2008)](https://www.nature.com/articles/nphys961).
* J. Altepeter, D.F.V. James, and P.G. Kwiat, Qubit quantum state tomography, [_Lecture Notes in Physics_ **649**, 113 (2004)](https://link.springer.com/chapter/10.1007/978-3-540-44481-7_4).
* B. E. A. Saleh, and M. C. Teich, Fundamentals of Photonics, [John Wiley & Sons, (2019)](https://onlinelibrary.wiley.com/doi/book/10.1002/0471213748).
* R. J. Drost, T. J. Moore, and M. Brodsky, Switching Networks for Pairwise-Entanglement Distribution, [_Journal of Optical Communications and Networking_, **8**, 331 (2016)](https://opg.optica.org/jocn/fulltext.cfm?uri=jocn-8-5-331&id=340335).
* M. Koyama, C. Yun, A. Taherkhani, N. Benchasattabuse, B. O. Sane, M. Hajdušek, S. Nagayama, R. Van Meter, Optimal Switching Networks for Paired-Egress Bell State Analyzer Pools, [_arXiv:2405.09860_ (2024)](https://arxiv.org/abs/2405.09860).
* Polatis Series 6000i Instrument Optical Matrix Switch, [https://www.viavisolutions.com/en-us/literature/polatis-series-6000-osm-network-switch-module-data-sheets-en.pdf](https://www.viavisolutions.com/en-us/literature/polatis-series-6000-osm-network-switch-module-data-sheets-en.pdf).
~~~
