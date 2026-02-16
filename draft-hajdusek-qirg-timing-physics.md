---
title: "Timing Regimes in Quantum Networks and their Physical Underpinnigs"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-todo-yourname-protocol-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
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
 -  ins: M. Hajdusek
    fullname: Michal Hajdusek
    organization: Keio University
    email: your.email@example.com

normative:

informative:

...

--- abstract

TODO Abstract


--- middle

## Prologue

In 1982, Digital Equipment Corporation, Intel, and Xerox published  __The Ethernet: A Local Area Network Data Link Layer and Physical Layer Specifications__. This 120-page document specifies pretty much everything: diameter of the coaxial cable, its impedance, dispersion, maximum cable length, voltages and currents, signal rise times, etc. The types of physical connectors allowed. How a bit is encoded in the signal. How a frame is demarcated. How collisions are detected. The format of messages. Addressing. Multicasting. Polynomials for error correction. It's ALL there.

Equally importantly, it specifies __timing requirements__.  For example, the rise time for a signal on the coaxial cable shall be $25\pm 5$ nanoseconds. The total worst-case round-trip delay is calculated in a table to be $46.38\mu\textrm{s}$. How the entries in that table are combined to produce that number is fairly obvious; however, the numerical entries themselves are mostly unjustified in the specification itself, only stated. One exception is the statement, "Rise and fall times meet 10,000 series ECL requirements," referring to a specific series of well-known digital emitter-coupled logic parts, and hence incorporating a great deal of prior knowledge and work by reference.

In the quantum world, we are starting from first principles. Hence, we must begin at the beginning. We want to have specifications like Ethernet's, but first we must describe how the entries in e.g. the physical propagation delay budget are determined. The role of this document is to provide the underpinnings that give a shared understanding of how the basic numbers are determined and how they can be combined in a particular system design.

Thanks, DIX Ethernet creators, for showing the way!

# Introduction

Quantum networks that distribute end-to-end entanglement involve a number of tasks with varying demands on timing precision and jitter. The design of a quantum network will involve a layered protocol architecture where different layers take responsibility for meeting these differing constraints. This document describes the various timing regimes, from most to least stringent, in order to assist the process of making key design decisions.

The range of time scales of interest extends from ensuring the sub-wavelength stability of optical paths up to batch monitoring of the operation of the network itself. Light with a wavelength of $1.5\mu\text{m}$ (common in communications, including quantum communications) has a frequency of approximately $200$ THz ($2\times 10^{14}$ Hertz), for a cycle time of $5\times 10^{-15}$ seconds.  Ranging from sub-wavelength stabilization through background operations such as routing, therefore, covers some 16 or more decimal orders of magnitude.  Add in a 24-hour thermal drift that must be compensated for in many cases, and we reach twenty decimal orders of magnitude from the bottom to the top. Naturally, meeting this range of demands requires the use of a variety of mechanisms.  This document avoids specifying solutions to the problems, and instead presents the functions and how their requirements are calculated (or measured).  Thus, each individual network design should apply the methods introduced here and present a numerical summary of the resulting values, after which corresponding solutions can be proposed and implemented.

Summary of timing regimes:

* A. __interferometric stabilization:__ photon wavepacket overlap, technology dependent, roughly nanoseconds
* B. __opening and closing of detector timing windows, detector recovery time:__ nanoseconds to microseconds
* C. __measurement basis selection (if required in BSA):__ performance will constrain entanglement attempt rate
* D. __optical switch control:__ switching of trains of wave packets
* E. __pre-configured event-driven tasks such as timing-triggered or measurement-triggered execution of quantum circuits:__ microseconds
* F. __urgent but not synchronization-critical tasks (e.g. execution of classical code that processes RuleSet messages and selects or creates new quantum circuits for execution):__ milliseconds
* G. __host-side application-level tasks (e.g. post-measurement operations):__ milliseconds
* H. __background tasks (link tomography calculations, routing table updates):__ seconds to minutes

Some of these can only be achieved using high-quality hardware, while others are software tasks. Detailed analysis of these regimes will affect core software design in each network node type.

## A. Interferometric Stabilization

Entanglement distribution in quantum networks is performed by entanglement swapping (ES) on photonic qubits.
Central to photonic ES is the Hong-Ou-Mandel (HOM) interference [1,2], regardless of the photonic qubit encoding or of the particular protocol implementing photonic ES.
We begin by introducing the notation used, giving a brief overview of the effect, as well as discussing how to quantify the effect.
We then continue with a discussion of the requirements that must be satisfied in order to observe the effect.

### A.1. Hong-Ou-Mandel interference

Consider two photons incident on a beamsplitter (BS) with reflectivity $r$.
We label the input modes $a$ and $b$.
The output modes of the BS are labelled with $a$ and $b$ as well, with the understanding that output mode $a$ corresponds to the input mode $a$ being transmitted.
Similarly for output mode $b$, as shown in the Figure below.

<p align="center">
  <img src="Figures/HOM.svg"/>
</p>

The input state can be expressed as
$$|\psi ^{\text{in}}\rangle_{ab} = \hat{a}^{\dagger}_j\hat{b}^{\dagger}_k |0\rangle _{ab},$$
where $\hat{a}^{\dagger}_j$ and $\hat{b}^{\dagger}_k$ are the bosonic creation operators corresponding to BS input modes $a$ and $b$, respectively.
The indices $j$ and $k$ represent other properties of the photons that determine how distinguishable the photons are.
For example, $j$ and $k$ could represent
- polarizations (for polarization-encoded qubits),
- spectral modes,
- temporal modes (for time-bin qubits),
- arrival time (discussed in Section B),
- transverse spatial mode.

We are interested in the observed behavior at the output modes of the BS.
There are four possible cases that may occur:
- Case A: photon in mode _a_ is reflected, while photon in mode _b_ is transmitted.
- Case B: both photons are transmitted.
- Case C: both photons are reflected.
- Case D: photon in mode _a_ is transmitted, while photon in mode _b_ is reflected.

The action of the BS is represented by a unitary operator $\hat{U}_{ab}$,
$$\hat{a}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{1-r}\hat{a}^{\dagger} + \sqrt{r}\hat{b}^{\dagger}, \qquad \hat{b}^{\dagger} \xrightarrow{\hat{U} _{ab}} \sqrt{r}\hat{a}^{\dagger} - \sqrt{1-r}\hat{b}^{\dagger}.$$
The output state of the two photons is
$$|\psi ^{\text{out}}\rangle _{ab} = \hat{U} _{ab} |\psi ^{\text{in}}\rangle _{ab} = \left( \sqrt{r(1-r)}\hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + r \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - (1-r) \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \sqrt{r(1-r)} \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
For a 50:50 BS with $r=1/2$, we obtain,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{2} \left( \hat{a} ^{\dagger} _{j} \hat{a} ^{\dagger} _{k} + \hat{a} ^{\dagger} _{k} \hat{b} ^{\dagger} _{j} - \hat{a} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} - \hat{b} ^{\dagger} _{j} \hat{b} ^{\dagger} _{k} \right) |0\rangle _{ab}.$$
From this expression, we can see that when $j=k$, in other words when the input photons are indistinguishable, the output state has the following form,
$$|\psi ^{\text{out}}\rangle _{ab} = \frac{1}{\sqrt{2}} \left( |2\rangle_a - |2\rangle_b \right).$$
The probability amplitudes for the cases where both input photons are transmitted or both reflected (Cases B and C in the figure above) interfere destructively.
Perfectly indistinguishable input photons always exit the BS in the same ouput mode.
It is this interference effect that is at the heart of quantum networking.

In order to quantify the effect that distinguishability has on the HOM interference, we consider the __probability of a coincidence detection__, $p_{\text{coin}}$, where one photon is detected in the BS output mode $a$, and the other photon in output mode $b$.
This probability is defined as
$$p_{\text{coin}} = \langle\psi^{\text{out}}|_{ab} \hat{P}_a \otimes \hat{P}_b |\psi^{\text{out}}\rangle _{ab},$$
where $\hat{P}_i$, for $i=a,b$, are the projection operators representing a detection of a single photon in output mode $i$ of the BS.
For completely indistinguishable input photons that undergo the full HOM interference, we have $p _{\text{coin}}=0$.
On the other hand, for fully distinguishable photons, the probability of a coincidence detection attains its maximum value $p _{\text{coin}}=1/2$.

An often-used measure that quantifies the degree of HOM interference is the __visibility__ $V$, defined via the probability of a coincidence detection,
$$V = \frac{p_{\text{coin}}^{\text{max}} - p_{\text{coin}}^{\text{min}}}{p_{\text{coin}}^{\text{max}}} = 1 - 2 p_{\text{coin}}^{\text{min}},$$
where we used the fact that the maximum probability of a coincidence detection is $1/2$.
We observe that the visibility varies from $V=0$ for fully distinguishable input photons to $V=1$ for perfectly indistinguishable ones.

Visibility $V$ plays a useful role when modelling the effects of imperfect HOM interference in the context of entanglement swapping.
Consider the case when the input photons $a$, $b$ are entangled with auxiliary systems $s_1$ and $s_2$, respectively.
The BSA performs ES by measuring the input photons, entangling systems $s_1$ and $s_2$ in the process.
Fidelity of the new entangled pair is directly proportional to the visibility $V$ of the HOM interference.
Denote by $\rho_{s_1s_2}^{\text{no-deph}}$ the density matrix resulting from an ideal ES at the BSA with unit visibility of the HOM interference.
Non-ideal HOM interference can be modelled as a two-qubit dephasing [3],
$$\rho_{s_1s_2} = V \times \rho_{s_1s_2}^{\text{no-deph}} + (1 - V) \times \rho_{s_1s_2}^{\text{deph}},$$
where $\rho_{s_1s_2}^{\text{deph}}$ is a fully dephased state obtained by setting all off-diagonal elements of $\rho_{s_1s_2}^{\text{no-deph}}$ to zero.

In the following subsections, we address and quantify how distinguishable photons affect the visibility of the HOM interference.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
