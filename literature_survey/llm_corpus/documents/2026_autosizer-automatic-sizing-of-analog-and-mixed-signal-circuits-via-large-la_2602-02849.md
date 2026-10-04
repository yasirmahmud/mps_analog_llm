---
title: "AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents"
authors: "Yu, Xi; Torbunov, Dmitrii; Mandal, Soumyajit; Ren, Yihui"
date: "2026/02/02"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2602.02849"
pdf: "papers/2026_autosizer-automatic-sizing-of-analog-and-mixed-signal-circuits-via-large-la_2602-02849.pdf"
sha256: "5beb9ff37ae63128d2cfca24627c362c29f3d68e47208d1c96c796dabaee4cc5"
pages_extracted: 26
---

# AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AUTOSIZER: Automatic Sizing of Analog and Mixed-Signal Circuits
via Large Language Model (LLM) Agents
Xi Yu 1
Dmitrii Torbunov 1
Soumyajit Mandal 2
Yihui Ren 1
Abstract
The design of Analog and Mixed-Signal (AMS)
integrated circuits remains heavily reliant on
expert knowledge, with transistor sizing a ma-
jor bottleneck due to nonlinear behavior, high-
dimensional design spaces, and strict performance
constraints. Existing Electronic Design Automa-
tion (EDA) methods typically frame sizing as
static black-box optimization, resulting in inef-
ficient and less robust solutions. Although Large
Language Models (LLMs) exhibit strong reason-
ing abilities, they are not suited for precise nu-
merical optimization in AMS sizing. To address
this gap, we propose AUTOSIZER, a reflective
LLM-driven meta-optimization framework that
unifies circuit understanding, adaptive search-
space construction, and optimization orchestra-
tion in a closed loop. It employs a two-loop opti-
mization framework, with an inner loop for circuit
sizing and an outer loop that analyzes optimiza-
tion dynamics and constraints to iteratively refine
the search space from simulation feedback. We
further introduce AMS-SIZINGBENCH, an open
benchmark comprising 24 diverse AMS circuits
in SKY130 CMOS technology, designed to evalu-
ate adaptive optimization policies under realistic
simulator-based constraints. AUTOSIZER experi-
mentally achieves higher solution quality, faster
convergence, and higher success rate across vary-
ing circuit difficulties, outperforming both tradi-
tional optimization methods and existing LLM-
based agents. Our code is available at https://
github.com/yuxi120407/AutoSizer.
1
Artificial Intelligence Department, Brookhaven National Lab-
oratory, Upton, NY 2
Instrumentation Department, Brookhaven
National Laboratory, Upton, NY 11973 . Correspondence to: Xi
Yu <xyu1@bnl.gov>.
Proceedings of the 43rd
International Conference on Machine
Learning, Seoul, South Korea. PMLR 306, 2026. Copyright 2026
by the author(s).
1. Introduction
The design of Analog and Mixed-Signal (AMS) circuits
typically follows a multi stage workflow that begins with
topology selection and transistor sizing and then proceeds
to layout level steps such as placement routing and verifi-
cation while optimizing power efficiency maximizing per-
formance and minimizing area to meet target specifications.
These multi-stage workflows are complex, requiring time-
consuming simulations along with human expertise. Among
these stages, circuit sizing is particularly challenging be-
cause it involves a high dimensional design space and com-
plex trade offs among multiple performance metrics, where
small variations in device parameters can lead to significant
and often non linear changes in circuit behavior and over-
all performance. As a result, automatic circuit sizing has
emerged as a crucial problem in electronic design automa-
tion (EDA) and has attracted increasing research attention.
Traditional methods formulate the circuit sizing as an
optimization problem, such as Bayesian Optimization
(BO) (Touloupas et al., 2021; Lyu et al., 2018), Rein-
forcement Learning (Wang et al., 2020), and genetic al-
gorithms (Cohen et al., 2015). However, these approaches
treat the circuit design space as a static black box, ignoring
domain-specific analog design expertise and often yield-
ing suboptimal solutions. Recently, LLM-based agents have
shown promise in symbolic reasoning and scientific problem
solving. Several studies have explored LLMs for automated
analog circuit design, focusing on circuit knowledge repre-
sentation and sizing optimization. EEsizer (Liu & Chitnis,
2025b) directly uses an LLM for both circuit understanding
and sizing optimization. Other LLM-agent approaches (Liu
et al., 2024; Somayaji & Li, 2025) emphasize improving cir-
cuit comprehension through document parsing or structured
knowledge bases. LEDRO (Kochar et al., 2025) leverages
LLMs to reduce the design search space and improve opti-
mization efficiency, while LLMACD (Xu et al., 2025) incor-
porates a pre-trained Transistor Behavioral Model (TBM) to
enhance design consistency. Despite these advances, most
existing agents rely on fixed optimization algorithms and
single-stage workflows, in which the LLM interprets the
circuit, applies a predetermined optimization strategy, and
outputs results without iterative self-reflection or feedback-
1
arXiv:2602.02849v2
[cs.AI]
28
May
2026

## Page 2

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
driven refinement. This limits their ability to adapt the
optimization process itself in response to intermediate out-
comes, particularly for complex AMS circuits with sparse
feasibility regions and competing design objectives.
To address this gap, we propose AUTOSIZER, a reflective
meta-optimization framework based on multi-agent LLMs
for automatic sizing optimization of analog and mixed-
signal circuits. AUTOSIZER integrates circuit understand-
ing, adaptive search-space construction, and optimization
algorithm orchestration into a unified, closed-loop workflow
with an evaluation agent, enabling efficient and robust ex-
ploration of high-dimensional design spaces. Starting from
user specifications and circuit netlists, an LLM-based circuit
understanding agent analyzes the circuit topology to iden-
tify key design variables and infer their functional roles and
performance sensitivities. Based on this analysis, a search-
space decision agent prioritizes the identified variables and
dynamically assigns feasible ranges, enabling the frame-
work to focus optimization effort on the most impactful
parameters. Sizing optimization is carried out in a two-loop
optimization framework. The inner loop performs numer-
ical circuit sizing using a configurable optimization engine
that supports multiple algorithms, such as Latin Hypercube
Sampling (LHS), genetic algorithms, Bayesian Optimiza-
tion (BO), and related methods, and evaluates simulation
results against user-defined specifications. The outer loop
performs reflective search-space evaluation and refine-
ment by analyzing optimization history, convergence behav-
ior, and constraint satisfaction. Based on this feedback, the
framework adaptively revises variable priorities, tightens or
expands parameter ranges, and updates circuit understand-
ing, enabling progressive and data-efficient refinement of
the design space across iterations.
To support systematic and reproducible evaluation of adap-
tive sizing strategies, we introduce AMS-SIZINGBENCH, a
new benchmark suite for analog and mixed-signal circuit siz-
ing optimization built entirely on the open-source SKY130
CMOS technology. The benchmark comprises of 24 repre-
sentative analog and mixed-signal circuits, spanning a wide
range of design complexities from basic building blocks to
advanced functional modules, and is carefully organized to
provide a fully open and standardized sizing environment.
Each benchmark instance includes the SKY130-based cir-
cuit netlist, a clearly defined set of sizing variables and
optimization constraints, and a complete simulation work-
flow for performance evaluation. AMS-SIZINGBENCH thus
provides a fully open and standardized testbed for evaluating
adaptive optimization policies and facilitates fair compari-
son, extensibility, and reuse in AMS sizing research.
Our contributions can be summarized as follows:
• An LLM-driven two-loop multi-agent framework
for analog and mixed-signal sizing. We propose
AUTOSIZER, a multi-agent LLM framework that that
formulates AMS sizing as a meta-optimization prob-
lem, integrating circuit understanding, adaptive search-
space construction, and optimization algorithm or-
chestration into a unified closed-loop workflow. The
framework combines an inner sizing optimization loop
that supports multiple optimization algorithms with an
outer search-space refinement loop that analyzes opti-
mization history, convergence behavior, and variable
impact, enabling adaptive revision of variable priori-
ties and parameter ranges for efficient exploration of
high-dimensional design spaces.
• AMS-SIZINGBENCH: an open benchmark for
AMS circuit sizing systematic evaluation. We
present AMS-SIZINGBENCH, a fully open-source
benchmark suite built on the SKY130 CMOS technol-
ogy, comprising 24 representative analog and mixed-
signal circuits spanning a wide range of design com-
plexities. Each benchmark instance includes a stan-
dardized circuit netlist, clearly defined sizing variables
and constraints, and a complete simulation and evalu-
ation workflow, enabling reproducible evaluation and
fair comparison of automated sizing methods.
• Comprehensive experimental evaluation and per-
formance gains. We conduct extensive experiments
on AMS-SIZINGBENCH and show that AUTOSIZER
consistently outperforms both traditional optimization
methods and existing LLM-based agents in terms of
convergence speed, solution quality, and robustness
across diverse circuit types.
Conflict of Interest Disclosure. The authors declare no
financial conflicts of interest related to this work.
2. Related Work
Automated sizing of AMS circuits has been a long-standing
pursuit in the EDA community, evolving from manual
heuristics to simulation-driven optimization and emerging
LLM-based agent workflows.
2.1. Traditional Circuit Sizing Methods
Traditional circuit sizing methods formulate design as a
static black-box optimization problem, obtaining perfor-
mance metrics through repeated simulations. Early ap-
proaches use general-purpose algorithms like simulated an-
nealing and genetic algorithms (Wolfe & Vemuri, 2003;
Pessoa et al., 2018), which require tens of thousands of
simulations and lack mechanisms for adapting the search
space or transferring knowledge across runs. Bayesian opti-
mization (BO) improves sample efficiency using surrogate
models (Lyu et al., 2018; Liu et al., 2014; Shahriari et al.,
2

## Page 3

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Table 1. Comparison of LLM-based Agents circuit-sizing methods and benchmark coverage.
Method
Self
Reflection
Adaptive
Optimization
Tools
Search Space
construction
Benchmark
(Circuit Type & # Num)
Amp. Osc. SC
Logic/
Comp.
Ref./
Power
Filter # Num
AmpAgent (Liu et al., 2024) ✗ ✗ ✗ ✓ ✗ ✗ ✗ ✗ ✗ 7
ADO-LLM (Yin et al., 2024) ✗ ✗ ✗ ✓ ✗ ✗ ✓ ✗ ✗ 2
LLM-USO (Somayaji & Li, 2025) ✗ ✗ ✗ ✓ ✗ ✗ ✓ ✓ ✗ 4
LLMACD (Xu et al., 2025) ✗ ✗ ✗ ✓ ✗ ✗ ✗ ✗ ✗ 2
LEDRO (Kochar et al., 2025) ✓ ✗ ✓ ✓ ✗ ✗ ✗ ✗ ✗ 22
EEsizer (Liu & Chitnis, 2025b) ✓ ✗ ✗ ✓ ✓ ✗ ✓ ✗ ✗ 6
AutoSizer ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓ 24
2015), but still relies on fixed parameter spaces and prede-
fined acquisition strategies, and struggles as dimensionality
increases. More recently, reinforcement learning (RL) meth-
ods (Wang et al., 2018; Settaluri et al., 2020; Yang et al.,
2021) demonstrate improved scalability and transfer learn-
ing (Zhang et al., 2023; Ahmadzadeh et al., 2025b), yet
still require hundreds of computationally expensive sim-
ulations and yield opaque, task-specific policies, limiting
interpretability and hindering industrial adoption.
2.2. LLM Agents
Recent advances in large language models have enabled
agent-based systems that move beyond passive text gener-
ation toward autonomous, goal-oriented behavior. These
LLM agents leverage structured planning, tool interaction,
and iterative feedback loops to decompose tasks and refine
solutions (Yao et al., 2022; Chen et al., 2024). Many frame-
works incorporate memory mechanisms and self-reflective
reasoning to accumulate experience and adapt strategies
over time. They have been successfully applied across
diverse domains including software engineering (Jimenez
et al., 2024), robotics (Brohan et al., 2023; Black et al.,
2024), data science (Guo et al., 2024; Hong et al., 2025),
and scientific research (Yu et al., 2025; Yamada et al., 2025;
Baek et al., 2025). In electronic design automation, recent
works explore LLM-based approaches for analog circuit de-
sign: AnalogXpert (Zhang et al., 2025) automates topology
synthesis, EEschematic (Liu & Chitnis, 2025a) generates
schematics via multimodal agents, AnalogCoder-Pro (Lai
et al., 2026) unifies circuit generation and optimization, and
SimuGen (Ren et al., 2025) constructs block diagram-based
simulation models through multi-modal agents.
2.3. LLM-based Agents for Circuit Sizing
Recent work explores using LLMs as autonomous agents
for analog circuit sizing. Unlike black-box optimization
methods, LLM-based approaches leverage language mod-
els to reason about circuit structure, interpret simulation
feedback, and guide optimization.. AmpAgent (Liu et al.,
2024) retrieves the amplifier-specific knowledge from es-
says and guides the LLM to optimize the circuit sizing.
LLM-USO (Somayaji & Li, 2025) constructs a graph knowl-
edge space and leverage it for the next circuit design. LL-
MACD (Xu et al., 2025) generates Transistor Behavioral
Circuit Representations (TBCRs) that meet performance tar-
gets using knowledge-embedded prompts. LEDRO (Kochar
et al., 2025) utilized LLMs alongside fixed optimization
techniques to iteratively refine the design space for analog
circuit sizing. AnaFlow (Ahmadzadeh et al., 2025a) lever-
ages both reasoning sizing and optimizer sizing. However,
existing LLM-based sizing agents rely on either fixed search
spaces or fixed optimization algorithms with predetermined
configurations, often validated on a single circuit class (e.g.,
amplifiers). This rigidity limits scalability across diverse de-
signs. AutoSizer addresses these limitations by formulating
circuit sizing as a reflective meta-optimization problem, dy-
namically refining search spaces and orchestrating multiple
optimization algorithms based on intermediate optimiza-
tion dynamics. We evaluate AutoSizer on 24 diverse AMS
circuit topologies based on SKY130 CMOS technology, in-
cluding OTAs, oscillators, voltage references, filters, and
voltage regulators, demonstrating robust performance across
varied circuit types and specifications.
3. Method
3.1. Problem Formulation
We formulate AMS circuit sizing as a simulation-driven con-
strained optimization problem. Let x ∈ X denote the vector
of sizing parameters and y = [y1, . . . , ym] = f(x) de-
note the corresponding performance metrics obtained from
simulation. A scalar figure of merit (FoM) is defined to
capture the trade-off between metrics to be maximized and
minimized:
FoM(y) =
Q
i∈D ỹi
Q
j∈M ỹj
, (1)
3

## Page 4

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Circuit
Understanding
Topology & Variables
Circuit Sizing Optimization
Simulation Results Analysis & Evaluation
Inner Loop
Optimization Engine
Algorithm Pools:
BO, LHS, Genetic…
➢ Convergence check, performance analysis
➢ User Specification Check
Search Space Evaluation
➢ Variables Impact
➢ Convergence Analysis
User
Specification
Circuit Netlist &
Simulation
Outer Loop
Satisfied
Search Space
Decision
Variable Priority
& Ranges
No
Satisfied
No
Satisfied
Self-Reflection
✓ Circuit
re-understanding
✓ Search Space
re-decision
Algorithm Usages
History
Performance
Distribution
Figure 1. AutoSizer workflow. The framework combines LLM-based circuit understanding with an inner loop that sequentially selects,
configures, and applies multiple sizing algorithms and an outer loop that adaptively refines the search space, forming a two-loop
closed-loop architecture that enables iterative refinement of variable priorities and parameter ranges based on optimization feedback.
where D and M denote the sets of metrics to be maximized
and minimized, respectively, and ỹi = yi/yi,spec are nor-
malized with respect to their specification targets yi,spec.
This normalization ensures FoM values are dimensionless
and comparable across circuits with different performance
scales. A design is feasible when ỹi ≥ 1 for all i ∈ D and
ỹj ≤ 1 for all j ∈ M. The sizing task is expressed as:
max
x∈X
FoM(f(x)) s.t. ỹi ≥ 1 ∀i ∈ D, ỹj ≤ 1 ∀j ∈ M
(2)
3.2. Overview of the AutoSizer Framework
AutoSizer is a multi-agent, LLM-driven framework de-
signed for reflective automatic sizing optimization of AMS
circuits. Given user specifications and a circuit netlist, Auto-
Sizer integrates circuit understanding, adaptive search-space
construction, and simulation-driven optimization into a uni-
fied, closed-loop workflow. Multiple specialized agents
collaborate to iteratively refine design decisions using simu-
lation feedback.
As illustrated in Fig. 1, AutoSizer adopts a two-loop opti-
mization architecture. The inner loop optimizes device sizes
by invoking a configurable optimization engine and eval-
uating candidate designs through circuit simulation. The
outer loop analyzes optimization outcomes, including con-
vergence behavior and performance distributions, to eval-
uate optimization effectiveness and constraint satisfaction
under the current formulation. Based on this analysis, the
framework revises the optimization problem, updating vari-
able priorities, parameter ranges, and circuit understand-
ing through self-reflection, enabling progressive and data-
efficient exploration of high-dimensional design spaces.
3.3. LLM-Based Circuit Understanding
The purpose of the circuit understanding module is to extract
high-level structural and functional knowledge from circuit
netlists to guide sizing optimization. Given a circuit netlist
and user specifications as input, the LLM produces a struc-
tured summary of the circuit topology, key sizing variables,
and their qualitative impact on performance metrics.
The model analyzes circuit connectivity, signal paths, and
biasing structures to identify major functional blocks, and
maps optimization variables to the corresponding circuit
components (See circuit understanding prompt in Ap-
pendix B.1). Based on this analysis, it estimates the relative
influence of sizing variables on key metrics and identifies
fundamental trade-offs.
As can be seen in Figure. 2, circuit understanding agent
provides an interpretable abstraction of the design space and
serves as an explicit input to reflective optimization, directly
informing the adaptive search-space construction stage by
prioritizing impactful variables and guiding range selection.
3.4. Adaptive Search-Space Construction
The adaptive search-space construction module aims to re-
duce optimization complexity by focusing computational
effort on the most impactful design parameters. Its input
consists of the structured circuit understanding output and
optimization dynamics and feasibility feedback from pre-
vious iterations, and its output is a dynamically defined
search space with prioritized variables and feasible parame-
ter ranges.
Directly optimizing over all possible sizing variables with
wide parameter ranges leads to prohibitively large search
4

## Page 5

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
LLM-Based Circuit Understanding Output
Circuit Topology: Telescopic operational transconductance amplifier (OTA)
with an NMOS differential input pair, cascoded NMOS and PMOS stages, and
PMOS current-mirror loads.
Key Sizing Variables: Wdiff (input differential pair), Wcasc (cascode devices),
Wtail (tail current source), Wload (PMOS load devices).
Performance Sensitivity: DC gain is primarily influenced by Wdiff and Wcasc.
UGBW is dominated by Wdiff, with secondary dependence on current-scaling
variables. Power consumption is mainly controlled by Wtail and Wload.
Key Trade-offs: Improving gain and bandwidth increases power consumption
and parasitic capacitance, requiring careful balance for FoM optimization.
Figure 2. LLM-generated circuit understanding for the telescopic
OTA, including topology analysis, key sizing variables, perfor-
mance sensitivities, and design trade-offs.
spaces and poor sample efficiency. In practice, only a sub-
set of variables significantly influences circuit performance,
and their effective operating ranges are often much narrower
than the full technology limits. Inspired by common analog
design practice, AutoSizer adopts a progressive, feedback-
driven search-space expansion strategy: it begins with a
compact search space defined by a small set of high-impact
variables and conservative ranges (See Figure 3), and incre-
mentally expands the search space only when optimization
outcomes indicate infeasibility or stagnation.
First-Round Adaptive Search-Space Construction
Optimization Target: Maximize DC gain and unity-gain bandwidth (UGBW)
while minimizing power consumption.
Variable Prioritization: Wdiff is identified as the most critical variable due to
its direct control of input transconductance, which dominates both gain and
UGBW. Wtail and Wload are ranked next, as they scale bias current and govern
the primary power–performance trade-off. Wcasc is treated as a secondary
knob that mainly affects gain through output resistance.
Active Variables and Ranges: Wdiff ∈ {0.84, 1.26, 1.68, 2.10, 2.52},
Wtail ∈ {0.84, 1.47, 2.10, 2.52}, Wload ∈ {0.84, 1.47, 2.10, 2.52}.
These ranges are chosen to sparsely yet effectively explore performance scaling
and the FoM trade-off.
Fixed Variable: Wcasc is fixed at 1.89 to reduce search dimensionality while
preserving high output resistance for adequate DC gain.
Search-Space Reduction: The original full space of 94
= 6561 combi-
nations is reduced to 5 × 4 × 4 = 80 combinations, achieving an ∼82×
reduction while maintaining coverage of the dominant design trade-offs.
Figure 3. LLM-generated adaptive search space configuration for
the first optimization round, showing variable prioritization, range
selection, and dimensionality reduction strategy.
This reduces unnecessary exploration in early stages, focus-
ing effort on promising regions. Upon stagnation or infeasi-
bility, AutoSizer systematically reformulates the problem
by unfixing variables or expanding ranges.
3.5. Two-Loop Optimization Framework
AutoSizer adopts a two-loop optimization framework that
separates numerical parameter search from adaptive for-
mulation of the optimization problem. The inner loop is
responsible for optimizing sizing parameters within a fixed
search space, while the outer loop evaluates the effectiveness
of the current search space and revises it when necessary.
In the inner loop, AUTOSIZER adaptively selects and con-
figures optimization algorithms from a predefined algorithm
pool (See Algorithm Orchestration Prompt in Appendix B.3)
based on the current optimization stage, available simula-
tion history, and search-space characteristics determined
by the LLM-based search-space decision agent. Candidate
designs are evaluated via circuit simulation and scored us-
ing the specifications and FoM, with results shared across
algorithms. The inner loop terminates upon convergence or
budget exhaustion.
The outer loop monitors optimization outcomes produced
by the inner loop, including feasibility status, convergence
behavior, and performance trends. If the inner loop fails to
identify feasible solutions or exhibits stagnation, the outer
loop adaptively revises the optimization problem by adjust-
ing parameter ranges, activating additional variables, or up-
dating variable priorities. This separation of responsibilities
enables efficient exploration in early stages while preserving
robustness against suboptimal initial search-space assump-
tions. The complete two-loop optimization procedure is
formally described in Algorithm 1.
Algorithm 1 Two-Loop Optimization Framework
Require: Circuit netlist N, user specifications S, initial
search space X0
Ensure: Optimized sizing parameters x⋆
1: Initialize search space decision X ← X0
2: Initialize optimization history H ← ∅
3: while termination condition not met do
4: Inner Loop: Algorithm Selection & Execution
5: Select optimization algorithm
6: Run optimizer Ak(θk) within current search space
X to generate candidate search samples
7: for each candidate x do
8: Simulate circuit and evaluate performance
9: Update optimization history H
10: end for
11: Outer Loop: Search-Space Refinement
12: if feasible solution found then
13: break
14: end if
15: if convergence stagnates or constraints violated then
16: Analyze H to identify limiting variables
17: Update search space X by adjusting ranges or ac-
tivating new variables
18: end if
19: end while
20: return Best feasible solution x⋆
5

## Page 6

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
3.6. Evaluation and Feedback Mechanism
The evaluation and feedback mechanism in AUTOSIZER op-
erates at two levels, supporting both inner-loop optimization
control and outer-loop search-space refinement, as illus-
trated in Figure 4. Its role is to assess optimization progress,
diagnose bottlenecks, and provide structured feedback for
adaptive decision-making across iterations.
Search Space Evaluation and Feedback
Optimization Status: 5 iterations completed with 72 designs evaluated. Meth-
ods used: LHS (25 designs), BO (15 designs each in iterations 2 and 4), and
Simulated Annealing (18 designs). Best FOM of 0.0990 achieved in iteration
2 and remained unchanged through iterations 3 and 4, indicating stagnation.
Convergence Analysis: FOM progression: [0.095, 0.099, 0.099, 0.099].
Status: converging with stagnant value at 0.099. Reason: recent improvements
< 2%(0.00%). Best FOM unchanged for 3 consecutive iterations.
Search Space Issues (4 detected): - Wdiff: 10/10 top designs at lower bound-
ary (0.84 µm) → High severity, expand lower range - Wload: 10/10 top designs
at upper boundary (1.68 µm) → High severity, expand upper range - Wload:
10/10 top designs at lower boundary (1.68 µm) → High severity, expand lower
range - Stagnation: Best FOM unchanged for 3 iterations (0.0990) → Medium
severity
Variable Impact Analysis: Top design clustering: Wcasc range [1.26, 2.52]
with most common values (1.68: 4×, 2.1: 3×, 1.26: 2×). Wdiff range [0.84,
0.84] with only 0.84 appearing (10×). Wload range [1.68, 1.68] with only 1.68
appearing (10×). Wtail range [0.84, 2.52] with most common values (1.26: 3×,
2.1: 3×, 2.52: 2×).
Recommendations: Priority: HIGH. Should regenerate: YES. Actions: (1)
Consider stopping due to recent improvements < 2%(0.00%), (2) Expand
lower range for Wdiff based on boundary clustering and top design analysis,
(3) Expand both ranges for Wload due to dual boundary saturation, (4) Keep
current ranges for Wcasc and Wtail with adequate distribution, (5) Expand
search space or change strategy to escape stagnation.
Figure 4. Search space evaluation and feedback from AutoSizer
agent after inner loop finished with 4 optimization iterations.
Inner-loop evaluation assesses the effectiveness of the cur-
rent sizing optimization strategy and guides algorithm se-
lection and configuration. After each iteration, AutoSizer
summarizes optimization outcomes using statistics such as
FoM improvement, feasibility status, and parameter diver-
sity. Based on these signals, the framework dynamically
selects and configures optimization algorithms to balance
exploration and exploitation, and determines termination
when specifications are satisfied or convergence is detected.
Outer-loop evaluation assesses whether the current search
space is sufficient to meet design objectives and guides high-
level adaptation when necessary. It aggregates inner-loop
optimization outcomes across iterations, including feasi-
bility trends, FoM improvement trajectories, performance
distributions, and parameter concentration patterns. By an-
alyzing these signals, the outer loop identifies cases where
optimization progress saturates within the current parameter
bounds or where high-performing solutions consistently oc-
cur near search-space edges. In such cases, the search space
is refined by adjusting parameter ranges, re-prioritizing vari-
ables, or activating previously fixed variables. By decou-
pling evaluation at the optimization and search-space lev-
els, AutoSizer ensures that both numerical optimization
decisions and higher-level design-space refinement utilize
quantitative, simulation-based evidence.
4. Experiments
4.1. Experimental Setup
Benchmark and Evaluation. We evaluate on AMS-
SIZINGBENCH, an open SKY130-based benchmark suite
consisting of 24 analog and mixed-signal circuits, including
logic blocks, OTAs, oscillators, switched-capacitor circuits,
voltage references, and voltage regulators, with predefined
Easy, Medium, and Hard difficulty splits. Additional details
of the AMS-SIZINGBENCH benchmark, including circuit
descriptions, input signals, specifications, simulations, and
configuration formats, are provided in Appendix A. Each
circuit sizing task is a simulation-driven constrained op-
timization problem maximizing a figure-of-merit (FoM).
Circuit performance evaluation uses Ngspice (Vogt et al.,
2021) for electrical simulation with the SKY130 PDK.
Baselines. We compare AutoSizer against two categories
of circuit sizing methods. Traditional optimization algo-
rithms include Genetic Algorithm (GA) (Taherzadeh-Sani
et al., 2003), Bayesian Optimization (BO), and Trust Region
Bayesian Optimization (TuRBO) (Eriksson et al., 2019),
representing evolutionary, surrogate-based, and trust region
approaches respectively. LLM-based agent methods include
ADO-LLM (Yin et al., 2024), LEDRO (Kochar et al., 2025),
and EE-Sizer (Liu & Chitnis, 2025b). The remaining three
baselines are not included due to reproducibility consid-
erations. AmpAgent, LLM-USO, and LLM-ACD are not
open-source, and the available descriptions in their respec-
tive papers lack sufficient detail for reliable reproduction.
Implementation Details. AutoSizer is powered by
Gemini-2.5-Flash with a temperature of 0.4, top-p =
0.85, top-k = 20, and a maximum of 8192 output tokens.
To isolate algorithmic contributions from LLM backend
differences, we re-implemented LEDRO, ADO-LLM, and
EE-Sizer using the same Gemini-2.5-Flash configu-
ration and a unified 300-sample budget as AutoSizer. We
maintain the core algorithmic logic of each method while
standardizing the LLM interface and sample accounting,
which enables fair comparison of optimization strategies
independent of LLM-specific capabilities. Traditional base-
lines include: (i) a genetic algorithm (GA) with population
size 20, crossover rate 0.8, mutation rate 0.1, tournament se-
lection, and elitism; (ii) Bayesian optimization (BO) using a
Gaussian Process with a Matérn kernel (ν = 2.5) and UCB
acquisition (β = 2.0), initialized with 10 random samples;
and (iii) TuRBO with adaptive trust regions initialized at 0.8
and scaled by 2 × /0.5× on success/failure. All methods
use a total budget of searching 300 samples. EE-Sizer runs
6

## Page 7

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
10 iterations with 30 LLM-generated candidates per itera-
tion. AutoSizer uses up to three outer-loop iterations with
100 samples per inner loop, dynamically allocating samples
across multiple optimization algorithms within each inner
loop.
4.2. Overall Performance on AMS-SIZINGBENCH
Table 2 reports average three trials experimental results on
the AMS-SIZINGBENCH across three difficulty levels. Per-
formance is evaluated using Figure of Merit (FOM), num-
ber of evaluations to best solution (Evals), runtime per trial
(Time), and success rate (SR), defined as the percentage of
trials that meet user specifications. On the Hard benchmark,
baseline LLM-based agents significantly outperform tradi-
tional optimization methods, achieving higher success rates
with substantially fewer search samples. As circuit complex-
ity increases, the design space becomes highly nonlinear and
constrained; in this regime, LLM-based agents can leverage
embedded circuit knowledge to guide the search more effi-
ciently, resulting in improved robustness and efficiency. In
contrast, traditional methods suffer from degraded perfor-
mance and lower success rates. For the Easy and Medium
benchmarks, baseline LLM-based agents exhibit compa-
rable performance to traditional methods, indicating that
when the search space is relatively simple, conventional
optimization techniques remain competitive. Among the
LLM-based baselines, ADO-LLM performs worse due to
the absence of a self-reflection mechanism, which prevents it
from guaranteeing that the LLM-generated search directions
remain valid, leading to suboptimal exploration. EE-Sizer
relies solely on the LLM without an explicit optimization
algorithm; consequently, it requires more LLM interactions
per iteration and explores a larger search space, resulting
in increased runtime. LEDRO adopts a fixed optimization
configuration, which limits its flexibility and reduces effec-
tiveness, particularly for complex circuits. In contrast, our
method leverages adaptive optimization strategies guided by
LLM reasoning, enabling more efficient exploration of the
design space and consistently superior performance across
all difficulty levels, especially on complex circuit sizing
tasks.
4.3. Ablation Studies
We systematically disable individual modules, Circuit Un-
derstanding (CU), Initial Search Space Decision (SSD), Op-
timization Engine (OE), and Self-Reflection Loop (SRL),
and measure impact on 5 circuits (1 easy, 2 medium, 2 hard)
with 3 trials each. Table 3 shows averaged results, revealing
component importance and interdependencies.
Component ranking. Ablation results establish clear hier-
archy: SRL (-39.4% FOM) > OE (-23.9%) > CU (-19.2%).
This ranking demonstrates that iterative refinement (SRL) is
more critical than getting initial decisions right (CU), and
how you search (OE) matters more than where you search
(CU). Efficient exploration can partially compensate for sub-
optimal spaces, but optimal spaces cannot compensate for
inefficient exploration. SRL enables recovery from mis-
takes. Removing SRL causes 80% success rate drop (100%
→ 20%) and 39.4% FOM loss. Single-cycle optimization
is fragile because initial search space decisions may be in-
correct and cannot be corrected. SRL’s iterative refinement,
early cycles explore broadly, later cycles exploit refined
regions which is essential for robust convergence across
diverse circuits. OE improves sample efficiency. Without
OE, fixed Bayesian Optimization requires 37.5% more eval-
uations and 2× longer runtime, achieving only 50% success
rate. Adaptive algorithm selection matches algorithms to
optimization phases reducing wasted evaluations. CU and
SSD are interdependent. A counterintuitive result: w/o
CU degrades to 28.65 FOM (-19.2%), but w/o CU & SSD
improves to 33.43 FOM (-5.7%). This demonstrates that
search space reduction requires accurate circuit under-
standing. Without CU insights, SSD uses naming heuristics
that make incorrect assumptions, constraining search to sub-
optimal subspaces or fixing critical variables. The full space
(w/o CU & SSD) avoids these errors, outperforming mis-
guided reduction by 16.7% despite requiring 13.9% more
evaluations. Synergistic design. Components work syn-
ergistically to achieve best performance. Removing any
component breaks this synergy, with poorly informed re-
duction worse than no reduction, validating the necessity of
LLM-based circuit analysis.
4.4. Analysis and Discussion
To understand AutoSizer’s optimization dynamics, we vi-
sualize search space evolution, algorithm selection and
converge patterns within the inner loop, and average self-
reflection circle of ring oscillator, and search space reduction
across circuit difficulties, illustrated in Figure 5.
The search space evolution demonstrates strategic progres-
sive refinement. Figure 5a shows how variable ranges adapt
across three Self-Reflection Loop cycles starting from the
Original full search space (blue). Loop 1 (orange) aggres-
sively reduces the space by fixing several variables (Wnmos1,
Wpmos2, Wnmos2) at specific values while keeping others
active (Wpmos0, Wnmos0, Linv). Loop 2 (green) refines fur-
ther but strategically re-expands certain dimensions (Wpmos1,
Wnmos1) that proved important in Loop 1, demonstrating
adaptive recovery from over-aggressive reduction. Loop 3
(red) achieves the most focused search space, concentrat-
ing exploration on the critical subset of variables (Wnmos0,
Wpmos2, Wnmos2) while fixing others. The non-monotonic
refinement pattern where some dimensions shrink while oth-
ers expand, confirming that space adaptation is data-driven
based on observed performance rather than following pre-
7

## Page 8

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Table 2. Experimental results on AMS-SIZINGBENCH. FOM, Eval, Time (s), and SR% denote Figure of Merit, Evaluations to best, time
per trial, and % of trials meeting user specs, respectively. Bold indicates the best result and underline indicates the second best result.
Methods
Easy Medium Hard
FOM↑ Evals↓ Time (s)↓ SR%↑ FOM↑ Evals↓ Time (s)↓ SR%↑ FOM↑ Evals↓ Time (s)↓ SR%↑
Traditional Method
Genetic 4.0 ± 1.2 110.5 ± 42.9 515.6 ± 28.5 93.3% 33.6 ± 28.0 193.7 ± 47.9 1638.6 ± 18.4 73.3% 3.4 ± 1.3 165.6 ± 49.2 1689.2 ± 65.1 50.0%
BO 3.2 ± 1.5 84.0 ± 21.4 415.0 ± 55.8 80.0% 34.1 ± 39.5 153.6 ± 59.8 1492.5 ± 48.7 73.3% 3.5 ± 0.2 151.3 ± 51.2 1875.1 ± 73.5 50.0%
TuRBO 2.1 ± 1.5 20.6 ± 6.9 141.0 ± 15.6 39.8% 30.0 ± 47.6 75.7 ± 25.1 1450.2 ± 18.3 39.8% 2.7 ± 1.9 107.2 ± 19.1 2105.2 ± 42.7 25.0%
LLM-based Agent
ADO-LLM 3.3 ± 0.5 87.2 ± 11.5 527.2 ± 20.8 80.0% 31.9 ± 30.5 148.5 ± 23.5 1328.6 ± 25.8 80.0% 3.5 ± 0.7 162.3 ± 42.3 1478.5 ± 68.5 41.0%
LEDRO 4.1 ± 1.1 65.6 ± 13.4 385.6 ± 15.5 100% 34.6 ± 56.8 142.5 ± 32.5 1150.7 ± 25.4 80.0% 3.8 ± 0.6 140.8 ± 23.2 1350.8 ± 62.5 80.0%
EE-Sizer 4.0 ± 1.3 95.3 ± 21.8 756.2 ± 34.2 100% 33.3 ± 52.9 165.2 ± 37.4 1485.3 ± 37.5 80.0% 3.7 ± 0.9 170.8 ± 20.5 1872.6 ± 54.6 56.4%
AUTOSIZER 4.4 ± 1.6 38.9 ± 31.8 257.0 ± 16.4 100% 36.3 ± 58.8 42.2 ± 51.0 1359.9 ± 29.7 100% 4.5 ± 2.7 63.6 ± 62.5 1473.2 ± 39.3 100%
Wpmos0
Wnmos0
Wpmos1
Wnmos1
Wpmos2
Wnmos2
Linv
1
2
3
4
5
6
(a) Search Space Change across Outer Loop
Original
Loop 1
Loop 2
Loop 3
0 50 100 150 200
Evaluation Number
60
80
100
120
140
160
Best
FOM
Found
So
Far
LHS
BO
BO GA LHS LHS BO
BO BO SA BO
LHS
BO
Inner Loop 0 Inner Loop 1 Inner Loop 2
(b) Convergence with Algorithm Selection
Algorithms
LHS
BO
GA
SA
Easy Medium Hard
Difficulty
0.0
0.5
1.0
1.5
2.0
2.5
3.0
3.5
4.0
Self-Reflection
Loops
(mean
±
std)
1.8 ± 0.7
2.0 ± 0.6
2.5 ± 0.5
(c) Average Self-Reflection Cycles and Search Space Reduction Ratio
Self-Reflection Reduction
0.0
0.2
0.4
0.6
0.8
1.0
Search
Space
Reduction
Ratio
90.8%
83.7%
80.8%
Figure 5. AutoSizer’s optimization dynamics across three outer loops. (a) Progressive search space changes showing active variables
(multiple values) and fixed variables (single value) in each outer loop of the ring oscillator. (b) Convergence trajectory with adaptive
algorithm selection (LHS, BO, GA, SA) across inner loop of the ring oscillator. (c) Self-reflection loops and search space reduction ratios
across circuit difficulties.
Table 3. Ablation study of AutoSizer on 5 representive circuits (1
easy, 2 medium, and 2 hard). CU: Circuit Understanding. SSD:
Initial Search Space. OE: Optimization Engine (LLM with algo-
rithm pool). SRL: Self-Reflection Loop.
Methods
Components Metrics
CU SSD OE SRL FOM↑ Evals↓ Time↓ SR%↑
AutoSizer (Full) ✓ ✓ ✓ ✓ 35.47 127.5 925.8 100%
w/o CU ✗ ✓ ✓ ✓ 28.65 137.5 1206.4 73.4%
w/o CU & SSD ✗ ✗ ✓ ✓ 33.43 145.2 1053.8 86.8%
w/o OE ✓ ✓ ✗ ✓ 27.00 175.4 1881.4 50.0%
w/o SRL ✓ ✓ ✓ ✗ 25.32 150.9 956.8 50.0%
w/o CU & SSD & SRL ✗ ✗ ✓ ✗ 21.51 136.5 857.6 20.0%
determined monotonic reduction, enabling the system to
recover from suboptimal early decisions.
The convergence curves show consistent improvement
across refinement cycles. Figure 5b demonstrates that each
outer loop starts with higher FOM than the previous cy-
cle’s starting point and achieves progressively better final
performance. Within each cycle, the system saves optimiza-
tion history at every iteration, enabling the LLM to make
informed decisions about algorithm selection and configura-
tion based on observed convergence patterns. This pattern
where later cycles both start better and converge faster val-
idates that Self-Reflection Loop’s search space decisions
are correct: refined spaces eliminate unpromising regions,
enabling each cycle to build upon previous insights rather
than restarting from scratch.
Circuit difficulty affects iteration requirements but not re-
duction effectiveness. Figure 5c shows that harder circuits
require more self-reflection outer loop due to increased spec-
ification complexity and sparser feasible design regions.
These additional iterations are needed to progressively refine
the search space through variable unfixing and range expan-
sion when initial configurations prove insufficient. However,
search space reduction remains consistent across all diffi-
culty levels, demonstrating that LLM-based circuit analysis
effectively identifies high-impact variables regardless of
topology complexity.
4.5. LLM Strategy Decision Analysis
To validate that AutoSizer’s LLM-based strategy selection
produces non-trivial, adaptive decisions rather than follow-
ing a fixed pattern, we logged every strategy decision across
optimization cycles. Table 4 shows the complete decision
8

## Page 9

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
trace for the bandgap reference circuit. Three adaptive be-
haviors emerge that no fixed or random schedule can repli-
cate, summarized in Table 5.
Table 4. LLM strategy decision trace for the bandgap circuit across
three optimization cycles.
Cycle 0 (3 vars) Cycle 1 (5 vars) Cycle 2 (4 vars)
Iter 0 LHS n=15 BO n=35 BO n=25
Iter 1 Genetic n=15 BO n=50 BO n=35
Iter 2 BO n=15 LHS n=50 (re-explore) —
Iter 3 SA n=15 — —
Iter 4 LHS n=50 (re-explore) — —
#Evals 110 135 60
Vars 3 5 4
Best FOM 0.1085 0.1152 (+6.2%) 0.1209 (+11.5%)
Table 5. Adaptive behaviors observed in LLM-guided strategy
selection and why fixed/random schedules cannot replicate them.
Adaptive behavior What the LLM did Why fixed/random fails
Mid-cycle re-
exploration
Detected stagnation in Cycle 1,
switched back to LHS
Fixed never re-explores; ran-
dom re-explores arbitrarily
Circuit-specific selec-
tion
Bandgap: 4 methods incl. SA.
Switched-cap: skipped SA, al-
ternated BO↔LHS
One fixed schedule cannot fit
different FOM landscapes
Dimension-aware siz-
ing
n=15 for 3 vars → n=35–50
for 5 vars → n=25–35 for 4
vars
Fixed uses constant n regard-
less of space size
4.6. Search-Space Adaptation Analysis
To analyze what drives search-space transitions, we examine
the five categories of feedback signals that the LLM receives
at each regeneration cycle boundary (Table 6). These signals
are programmatically extracted from the inner optimization
loop with no manual curation.
Table 6. Feedback signals provided to the LLM at each outer-loop
regeneration cycle boundary.
Signal What the LLM sees Example trigger
Performance pro-
gression
FOM history across all iterations Stagnation triggers expand or
unfix
Convergence status Whether FOM improved in last 2
iterations, with stagnation flag
stagnant=True → vari-
able unfixing
Variable impact Which variables show variance
vs. convergence in top designs
70%+ converge to one value
→ fix it, unfix another
Boundary detection Whether top designs cluster at
range edges
20%+ at boundaries → expand
ranges
Top design details Full parameter values and met-
rics of best N designs
Reveals true performance
drivers
Based on these signals, the LLM selects one of six actions:
continue current, expand ranges, narrow ranges, unfix vari-
ables, change focus, or converged. Table 7 reports the fre-
quency of each action, its average FOM impact, and the
number of iterations before the action is triggered, stratified
by circuit difficulty.
The expand ranges or unfix variables action grows from
Table 7. Outer-loop action frequency, FOM impact, and timing
across circuit difficulty levels.
Diff. Action Freq. Avg FOM ∆ Iters before (mean±std)
Easy
expand / unfix 36.4% +1.5% 104±41
narrow ranges 6.0% +0.5% 164±32
change focus 7.1% +0.3% 154±63
continue current 50.5% +0.0% 125±64
Med.
expand / unfix 52.4% +2.5% 104±80
narrow ranges 20.7% +1.3% 118±21
change focus 16.3% +0.3% 155±36
continue current 10.6% +0.1% 191±80
Hard
expand / unfix 57.1% +16.4% 105±40
narrow ranges 10.0% −3.8% 180±48
change focus 20.0% +2.9% 183±52
continue current 12.9% +1.6% 197±35
36.4% on easy circuits to 57.1% on hard ones, with FOM
impact rising from +1.5% to +16.4%, while continue current
dominates easy circuits (50.5%), confirming the inner loop
alone suffices for low-dimensional problems. On harder
circuits, the LLM increasingly reshapes the search space.
The “Iters before” column reveals a principled ordering: ex-
pansion occurs early, while narrowing and refocusing occur
later, reflecting an exploration-first strategy that emerges
from optimization feedback rather than hard-coded rules.
5. Conclusion
In this work, we propose AUTOSIZER, a reflective meta-
optimization framework based on LLMs-based agents for
automatic analog and mixed-signal circuit sizing. Addition-
ally, we introduce AMS-SIZINGBENCH, a comprehensive
open-source benchmark for evaluating LLM-based agents
on analog and mixed-signal circuit sizing that includes
SKY130-based circuit netlists, a well-defined set of sizing
variables, and a complete simulation workflow for perfor-
mance evaluation. Comprehensive experiments on the pro-
posed AMS-SIZINGBENCH demonstrate that AUTOSIZER
consistently outperforms prior methods in both success rate
and figure of merit, particularly on hard, constraint-tight de-
signs. We hope that AUTOSIZER and AMS-SIZINGBENCH
provide a foundation for future research in reliable, scalable
LLM-driven AMS circuit design automation.
Acknowledgments
This work was supported by the U.S. Department of Energy
(DOE) Office of Science, Office of Advanced Scientific
Computing Research (ASCR), through an ASCR EXPRESS
award (DE-SCL0000083), and by the Laboratory Directed
Research and Development (LDRD) Program (LDRD 25-
041 and LDRD 25-006) at Brookhaven National Laboratory,
which is operated and managed for the DOE Office of Sci-
ence by Brookhaven Science Associates under Contract No.
9

## Page 10

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
DE-SC0012704. We also express our sincere gratitude to Dr.
Yuewei Lin and Dr. Shinjae Yoo at Brookhaven National
Laboratory for their valuable advice and feedback.
Impact Statement
This paper presents work whose goal is to advance the fields
of Electronic Design Automation and Machine Learning
through automated analog circuit design. We note that our
framework is designed to augment rather than replace hu-
man circuit designers, with interpretable LLM-generated
reasoning serving as an educational and verification tool.
Users should validate AutoSizer-generated designs through
standard verification flows before fabrication. The release
of AMS-SizingBench as an open-source benchmark pro-
motes reproducibility and accessibility in circuit optimiza-
tion research, supporting both academic study and practical
deployment of AI-assisted design automation.
References
Ahmadzadeh, M., Chen, K., and Gielen, G. Anaflow:
Agentic llm-based workflow for reasoning-driven explain-
able and sample-efficient analog circuit sizing. In 2025
IEEE/ACM International Conference On Computer Aided
Design (ICCAD), pp. 1–7. IEEE, 2025a.
Ahmadzadeh, M., Lappas, J., Wehn, N., and Gielen, G.
Anacraft: Duel-play probabilistic-model-based reinforce-
ment learning for sample-efficient pvt-robust analog cir-
cuit sizing optimization. IEEE Transactions on Computer-
Aided Design of Integrated Circuits and Systems, 2025b.
Baek, J., Jauhar, S. K., Cucerzan, S., and Hwang, S. J.
Researchagent: Iterative research idea generation over
scientific literature with large language models. In Pro-
ceedings of the 2025 Conference of the Nations of the
Americas Chapter of the Association for Computational
Linguistics: Human Language Technologies (Volume 1:
Long Papers), pp. 6709–6738, 2025.
Black, K., Brown, N., Driess, D., Esmail, A., Equi, M.,
Finn, C., Fusai, N., Groom, L., Hausman, K., Ichter,
B., et al. π0: A vision-language-action flow model for
general robot control. arXiv preprint arXiv:2410.24164,
2024.
Brohan, A., Chebotar, Y., Finn, C., Hausman, K., Herzog,
A., Ho, D., Ibarz, J., Irpan, A., Jang, E., Julian, R., et al.
Do as i can, not as i say: Grounding language in robotic
affordances. In Conference on robot learning, pp. 287–
318. PMLR, 2023.
Chen, W., Su, Y., Zuo, J., Yang, C., Yuan, C., Chan, C.-M.,
Yu, H., Lu, Y., Hung, Y.-H., Qian, C., et al. Agentverse:
Facilitating multi-agent collaboration and exploring emer-
gent behaviors. In ICLR, 2024.
Cohen, M. W., Aga, M., and Weinberg, T. Genetic algorithm
software system for analog circuit design. Procedia CIRP,
36:17–22, 2015.
Eriksson, D., Pearce, M., Gardner, J., Turner, R. D., and
Poloczek, M. Scalable global optimization via local
bayesian optimization. Advances in neural information
processing systems, 32, 2019.
Guo, S., Deng, C., Wen, Y., Chen, H., Chang, Y., and Wang,
J. DS-agent: Automated data science by empowering
large language models with case-based reasoning. In
Proceedings of the 41st International Conference on Ma-
chine Learning, volume 235 of Proceedings of Machine
Learning Research, pp. 16813–16848. PMLR, 2024.
Hong, S., Lin, Y., Liu, B., Liu, B., Wu, B., Zhang, C., Li, D.,
Chen, J., Zhang, J., Wang, J., et al. Data interpreter: An
llm agent for data science. In Findings of the Association
for Computational Linguistics: ACL 2025, pp. 19796–
19821, 2025.
Jimenez, C. E., Yang, J., Wettig, A., Yao, S., Pei, K., Press,
O., and Narasimhan, K. Swe-bench: Can language mod-
els resolve real-world github issues? In International
Conference on Learning Representations, volume 2024,
pp. 54107–54157, 2024.
Kochar, D. V., Wang, H., Chandrakasan, A. P., and Zhang,
X. Ledro: Llm-enhanced design space reduction and opti-
mization for analog circuits. In 2025 IEEE International
Conference on LLM-Aided Design (ICLAD), pp. 141–148.
IEEE, 2025.
Lai, Y., Poddar, S., Lee, S., Chen, G., Hu, M., Yu, B., Luo, P.,
and Pan, D. Z. Analogcoder-pro: Unifying analog circuit
generation and optimization via multi-modal llms. IEEE
Transactions on Computer-Aided Design of Integrated
Circuits and Systems, 2026.
Liu, B., Zhao, D., Reynaert, P., and Gielen, G. G. Gas-
pad: A general and efficient mm-wave integrated circuit
synthesis method based on surrogate model assisted evo-
lutionary algorithm. IEEE Transactions on Computer-
Aided Design of Integrated Circuits and Systems, 33(2):
169–182, 2014.
Liu, C. and Chitnis, D. Eeschematic: Multimodal-llm based
ai agent for schematic generation of analog circuit. arXiv
preprint arXiv:2510.17002, 2025a.
Liu, C. and Chitnis, D. Eesizer: Llm-based ai agent for siz-
ing of analog and mixed signal circuit. IEEE Transactions
on Circuits and Systems I: Regular Papers, 2025b.
10

## Page 11

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Liu, C., Chen, W., Peng, A., Du, Y., Du, L., and Yang, J. Am-
pagent: An llm-based multi-agent system for multi-stage
amplifier schematic design from literature for process and
performance porting. arXiv preprint arXiv:2409.14739,
2024.
Lyu, W., Yang, F., Yan, C., Zhou, D., and Zeng, X. Batch
bayesian optimization via multi-objective acquisition en-
semble for automated analog circuit design. In Interna-
tional conference on machine learning, pp. 3306–3314.
PMLR, 2018.
Pessoa, T., Lourenço, N., Martins, R., Póvoa, R., and Horta,
N. Enhanced analog and rf ic sizing methodology using
pca and nsga-ii optimization kernel. In 2018 Design,
Automation & Test in Europe Conference & Exhibition
(DATE), pp. 660–665. IEEE, 2018.
Ren, X., Zang, Q., and Guo, Z. Simugen: Multi-modal
agentic framework for constructing block diagram-based
simulation models. arXiv preprint arXiv:2506.15695,
2025.
Settaluri, K., Haj-Ali, A., Huang, Q., Hakhamaneshi, K.,
and Nikolic, B. Autockt: Deep reinforcement learning of
analog circuit designs. arXiv preprint arXiv:2001.01808,
2020.
Shahriari, B., Swersky, K., Wang, Z., Adams, R. P., and
De Freitas, N. Taking the human out of the loop: A review
of bayesian optimization. Proceedings of the IEEE, 104
(1):148–175, 2015.
Somayaji, N. K. and Li, P. Llm-uso: Large language model-
based universal sizing optimizer. IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Sys-
tems, 2025.
Taherzadeh-Sani, M., Lotfi, R., Zare-Hoseini, H., and
Shoaei, O. Design optimization of analog integrated
circuits using simulation-based genetic algorithm. In Sig-
nals, Circuits and Systems, 2003. SCS 2003. International
Symposium on, volume 1, pp. 73–76. IEEE, 2003.
Touloupas, K., Chouridis, N., and Sotiriadis, P. P. Local
bayesian optimization for analog circuit sizing. In 2021
58th ACM/IEEE design automation conference (DAC),
pp. 1237–1242. IEEE, 2021.
Vogt, H., Atkinson, G., Nenzi, P., and Warning, D. Ngspice
user’s manual version 34 (ngspice release version), 2021.
Wang, H., Yang, J., Lee, H.-S., and Han, S. Learning to
design circuits. arXiv preprint arXiv:1812.02734, 2018.
Wang, H., Wang, K., Yang, J., Shen, L., Sun, N., Lee, H.-S.,
and Han, S. Gcn-rl circuit designer: Transferable transis-
tor sizing with graph neural networks and reinforcement
learning. In 2020 57th ACM/IEEE Design Automation
Conference (DAC), pp. 1–6. IEEE, 2020.
Wolfe, G. and Vemuri, R. Extraction and use of neural
network models in automated synthesis of operational
amplifiers. IEEE Transactions on Computer-Aided De-
sign of Integrated Circuits and Systems, 22(2):198–212,
2003.
Xu, S., Zhi, H., Li, J., and Shan, W. Llmacd: An llm-based
analog circuit designer driven by behavior parameters.
In 2025 International Symposium of Electronics Design
Automation (ISEDA), pp. 157–162. IEEE, 2025.
Yamada, Y., Lange, R. T., Lu, C., Hu, S., Lu, C., Foerster, J.,
Clune, J., and Ha, D. The ai scientist-v2: Workshop-level
automated scientific discovery via agentic tree search.
arXiv preprint arXiv:2504.08066, 2025.
Yang, K.-E., Tsai, C.-Y., Shen, H.-H., Chiang, C.-F., Tsai,
F.-M., Wang, C.-A., Ting, Y., Yeh, C.-S., and Lai, C.-T.
Trust-region method with deep reinforcement learning in
analog design space exploration. In 2021 58th ACM/IEEE
design automation conference (DAC), pp. 1225–1230.
IEEE, 2021.
Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan,
K. R., and Cao, Y. React: Synergizing reasoning and
acting in language models. In The eleventh international
conference on learning representations, 2022.
Yin, Y., Wang, Y., Xu, B., and Li, P. Ado-llm: Analog
design bayesian optimization with in-context learning
of large language models. In Proceedings of the 43rd
IEEE/ACM International Conference on Computer-Aided
Design, pp. 1–9, 2024.
Yu, X., Yang, Y., Liu, Q., Du, Y., McSweeney, S., and Lin, Y.
Gencellagent: Generalizable, training-free cellular image
segmentation via large language model agents. arXiv
preprint arXiv:2510.13896, 2025.
Zhang, H., Sun, S., Lin, Y., Wang, R., and Bian, J. Analogx-
pert: Automating analog topology synthesis by incorpo-
rating circuit design expertise into large language models.
In 2025 International Symposium of Electronics Design
Automation (ISEDA), pp. 772–777. IEEE, 2025.
Zhang, J., Bao, J., Huang, Z., Zeng, X., and Lu, Y. Auto-
mated design of complex analog circuits with multiagent
based reinforcement learning. In 2023 60th ACM/IEEE
Design Automation Conference (DAC), pp. 1–6. IEEE,
2023.
11

## Page 12

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
A. Details of AMS-SIZINGBENCH
A.1. AMS-SIZINGBENCH Benchmark Circuit Summary
Table 8. AMS-SIZINGBENCH Benchmark Circuit Summary
Circuit Type Diff. #T Description Circuit Type Diff. #T Description
inverter Logic Easy 2 Basic CMOS logic gate that inverts
the input signal.
five trans ota Amp Med. 6 Compact operational transconduc-
tance amplifier with minimal transis-
tor count.
buffer Logic Easy 4 Cascaded inverter structure for restor-
ing logic levels.
vco Osc. Med. 10 Voltage-controlled oscillator with fre-
quency tunable by control voltage.
nand gate Logic Easy 4 CMOS logic gate implementing the
NAND Boolean function.
telescopic ota Amp Med. 10 High-gain telescopic cascode OTA us-
ing stacked devices.
resistive load amp Amp Easy 1 Single-transistor amplifier using a re-
sistive load.
current mirror ota Amp Med. 10 OTA based on current-mirror biasing
for gain and matching.
diode load amp Amp Easy 2 Amplifier employing a diode-
connected load.
folded cascode ota Amp Med. 10 Folded cascode OTA enabling wide
input common-mode range.
ring oscillator (3-stage) Osc. Med. 6 Three-stage inverter loop generating
oscillation.
ldo Power Hard 11 Low-dropout voltage regulator oper-
ating under limited headroom.
bandgap Ref. Hard 12 Temperature-stable voltage reference
circuit.
folded cascode ota low pass filter Filter Hard 10 Low-pass active filter implemented
using a Sallen–Key topology with a
folded-cascode OTA.
switched capacitor SC Hard 10 Switched-capacitor circuit with sam-
pling network and OTA.
folded cascode ota high pass filter Filter Hard 10 High-pass active filter implemented
using a Sallen–Key topology with a
folded-cascode OTA.
xor gate Logic Hard 14 CMOS logic gate implementing the
XOR operation.
folded cascode ota band pass filter Filter Hard 10 Band-pass OTA-based active filter im-
plemented using a Sallen–Key topol-
ogy with a folded-cascode OTA.
AZC Amp Hard 25 Active-Zero Compensation technique
for offset and low-frequency error re-
duction.
fdgb Amp Hard 26 Fully Differential Gain-Boosting Am-
plifier architecture.
nmcnr Amp Hard 24 Nested Miller Compensation with
Nulling Resistor for stability enhance-
ment.
smc Amp Hard 24 Simple Miller Compensation three-
stage amplifier.
dfcfc Amp Hard 26 Damping Factor Control Frequency
Compensation for multi-stage ampli-
fiers.
iac Amp Hard 35 Indirect Amplifier Compensation
technique for high-speed stability.
A.2. AMS-SIZINGBENCH Example
pdk_lib_path: ".../skywater-pdk-libs-sky130_fd_pr/combined_models/sky130.lib.spice"
align_pdk_path: ".../ALIGN-pdk-sky130/SKY130_PDK/"
pex_script_path: ".../ALIGN-public/test_magic_pex.py"
results_dir: "./telescopic_ota_test_folder"
user_specs: "Primary goal: Maximize DC gain and UGBW while minimizing power consumption
for a telescopic OTA with 1pF load."
user_specs_metric: "fom > 0.100 AND dc_gain_db > 55 AND ugbw > 10 AND power_dc < 50"
params:
L: 0.15
vdd: 1.8
vcm: 0.7
cload: 1e-12
ibias: 10e-6
vbiasp1: 1.15
variable:
W_tail_base: null
W_diff_base: null
W_casc_base: null
W_load_base: null
W_values: [0.84, 1.05, 1.26, 1.47, 1.68, 1.89, 2.10, 2.31, 2.52]
12

## Page 13

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
width_scales:
W_tail: [W_tail_base, 2]
W_diff: [W_diff_base, 4]
W_casc: [W_casc_base, 2]
W_load: [W_load_base, 2]
subckt_name: TELESCOPIC_OTA
subckt_pins:
- VBIASN
- VBIASNC
- VBIASP1
- VBIASP2
- VINN
- VINP
- VOUTN
- VOUTP
- VDD
- "0"
testbench_signals:
VBIASN: VBIASN
VBIASNC: VBIASNC
VBIASP1: VBIASP1
VBIASP2: VBIASP2
VINN: VINN
VINP: VINP
VOUTN: VOUTN
VOUTP: VOUTP
VDD: VDD
"0": "0"
metrics:
- dc_gain_db
- ugbw
- power_dc
- fom
base_metrics:
gain_db:
name: "Gain"
unit: "dB"
format: ".1f"
degradation_key: "gain_percent"
power_uw:
name: "Power"
unit: "µW"
format: ".1f"
degradation_key: "power_percent"
ugbw_mhz:
name: "UGBW"
unit: "MHz"
format: ".1f"
degradation_key: "ugbw_percent"
fom:
name: "FOM"
unit: ""
format: ".3f"
degradation_key: "fom_percent"
ota_subckt_template: |
.subckt TELESCOPIC_OTA VBIASN VBIASNC VBIASP1 VBIASP2 VINN VINP VOUTN VOUTP VDD 0
* Tail current sources
13

## Page 14

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
xm1 ID VBIASN 0 0 sky130_fd_pr__nfet_01v8 w={W_tail} l={L}
xm2 NET10 VBIASN 0 0 sky130_fd_pr__nfet_01v8 w={W_tail} l={L}
* Differential pair
xm3 NET8 VINP NET10 0 sky130_fd_pr__nfet_01v8 w={W_diff} l={L}
xm4 NET014 VINN NET10 0 sky130_fd_pr__nfet_01v8 w={W_diff} l={L}
* NMOS cascode (use VBIASNC here)
xm5 VOUTN VBIASNC NET8 0 sky130_fd_pr__nfet_01v8 w={W_casc} l={L}
xm6 VOUTP VBIASNC NET014 0 sky130_fd_pr__nfet_01v8 w={W_casc} l={L}
* PMOS cascode
xm7 VOUTN VBIASP1 NET06 VDD sky130_fd_pr__pfet_01v8 w={W_casc} l={L}
xm8 VOUTP VBIASP1 NET012 VDD sky130_fd_pr__pfet_01v8 w={W_casc} l={L}
* PMOS current source
xm9 NET06 VBIASP2 VDD VDD sky130_fd_pr__pfet_01v8 w={W_load} l={L}
xm10 NET012 VBIASP2 VDD VDD sky130_fd_pr__pfet_01v8 w={W_load} l={L}
.ends TELESCOPIC_OTA
testbench_template: |
* Telescopic OTA Simulation
.lib {pdk_lib_path} tt
{ota_subckt}
* Power supply
VDD VDD 0 DC {vdd}
* Bias Generation
IBIAS_N VDD VBIASN DC {ibias}
XMN_BIAS VBIASN VBIASN 0 0 sky130_fd_pr__nfet_01v8 w={W_tail} l={L}
IBIAS_P VBIASP2 0 DC {ibias}
XMP_BIAS VBIASP2 VBIASP2 VDD VDD sky130_fd_pr__pfet_01v8 w={W_load} l={L}
VBIASP1 VBIASP1 0 DC={vbiasp1}
IBIAS_NC VDD VBIASNC DC {ibias}
XMN_BIASC VBIASNC VBIASNC 0 0 sky130_fd_pr__nfet_01v8 w={W_casc} l={L}
* Differential inputs
VINP VINP 0 DC {vcm} AC 0.5
VINN VINN 0 DC {vcm} AC -0.5
* OTA instance
XOTA {inst_pins} {subckt_name}
* Load capacitors
CLOADP VOUTP 0 {cload}
CLOADN VOUTN 0 {cload}
.op
.control
op
* Power
let vdd_current = abs(i(VDD))
let power_dc = vdd_current * {vdd}
echo "=== POWER ==="
print vdd_current power_dc
* AC analysis
ac dec 100 1 10g
let vout_diff = v(VOUTP) - v(VOUTN)
let gain_db = db(vout_diff)
let dc_gain_db = gain_db[0]
echo "=== DC GAIN ==="
print dc_gain_db
* UGBW
if dc_gain_db > 0
meas ac ugbw WHEN gain_db=0 CROSS=1
end
quit
.endc
.end
14

## Page 15

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
A.3. Sizing Variables and Specification
This section provides the details of each circuit in the AMS-SIZINGBENCH input and output specifications, and the
simulation checklist used to validate each circuit block against design requirements.
Table 9. Circuit blocks: inputs, output specifications, and simulation checklist.
Circuit name Input signal(s) Output specification(s) Simulation part (what to run)
Inverter VDD = 1.8 V; VIN DC sweep
from 0 to 1.8 V with 10 mV step
and rail-to-rail pulse with 100 ps
rise/fall time; load capacitance
CL = 10 fF
Maximum DC voltage gain (dB); average
propagation delay at 50% threshold (tP LH ,
tP HL); dynamic power consumption.
DC sweep for transfer characteristic and gain extrac-
tion; transient analysis for delay and power; static
and dynamic power separation using supply current;
transistor width sweep for performance optimization
Buffer VDD = 1.8 V; VIN rail-to-
rail pulse from 0 to 1.8 V with
100 ps rise/fall time and 4 ns pe-
riod; DC sweep from 0 to 1.8 V
with 10 mV step; load capaci-
tance CL = 10 fF
Maximum DC voltage gain (dB); average
propagation delay at 50% threshold (tP LH ,
tP HL); dynamic power consumption.
DC sweep for overall buffer gain extraction; tran-
sient analysis for propagation delay and switching
behavior; average and static power estimation from
supply current; dynamic power computed as average
minus static power; transistor width sweep across
both stages for performance optimization
NAND (2-input) VDD = 1.8 V; two digital in-
puts A and B driven by piecewise-
linear waveforms to exercise
NAND switching cases, including
B rising and falling while A is
held high; rise/fall time 100 ps;
operating frequency 100 MHz;
load capacitance CL = 10 fF
Propagation delays tP HL and tP LH mea-
sured at 50% VDD threshold; average prop-
agation delay; static, dynamic, and total
power consumption; energy per transition;
output voltage levels VOH and VOL; noise
margins (NMH, NML)
Operating-point analysis for static current and static
power; transient simulation for delay, dynamic power,
and energy-per-transition extraction; delay measured
from input B to output Y at 50% VDD crossings;
output voltage extrema used to compute VOH , VOL,
and noise margins;
Resistive-Load Amplifier
(Common-Source + RL)
VDD = 1.8 V; input VIN bi-
ased at Vbias = 0.6 V with small-
signal excitation vac = 1 mV;
load RL = 50 kΩ and CL =
10 fF; AC sweep from 1 Hz to
10 GHz
Small-signal voltage gain (dB); −3 dB
bandwidth (MHz); DC power consumption
(µW); and output voltage swing (V)
Operating-point analysis to extract IDD and
VOUT,DC ; AC analysis (decade sweep) to obtain
gain response and determine −3 dB bandwidth;
power computed from IDDVDD; output swing es-
timated from the DC bias point.
Diode-Load Amplifier
(Common-Source +
Diode-Connected Load)
VDD = 1.8 V; input VIN biased
at 0.6 V with small-signal AC ex-
citation of 1 mV; load capaci-
tance CL = 10 fF; AC frequency
sweep from 1 Hz to 10 GHz
Small-signal voltage gain (dB); −3 dB
bandwidth (MHz); DC power consumption
(µW); output voltage swing (V)
Operating-point analysis to extract supply current
and DC output voltage; AC analysis to obtain gain
response and determine −3 dB bandwidth; power
computed from IDDVDD; output swing estimated
from the DC operating point; parametric sweep of
input and load transistor dimensions
3-stage ring oscillator VDD = 1.8 V; three cascaded
CMOS inverters connected in a
ring; initial condition applied to
one node for startup; load capaci-
tance CL = 10 fF; transient sim-
ulation
Oscillation frequency (MHz); oscillation pe-
riod (ns); average power consumption (µW);
delay per inverter stage (ps)
Transient simulation to capture steady-state oscilla-
tion; period measured from consecutive rising zero-
crossings at 50% VDD; oscillation frequency com-
puted from period; average power calculated from
mean supply current; delay per stage extracted as
period/(2N) for N = 3 stages
Bandgap reference Supply voltage VDD with nomi-
nal value 2.0 V; DC supply sweep
from 1.8 V to 3.3 V for line reg-
ulation measurement; AC small-
signal excitation at the supply for
PSRR analysis; temperature set to
27◦
C
Reference voltage VREF (V); DC power
consumption (µW); line regulation (%/V);
power supply rejection ratio (PSRR, dB)
Operating-point analysis to extract VREF and supply
current; DC sweep of VDD to evaluate line regu-
lation; AC analysis with supply perturbation to ex-
tract PSRR; power computed from supply current and
nominal VDD
Current-Mirror OTA VDD = 1.8 V; tail bias current
IBIAS = 20 µA; differential in-
put with common-mode voltage
VCM = 0.9 V and small-signal
excitation (+0.5 / −0.5 AC); load
capacitance CL = 1 pF
DC voltage gain (dB); unity-gain bandwidth
(UGBW, MHz); DC power consumption
(µW)
Operating-point analysis to extract supply current
and DC power; AC analysis to obtain differential
gain response; DC gain extracted at low frequency;
unity-gain bandwidth measured at 0 dB gain crossing;
Five-transistor OTA VDD = 1.8 V; tail bias current
IBIAS = 20 µA; differential in-
put with common-mode voltage
VCM = 0.9 V and small-signal
excitation (+0.5 / −0.5 AC); load
capacitance CL = 1 pF
DC voltage gain (dB); unity-gain bandwidth
(UGBW, MHz); DC power consumption
(µW).
Operating-point analysis to extract supply current
and DC power; AC analysis to obtain differential
gain response; DC gain extracted at low frequency;
unity-gain bandwidth measured at 0 dB gain crossing;
15

## Page 16

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Circuit name Input signal(s) Output specification(s) Simulation part (what to run)
Telescopic cascode OTA VDD = 1.8 V; tail bias cur-
rent IBIAS = 10 µA; differ-
ential input with common-mode
voltage VCM = 0.7 V and
small-signal excitation (+0.5 /
−0.5 AC); bias voltages VBIASN,
VBIASNC, VBIASP1 = 1.15 V,
and VBIASP2; differential load ca-
pacitance CL = 1 pF per output
DC differential voltage gain (dB); unity-gain
bandwidth (UGBW, MHz); DC power con-
sumption (µW)
Operating-point analysis to establish bias currents
and extract DC power; AC analysis of differential
output to obtain gain response; DC gain extracted
at low frequency; unity-gain bandwidth measured
at 0 dB differential gain crossing; parametric sweep
of tail, input, cascode, and load device widths to
evaluate gain–bandwidth–power trade-offs
Switched-Capacitor Cir-
cuit
VDD = 1.8 V; differential OTA
biased with IBIAS = 10 µA;
input signal VIN with DC bias
around mid-supply and AC ampli-
tude 0.25 V; two non-overlapping
clock phases at 1 MHz; sam-
pling, holding, and load capaci-
tors (Csamp, Chold, Cload)
Closed-loop gain (dB) and gain error; set-
tling time (ns); charge injection (mV); DC
power consumption (µW); total harmonic
distortion (THD, dB); output voltage swing
(V); unity-gain bandwidth (UGBW, MHz);
DC gain (dB); phase margin (deg)
Transient simulation of a switched-capacitor inte-
grator using two non-overlapping clock phases at
1 MHz; operating-point analysis to obtain DC bias
and initial output swing; closed-loop gain extracted
from steady-state output voltage after multiple clock
cycles; settling time measured using a 1% error crite-
rion relative to final value; charge injection quantified
from voltage disturbance at the sampling node dur-
ing clock transitions; average supply current used to
compute DC power consumption; FFT-based spec-
tral analysis of the output waveform to extract total
harmonic distortion (THD); separate open-loop OTA
AC analysis performed to extract DC gain, unity-gain
bandwidth (UGBW), and phase margin
Low-Dropout Regulator
(LDO)
Supply voltage VDD = 1.8 V
with DC sweep from 0.9VDD
to 1.1VDD; reference volt-
age VREF = 0.6 V; pro-
grammable load current swept
from 0.1ILOAD to 2ILOAD
(up to 20 mA); bias current
IBIAS = 10 µA
Regulated output voltage VOUT (V);
dropout voltage (mV); line regulation
(mV/V); load regulation (mV/mA); power
supply rejection ratio (PSRR, dB); DC
power consumption (µW); output voltage
variation over temperature (V)
Operating-point analysis to establish nominal bias
and output voltage; DC supply-voltage sweep from
0.9VDD to 1.1VDD to extract dropout voltage
and line regulation; DC load-current sweep from
0.1ILOAD to 2ILOAD to evaluate load regulation;
AC analysis with small-signal supply injection to ex-
tract PSRR; DC power consumption computed from
measured supply and load currents
Voltage-Controlled Oscil-
lator
Supply voltage VDD = 1.8 V;
control voltage VCTRL swept
from 0 to 1.8 V; five-stage CMOS
ring oscillator with inverter-based
delay cells; temperature set to
27◦
C
Oscillation frequency (Hz); DC power con-
sumption (µW); tuning range (%); fre-
quency at minimum and maximum control
voltage; VCO gain (MHz/V)
Transient simulation of a five-stage CMOS ring os-
cillator with voltage-dependent load capacitance to
induce frequency tuning; a single control-voltage
run used to extract steady-state oscillation frequency
and DC power consumption from time-domain wave-
forms; oscillation frequency measured from rising
zero-crossings after steady-state is reached; power
computed from the median supply current during
steady-state operation; extended characterization per-
formed by sweeping the control voltage across the
specified range and repeating transient simulations at
each point to extract frequency tuning behavior; tun-
ing range computed from minimum and maximum
oscillation frequencies; average VCO gain (MHz/V)
extracted from local frequency–voltage slopes; sta-
tistical filtering applied to remove high-power out-
liers before aggregating power metrics; overall per-
formance summarized using a composite figure of
merit (FOM)
XOR Gate (2-Input
CMOS XOR with Trans-
mission Gates)
VDD = 1.8 V; two digital in-
puts A and B driven by piecewise-
linear waveforms to exercise
XOR switching conditions, in-
cluding rising and falling transi-
tions of B while A is held high;
rise/fall time 100 ps; operating
frequency 100 MHz; load capaci-
tance CL = 10 fF
Propagation delays tP HL and tP LH (ps);
average propagation delay (ps); static, dy-
namic, and total power consumption (µW);
energy per transition (fJ); output voltage lev-
els VOH and VOL (V); noise margins (NMH,
NML)
Operating-point analysis to extract static supply cur-
rent and static power; transient simulation with rep-
resentative input transitions to measure propagation
delays at the 50% VDD threshold; average dynamic
power computed from mean supply current during
switching activity; energy per transition calculated
from dynamic power and switching frequency; output
voltage extrema used to extract VOH , VOL, and noise
margins
Active Zero Compen-
sated OTA (AZC)
VDD = 1.8 V; differential in-
put with VCM = 0.3 V; small-
signal AC excitation (+0.5/ −
0.5); bias current IBIAS = 1 µA;
load RL = 25 kΩ, CL = 15 nF
DC voltage gain (dB); unity-gain bandwidth
(UGBW); DC power consumption (µW).
Operating-point analysis to extract supply current and
DC power; AC analysis (1 Hz–10 GHz) with differ-
ential excitation to obtain gain response; DC gain
extracted from low-frequency AC magnitude; unity-
gain bandwidth measured at 0 dB gain crossing;
Damping-Factor-
Controlled Folded-
Cascode OTA (DFCFC)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/−0.5); bias
current IBIAS = 10 µA; load
RL = 25 kΩ, CL = 15 nF
DC voltage gain (dB); unity-gain bandwidth
(UGBW); DC power consumption (µW).
Operating-point analysis to establish bias currents
and compute DC power from supply current; AC
analysis (1 Hz–10 GHz) with differential excitation
to obtain gain response; DC gain extracted from low-
frequency AC magnitude; unity-gain bandwidth mea-
sured at 0 dB gain crossing;
16

## Page 17

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
Circuit name Input signal(s) Output specification(s) Simulation part (what to run)
Fully Differential Gain-
Boosted OTA (FDGB)
VDD = 1.8 V; fully differential
input with VCM = 0.9 V; small-
signal AC excitation (+0.5/ −
0.5); bias current IBIAS =
10 µA; differential load RL =
25 kΩ, CL = 15 nF per output
DC differential voltage gain (dB); unity-gain
bandwidth (UGBW); DC power consump-
tion (µW).
Operating-point analysis to establish bias currents
and compute DC power from supply current; AC anal-
ysis (1 Hz–10 GHz) with fully differential excitation
and differential output sensing (VOUT P − VOUT N );
DC gain extracted from low-frequency differential
AC magnitude; unity-gain bandwidth measured at
0 dB differential gain crossing.
Indirectly Compensated
OTA (IAC)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/−0.5); bias
current IBIAS = 47 µA; load
RL = 25 kΩ, CL = 15 nF
DC voltage gain (dB); unity-gain bandwidth
(UGBW); DC power consumption (µW).
Operating-point analysis to establish bias currents
and compute DC power from supply current; AC
analysis (1 Hz–10 GHz) with differential excitation
to obtain gain response; DC gain extracted from low-
frequency AC magnitude; unity-gain bandwidth mea-
sured at 0 dB gain crossing; compensation behavior
governed by indirect Miller network.
Nested-Miller OTA with
Nulling Resistor (NM-
CNR)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/−0.5); bias
current IBIAS = 19 µA; load
RL = 25 kΩ, CL = 15 nF
DC voltage gain (dB); unity-gain bandwidth
(UGBW); DC power consumption (µW).
Operating-point analysis to establish bias currents
and compute DC power from supply current; AC
analysis (1 Hz–10 GHz) with differential excitation
to obtain gain response; DC gain extracted from low-
frequency AC magnitude; unity-gain bandwidth mea-
sured at 0 dB gain crossing; nested Miller compen-
sation with nulling resistor used for pole–zero place-
ment.
Single-Miller-
Compensated OTA
(SMC)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/−0.5); bias
current IBIAS = 20 µA; load
RL = 25 kΩ, CL = 15 nF
DC voltage gain (dB); unity-gain bandwidth
(UGBW); DC power consumption (µW).
Operating-point analysis to establish bias currents
and compute DC power from supply current; AC
analysis (1 Hz–10 GHz) with differential excitation
to obtain gain response; DC gain extracted from low-
frequency AC magnitude; unity-gain bandwidth mea-
sured at 0 dB gain crossing; dominant-pole stabiliza-
tion achieved via single Miller compensation capaci-
tor; FOM computed from extracted gain, UGBW, and
power.
Folded-cascode OTA VDD = 1.8 V; tail current
ITAIL = 10 µA; differential
input with common-mode volt-
age VCM = 0.9 V and small-
signal excitation (+0.5 / −0.5
AC); bias voltages VBN = 0.7 V
and VBP = 1.0 V; load capaci-
tance CL = 1 pF
DC voltage gain (dB); unity-gain bandwidth
(UGBW, MHz); DC power consumption
(µW).
Operating-point analysis to establish bias currents
and extract DC power; AC analysis to obtain differ-
ential gain response; DC gain measured as maximum
low-frequency gain; unity-gain bandwidth extracted
at 0 dB gain crossing; parametric sweep of device
widths to optimize gain–bandwidth–power trade-offs
under a fixed 1 pF load.
Folded-Cascode OTA
with Integrated Low-Pass
Filter (LPF)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/ − 0.5); tail
current ITAIL = 10 µA; bias
voltages VBN = 0.6 V, VBP =
0.6 V; load capacitance CL =
1 pF
Low-pass cutoff frequency (fc); passband
gain (dB); roll-off rate (dB/dec); stopband at-
tenuation (dB); quality factor (Q); DC power
consumption (µW).
Operating-point analysis to establish bias currents
and DC operating conditions; AC analysis (1 Hz–
10 GHz) with differential excitation to extract fre-
quency response; cutoff frequency determined from
−3 dB point; passband gain measured at low fre-
quency; roll-off and stopband attenuation extracted
from AC magnitude slope; quality factor computed
from filter parameters; power calculated from supply
current.
Folded-Cascode OTA
with Integrated High-Pass
Filter (HPF)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/ − 0.5); tail
current ITAIL = 10 µA; bias
voltages VBN = 0.7 V, VBP =
1.0 V; load capacitance CL =
1 pF
High-pass cutoff frequency (fc); passband
gain (dB); roll-off rate (dB/dec); stopband at-
tenuation (dB); quality factor (Q); DC power
consumption (µW).
Operating-point analysis to establish bias currents
and DC operating conditions; AC analysis (1 Hz–
10 GHz) with differential excitation to extract fre-
quency response; cutoff frequency determined from
−3 dB point; passband gain measured at high fre-
quency; roll-off and stopband attenuation extracted
from AC magnitude slope; quality factor computed
from filter parameters; power calculated from supply
current.
Folded-Cascode OTA
with Integrated Band-
Pass Filter (BPF)
VDD = 1.8 V; differential input
with VCM = 0.9 V; small-signal
AC excitation (+0.5/ − 0.5); tail
current ITAIL = 10 µA; bias
voltages VBN = 0.7 V, VBP =
1.0 V; load capacitance CL =
1 pF
Center frequency (f0); bandwidth (Hz);
peak gain (dB); quality factor (Q); DC
power consumption (µW).
Operating-point analysis to establish bias currents
and DC operating conditions; AC analysis (1 Hz–
10 GHz) with differential excitation to extract band-
pass frequency response; center frequency identified
at peak gain; bandwidth measured between −3 dB
points; quality factor computed as Q = f0/BW;
peak gain extracted from AC magnitude response;
power calculated from supply current;
17

## Page 18

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
B. Prompts used of AUTOSIZER
B.1. Circuit Understanding Prompt
Circuit Understanding Prompt
You are an expert analog circuit designer. Your task is to understand the circuit
topology and the role of each component, with focus on the optimization variables.
**Circuit Name:** {subckt_name}
**Circuit Netlist:**
‘‘‘
{ota_subckt_template}
‘‘‘
**Fixed Design Parameters (DO NOT optimize):**
{params}
**Optimization Variables (WILL be optimized):**
{variables}
**Testbench:**
‘‘‘
{testbench_template}
‘‘‘
**Performance Metrics:**
The following metrics will be evaluated:
{metrics_list}
## Your Task - Part 1: Circuit Analysis
**IMPORTANT: Keep each main section to 3-5 sentences total. Focus on how the
OPTIMIZATION VARIABLES affect circuit performance, not the fixed parameters.**
### 1. Circuit Topology Overview (3-5 sentences total)
Identify the circuit type from the netlist, describe the overall architecture, and
explain the signal flow from input to output.
### 2. Optimization Variables Mapping (3-5 sentences total)
For each optimization variable, identify which transistors it controls (from the netlist
and scaling rules). Group the variables by the functional role of transistors they
control.
### 3. Optimization Variables Impact on Performance (3-5 sentences total for each
sub-section)
Impact on {metric_name}:
Explain how the optimization variables {{variables}} affect {metric_key}.
Focus on which variables have the strongest impact on {metric_key} and why.
### 4. Variable Interactions (3-5 sentences total)
Describe any interactions between optimization variables: which must be scaled together
for matching, which have conflicting effects, and which have synergistic effects.
### 5. Key Insights for Optimization (3-5 bullet points, 1 sentence each)
Provide 3-5 critical insights about optimizing this circuit, focusing on the
optimization variables.
## Output Requirements
You must provide your analysis in **valid JSON format only**.
Your response should contain ONLY the JSON object with no additional text before
or after.
18

## Page 19

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
### JSON Structure
‘‘‘json
{{
"circuit_topology_overview": "3-5 sentences identifying the circuit type, describing
the overall architecture, and explaining signal flow from input to output.",
"optimization_variables_mapping": "3-5 sentences for each optimization variable,
identifying which transistors it controls and grouping variables by functional role.",
"optimization_variables_impact": {impact_json_str},
"variable_interactions": "3-5 sentences describing interactions between optimization
variables: which must be scaled together for matching, which have conflicting effects,
and which have synergistic effects.",
"key_insights_for_optimization": [
"First critical insight about optimizing this circuit (1 sentence)",
"Second critical insight about optimizing this circuit (1 sentence)",
"Third critical insight about optimizing this circuit (1 sentence)",
"Fourth critical insight (optional, 1 sentence)",
"Fifth critical insight (optional, 1 sentence)"
]
}}
‘‘‘
**CRITICAL REQUIREMENTS:**
1. Your entire response MUST be valid JSON - no markdown, no explanations, no text
outside the JSON object
2. Do NOT wrap the JSON in code blocks or backticks
3. Each section should be 3-5 sentences focused on OPTIMIZATION VARIABLES
4. The "key_insights_for_optimization" array must contain 3-5 strings (bullet points
as array elements)
5. All strings must properly escape special characters (quotes, newlines, etc.)
Focus your analysis on the OPTIMIZATION VARIABLES and their impact on performance.
B.2. Initial Search Space Construction Prompt
Initial Search Space Construction Prompt
You are an expert analog circuit optimizer. Based on the circuit understanding analysis,
your task is to rank all optimization variables by their impact on the target metric
and determine the search space.
**Circuit Name:** {subckt_name}
**Optimization Target Metric:** {target_metric}
**Number of Variables to Actively Optimize:** {num_variables_to_optimize}
**Total Number of Variables:** {total_num_variables}
**Available Discrete Values:**
{variable_ranges}
**Scaling Rules (if applicable):**
{scaling_rules}
**Circuit Understanding Analysis:**
**Variable Impact on Metrics:**
{variable_impact_summary}
**Variable Interactions:**
{variable_interactions}
19

## Page 20

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
**Key Optimization Insights:**
{key_insights}
## Your Task - Part 2: Variable Prioritization and Search Space Configuration
Your task is to:
1. **Rank ALL optimization variables** (1 to {total_num_variables}) based on their
impact on {target_metric}
2. **Select the top {num_variables_to_optimize} variables** to actively optimize
3. **For the top {num_variables_to_optimize} variables**: Provide sparse search ranges
covering small to large values (3-7 values each, including extremes)
4. **For the remaining variables**: Determine appropriate fixed values
(will NOT be optimized)
**Important Considerations:**
- Prioritize variables that have the strongest impact on {target_metric}
- Consider matching requirements and circuit topology
- Balance between search space size and coverage
- Fixed values should enable good performance while allowing optimized variables to
have maximum impact
### JSON Structure
‘‘‘json
{{
"optimization_target": "copy target metric here",
"num_variables_to_optimize": {num_variables_to_optimize},
"variable_ranking": [
{{
"rank": <integer>,
"variable": "<variable_name>",
"impact_on_target": "<critical/high/medium/low>",
"reasoning": "<CONCISE explanation max 100 chars - no newlines>"
}}
],
"optimization_configuration": {{
"variables_to_optimize": {{
"<variable_name>": {{
"rank": <integer>,
"search_space": [<value1>, <value2>, <value3>, ...],
"num_choices": <integer>,
"range_reasoning": "<CONCISE max 100 chars - no newlines>",
"expected_behavior": "<CONCISE max 80 chars - no newlines>",
"sensitivity": "<high/medium/low>"
}}
}},
"variables_fixed": {{
"<variable_name>": {{
"rank": <integer>,
"fixed_value": <numeric_value>,
"fixed_reasoning": "<CONCISE max 100 chars - no newlines>",
"why_this_value": "<CONCISE max 80 chars - no newlines>",
"risk_if_suboptimal": "<low/medium/high>"
}}
}}
}},
"search_space_summary": {{
"original_full_space": <integer>,
20

## Page 21

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
"reduced_search_space": <integer>,
"reduction_factor": <number>,
"calculation": "<mathematical expression>",
"explanation": "<CONCISE max 100 chars - no newlines>"
}}
}}
B.3. Optimization Algorithm Orchestration Prompt
Optimization Algorithm Orchestration Prompt
ADVANCED SEARCH METHODS AVAILABLE
### Method Arsenal:
**1. ’lhs’ (Latin Hypercube Sampling)** - Pure Exploration
- **Use when**: Early exploration, design space poorly understood, <10 previous designs
- **Sample size**: Based on search space - larger space needs more samples (15-30%
of space, min 15)
- **Parameters**: ‘seed‘ (change each iteration for diversity)
**2. ’genetic’ (Genetic Algorithm)** - Evolutionary Exploration
- **Use when**: Want diverse solutions, need robust global search, 10-50 previous
designs
- **Sample size**: Based on search space - scales with complexity (10-20% of space,
20-50 samples typical, larger for big spaces)
- **Key Parameters**:
- ‘mutation_rate‘: 0.2 (normal) → 0.4+ (stuck)
- ‘crossover_rate‘: 0.8 (default)
- ‘tournament_size‘: 3
**3. ’bayesian’ (Gaussian Process Bayesian)** - Intelligent Exploitation
- **Use when**: 25+ previous results, need sample efficiency, discrete space
performance uncertain
- **Sample size**: Based on search space - more samples for larger spaces
(5-15% of space, 10-20 samples typical, can go higher for large spaces)
- **Key Parameters**:
- ‘acquisition_function‘: "EI" (balanced), "LCB" (explore), "UCB" (stuck),
"PI" (refine)
- ‘exploration_weight‘: 0.1-0.3 for EI/PI, 2.0-3.5 for LCB/UCB
**4. ’adaptive’ (Multi-Strategy Balance)** - Balanced
- **Use when**: Unsure whether to explore/exploit, want safe balanced approach
- **Sample size**: Based on search space - moderately scales with size
(10-20% of space, 15-25 samples typical, adjust for large spaces)
- **Key Parameters**:
- ‘explore_weight‘: 0.5 (default), 0.6+ (low diversity)
- ‘exploit_weight‘: 0.5 (default), 0.6+ (late stage)
- ‘random_weight‘: 0.2 (default), 0.3 (stuck)
**5. ’annealing’ (Simulated Annealing)** - Escape Local Optima
- **Use when**: Stuck in plateau, suspect local optimum, need to escape
- **Sample size**: Based on search space - needs more samples for larger spaces
(8-15% of space, 12-20 samples typical, scale up for big spaces)
- **Key Parameters**:
- ‘initial_temperature‘: 2.0 (default), 3.0 (desperate)
- ‘cooling_rate‘: 0.95
**6. ’multistart’ (Multi-Start Local Search)** - Find Alternatives
- **Use when**: Verify convergence, find diverse good designs, late stage
- **Sample size**: Based on search space - more starts for larger spaces
(8-15% of space, 12-25 samples typical, proportional to space complexity)
21

## Page 22

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
- **Key Parameters**:
- ‘n_starts‘: 5 (default), 7+ (want diversity or large space)
- ‘search_radius‘: 2
## STRATEGIC DECISION FRAMEWORK
**Diagnose Current Situation:**
- Exploring? → LHS, Genetic, Adaptive
- Exploiting? → Bayesian, Optuna
- Stuck? → Annealing, Multistart
- Converged? → Stop
**Key Factors to Consider:**
1. **Data availability**: <10 designs → LHS/Genetic; 15-25 → Bayesian 30+ → Any method
2. **Improvement trend**: >5% → keep exploring; 1-5% → balance; <1% → exploit or stop
3. **Plateau detection**: 2 iterations no improvement → switch method;
3+ → consider stopping
4. **Search space size**: ALL methods scale with space - larger spaces need more samples
**Parameter Tuning Rules:**
- Stuck/identical designs → increase exploration (higher mutation_rate,
exploration_weight, temperature)
- Low diversity (std < 0.01) → increase randomness
- Steady improvement → continue current approach
- Approaching convergence → increase exploitation
## STOPPING RULES
**STOP if:**
1. User specification is met (PRIMARY GOAL)
2. <2% improvement over 2 iterations with diverse methods
3. 3+ iteration plateau with method diversity
**Sample Size Guidelines:**
- **All methods scale with search space size**
- **Exploration methods** (LHS, Genetic, Adaptive): 15-30% of space
- **Exploitation methods** (Bayesian, Optuna, Annealing, Multistart): 5-15% of space
- **LLM Direct**: 5-15% of space (cap individual batches at 20-30 for token limits)
- **General rule**: Larger search space → more samples needed for effective coverage
- **Always**: Cap at remaining budget, consider iteration efficiency
## RESPONSE FORMAT
**CRITICAL - READ THIS CAREFULLY**
### TEMPLATE 1: If you want to CONTINUE optimizing (use "search")
‘‘‘json
{
"action": "search",
"method": "PUT_ALGORITHM_NAME_HERE",
"n_samples": PUT_NUMBER_HERE,
"parameters": {
// Include only relevant parameters for the chosen method
// For example, if using genetic:
"mutation_rate": 0.2,
"crossover_rate": 0.8,
"tournament_size": 3
// See method descriptions above for parameter details
},
"reasoning": "YOUR_REASONING_HERE",
"confidence": "high or medium or low",
"expected_improvement": "YOUR_EXPECTED_IMPROVEMENT",
"convergence_assessment": "YOUR_CONVERGENCE_ANALYSIS"
}
22

## Page 23

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
### TEMPLATE 2: If you want to STOP optimizing (use "stop")
‘‘‘json
{
"action": "stop",
"reasoning": "WHY_STOPPING",
"confidence": "high or medium or low",
"expected_improvement": "N/A - converged",
"convergence_assessment": "CONVERGENCE_EVIDENCE"
}
B.4. Search Space Regeneration Prompt
Search Space Regeneration Prompt
"""You are an expert optimization advisor. RE-EVALUATE and REGENERATE the search space
based on optimization results.
## CONTEXT
After {iterations_completed} iterations with {total_designs} designs evaluated,
re-analyze the circuit and adapt the search strategy.
## CIRCUIT NETLIST (Re-analyze based on actual results)
‘‘‘
{netlist}
‘‘‘
**Task:** Re-analyze this circuit considering the optimization results below.
Your previous understanding may have been incorrect.
## ORIGINAL SEARCH SPACE (Your Constraints)
### Available Optimization Variables
**You can ONLY optimize these variables (from YAML config):**
{available_variables}
### Fixed Parameters (CANNOT be changed to variables)
**CRITICAL: These parameters are PERMANENTLY FIXED and CANNOT be optimized
or unfixed:**
{fixed_parameters}
**WARNING: DO NOT use any variable names from the "Fixed Parameters" list above in
your optimization_configuration. They are not available for optimization under
any circumstances.**
### Value Ranges (Your choices MUST be subsets of these)
{value_ranges}
**Original Full Search Space:** {original_search_space_size} combinations
## CURRENT CONFIGURATION
### Search Space Comparison
{search_space_comparison}
### Optimized Variables
{optimized_vars_section}
### Fixed Variables
{fixed_vars_section}
**Current Search Space:** {current_search_space} combinations
23

## Page 24

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
## OPTIMIZATION RESULTS
### Performance Progression
{progression_section}
### Convergence Status
- Status: {convergence_status}
- Assessment: {convergence_reason}
- Best FOM: {best_fom}
- Stagnant: {stagnant}
### Variable Impact
{impact_section}
### Issues Detected
{issues_section}
### Top Designs
{top_designs_section}
## YOUR TASK
Decide on action based on the results:
- ‘continue_current‘: Keep current configuration (if performing well and not stagnant)
- ‘expand_ranges‘: Expand variable ranges (preferred if ANY designs at boundaries)
- ‘narrow_ranges‘: Narrow to promising regions (only if very clear winner region)
- ‘unfix_variables‘: Unfix some fixed variables (preferred if stagnation or suboptimal)
- ‘change_focus‘: Swap optimized/fixed variables (if optimized var has low impact)
- ‘converged‘: Optimization complete (only if target met or exhausted search)
## DECISION GUIDELINES
**Expand ranges (PREFERRED ACTION):**
- 20%+ of top designs at boundaries (lowered threshold)
- Current range seems restrictive
- FOM improvements still occurring
- Action: Add 2-3 adjacent values from original ranges on each boundary
- Bias: Favor expansion over narrowing
**Unfix variables (HIGHLY ENCOURAGED):**
- Any fixed variable could impact performance
- Stagnation detected (no improvement for 2+ iterations)
- Current optimized variables explored adequately
- Suspect interactions between fixed and optimized variables
- Action: Unfix 1-2 most promising fixed variables with 5-7 values
- Priority: Unfix before declaring convergence
**Narrow ranges (USE SPARINGLY):**
- Top 80%+ designs cluster tightly in middle 30% of range
- Very clear single winner region (not just top design)
- Action: Focus on 4-6 best values only
- Warning: Only use if extremely confident
**Change focus (ENCOURAGED FOR EXPLORATION):**
- Optimized variable shows minimal variance in top designs (>70% same value)
- All top designs converged to 1-2 values for a variable
- Action: Fix the converged variable, unfix a new one
- Enables exploring new dimensions
**Continue current:**
- Making steady progress (improvement in last iteration)
- Haven’t explored current space adequately (< 40% of combinations)
- No clear issues with current configuration
- Use temporarily, but bias toward taking action
24

## Page 25

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
**Converged (LAST RESORT):**
- Best FOM exceeds or meets target specification
- Improvements < 1% for 4+ consecutive iterations
- All promising variables explored with expanded ranges
- Fixed variables already unfixed and tested
- Warning: Only declare convergence after aggressive exploration
## DECISION PRIORITY ORDER
1. **Unfix variables** - if any fixed variables remain and performance stagnating
2. **Expand ranges** - if any boundary clustering or improvements still occurring
3. **Change focus** - if optimized variable converged but performance suboptimal
4. **Continue current** - if making progress and space not fully explored
5. **Narrow ranges** - only if overwhelming evidence of tight optimal region
6. **Converged** - only after exhausting all exploration options
## CRITICAL CONSTRAINTS
1. **ONLY use variables from "Available Optimization Variables" section above**
2. **Value ranges MUST be subsets of the ranges shown in "Value Ranges" section**
3. **DO NOT create new variables like ’L’ if they are fixed parameters**
4. **DO NOT use values outside the allowed ranges**
6. **Each variable: 5-7 discrete values (increased from 3-7 for more exploration)**
7. **Variables in "Fixed Parameters" section CAN be unfixed and optimized**
8. **Bias: When uncertain, choose expansion or unfixing over narrowing or convergence**
## EXPLORATION PHILOSOPHY
- **Default to action**: Prefer exploring (expand/unfix) over staying static
- **Maximize search space**: Unfix variables aggressively before claiming convergence
- **Trust the data**: Only narrow when >80% of evidence supports it
- **Avoid premature convergence**: If in doubt, explore more
## OUTPUT (JSON ONLY)
{{{{
"optimization_target": "{target_metric}",
"regeneration_reasoning": "<why regenerating>",
"action_taken": "<action>",
"changes_from_previous": "<summary of changes>",
"variable_ranking": [
{{{{
"rank": <1-n>,
"variable": "<name>",
"impact_on_target": "<critical|high|medium|low>",
"reasoning": "<explanation>"
}}}}
],
"optimization_configuration": {{{{
"variables_to_optimize": {{{{
"<variable_name>": {{{{
"rank": <rank>,
"search_space": [<values>],
"num_choices": <number>,
"range_reasoning": "<why this range>",
"expected_behavior": "<how target changes>",
"sensitivity": "<high|medium|low>",
"change_from_previous": "<what changed>"
}}}}
}}}},
"variables_fixed": {{{{
"<variable_name>": {{{{
"rank": <rank>,
"fixed_value": <value>,
25

## Page 26

AutoSizer: Automatic Sizing of Analog and Mixed-Signal Circuits via Large Language Model (LLM) Agents
"fixed_reasoning": "<why fixed>",
"why_this_value": "<why this value>",
"risk_if_suboptimal": "<risk>",
"change_from_previous": "<what changed>"
}}}}
}}}}
}}}},
"search_space_summary": {{{{
"original_full_space": <total>,
"reduced_search_space": <total>,
"reduction_factor": "<factor>",
"change_factor": "<expansion or reduction>",
"calculation": "<calculation>",
"explanation": "<explanation>"
}}}},
"expected_improvement": "<expected gain>",
"confidence": "<high|medium|low>"
}}}}
Return ONLY JSON based on data.
"""
26

