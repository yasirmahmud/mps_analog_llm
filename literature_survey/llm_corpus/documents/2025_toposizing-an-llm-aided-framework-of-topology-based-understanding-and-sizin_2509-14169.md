---
title: "TopoSizing: An LLM-aided Framework of Topology-based Understanding and Sizing for AMS Circuits"
authors: "Wei, Ziming; Kong, Zichen; Wang, Yuan; Pan, David Z.; Tang, Xiyuan"
date: "2025/09/17"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2509.14169"
pdf: "papers/2025_toposizing-an-llm-aided-framework-of-topology-based-understanding-and-sizin_2509-14169.pdf"
sha256: "e31a80c43b3fabd967c895f644004fbd6e7ec587685a818b079f22b2e9d60396"
pages_extracted: 14
---

# TopoSizing: An LLM-aided Framework of Topology-based Understanding and Sizing for AMS Circuits

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

TopoSizing: An LLM-aided Framework of
Topology-based Understanding and Sizing for AMS
Circuits
Ziming Wei1
, Zichen Kong2
, Yuan Wang2
, David Z. Pan3*
, Xiyuan Tang4*
1
School of EECS, Peking University, Beijing, China
2
School of Integrated Circuits, Peking University, Beijing, China
3
Department of Electrical and Computer Engineering, The University of Texas at Austin, USA
4
Institute for Artificial Intelligence, Peking University, Beijing, China
*Corresponding authors: dpan@ece.utexas.edu, xtang@pku.edu.cn
Abstract—Analog and mixed-signal (AMS) circuit design re-
mains challenging due to the shortage of high-quality data and
the difficulty of embedding domain knowledge into automated
flows. Traditional black-box optimization achieves sampling ef-
ficiency but lacks circuit understanding, which often causes
evaluations to be wasted in low-value regions of the design space.
In contrast, learning-based methods embed structural knowledge
but are case-specific and costly to retrain. Recent attempts with
large language models (LLMs) show potential, yet they often rely
on manual intervention, limiting generality and transparency.
We propose TopoSizing, an end-to-end framework that performs
robust circuit understanding directly from raw netlists and trans-
lates this knowledge into optimization gains. Our approach first
applies graph algorithms to organize circuits into a hierarchical
device–module–stage representation. LLM agents then execute
an iterative hypothesis–verification–refinement loop with built-
in consistency checks, producing explicit annotations. Verified
insights are integrated into Bayesian optimization (BO) through
LLM-guided initial sampling and stagnation-triggered trust-
region updates, improving efficiency while preserving feasibility.
On four real-world AMS circuits (OTA, FC-OTA, SACMP,
LDO) in a 55-nm CMOS process, TopoSizing achieves 100%
correctness in circuit understanding and parameter assignment.
In constraint-satisfaction tasks, it delivers 1.4×–4.8× higher
sample efficiency and 1.2×–3.5× faster runtime compared with
strong baselines, while requiring 2–4× fewer LLM calls than
prior LLM-based frameworks. Ablation studies confirm that
reliable circuit understanding is essential for these gains.
Index Terms—Analog and mixed-signal (AMS) circuits, elec-
tronic design automation (EDA), large language model (LLM),
Bayesian optimization (BO)
I. INTRODUCTION
Analog and mixed-signal (AMS) integrated circuits, such
as operational amplifiers, comparators, and voltage references,
remain indispensable in modern electronic systems. However,
their design continues to rely heavily on expert intervention
and domain-specific experience, making the process labor in-
tensive and difficult to scale. Among the various stages, device
sizing plays a particularly decisive role, as it determines key
performance metrics. Yet, sizing is notoriously challenging:
the design space is high-dimensional due to numerous in-
terdependent parameters, and performance evaluation requires
computationally expensive SPICE-level simulations. Together,
these factors render sizing a critical but highly complex stage,
one that this work directly aims to address.
The most straightforward way to formulate sizing is as
a black-box optimization problem, where device parameters
are directly mapped to performance metrics. In this setting,
heuristic algorithms [1] and Bayesian Optimization (BO) and
its variants [2]–[4] have been widely applied, offering high
sample efficiency and topology-agnostic applicability. How-
ever, black-box methods do not perform what expert designers
routinely do, namely circuit understanding. Without the
ability to interpret the structure and functionality of a circuit,
these approaches cannot map topology into meaningful design
constraints or guidance. As a result, they often fail to avoid
wasting evaluations in unpromising regions that a human
engineer would immediately exclude. This gap highlights the
importance of circuit understanding as a necessary ingredient
for practical automation.
Circuit understanding, in this context, refers to the ability
to reason from structure, parameters, and connectivity of
circuits to infer their implications on performance and to
translate this reasoning into design constraints or guidelines.
The pursuit of such understanding has motivated researchers to
explore learning-based methods. Early approaches trained sur-
rogate models that mapped design parameters to performance
metrics, effectively embedding domain knowledge into the
optimization loop [5]–[8]. Later efforts extended this idea with
graph-based algorithms to explicitly encode circuit structure,
capturing functional relationships between devices and mod-
ules [9]–[12]. While these methods enriched the representation
of circuits, they remain limited in practice: surrogate models
are typically case-specific and require retraining whenever
the topology changes, while dataset generation through costly
simulations often outweighs the savings they provide. Con-
sequently, despite their conceptual appeal, these approaches
cannot scale or deliver efficient design automation.
Up to this point, existing strategies reveal a fundamental
dilemma: achieving topology generalizability, sampling effi-
ciency, and embedded circuit understanding simultaneously
has appeared nearly impossible. Naturally, the criteria for a
arXiv:2509.14169v1
[cs.LG]
17
Sep
2025

## Page 2

form of circuit understanding suitable for automated design
can be summarized as follows: (a) explicit preservation
for reuse, meaning the extracted knowledge is represented
explicitly and stored in a form that can be reused across dif-
ferent design stages such as constraint formulation, testbench
generation, or layout guidance; (b) structural generality,
meaning the method can extend naturally to diverse circuit
families and topologies without case-specific retraining; and
(c) transparency and verifiability, meaning the intermediate
roles of devices, modules, and stages are made explicit in a
form that can be inspected, validated, and trusted.
However, achieving such circuit understanding is far from
trivial: it requires a powerful model that can reliably interpret
circuit topology, incorporate relevant domain knowledge, and
correctly map this understanding into design requirements.
Fortunately, the emergence of large-language models (LLMs)
offers a promising path forward. Trained in vast and diverse
corpora, LLMs possess broad prior knowledge that encom-
passes fundamental concepts in circuit theory and analog
design. In addition, they exhibit a degree of reasoning capa-
bility, enabling them to interpret circuit connectivity, propose
plausible sizing strategies, and support design-oriented infer-
ence. Moreover, their flexible input–output modalities make
them easily extensible within existing workflows. Together,
these characteristics suggest that LLMs could reconcile sam-
ple efficiency, topology transferability, and circuit knowledge
integration within a single framework. This possibility has
attracted increasing attention in the analog design automation
community [13]–[16].
Motivated by these advantages, recent works have explored
the use of LLMs in AMS electronic design automation
(EDA). However, current approaches still suffer from signif-
icant limitations in scalability, generality, and transparency;
a detailed review is provided in Section II. Together, the
limitations of existing approaches can be traced to two main
sources. The first lies in the unreliability of circuit under-
standing. LLMs are not naturally suited for reasoning over
graph-structured data [17] such as circuits, and raw netlists
pose particular difficulties. Unlike designer-authored netlists,
which often contain implicit hints such as ordered indexing,
hierarchical organization, or meaningful node names, raw
netlists, especially those extracted automatically from images
or heterogeneous sources, lack such structure. Their nodes
typically do not have informative labels, their ordering is
arbitrary, and functional modules are not delineated. Therefore,
it is unrealistic to assume that an LLM can directly parse
these netlists into the correct functional roles [18]. Achieving
reliable understanding requires working strictly from topology-
only information and deriving a robust, high-quality interpre-
tation akin to how human engineers decompose circuits. The
second difficulty lies in how to effectively leverage the attained
circuit understanding to achieve tangible improvements in
optimization efficiency. This requires first the choice of a prin-
cipled backbone optimization algorithm and then the design
of appropriate intervention mechanisms. However, existing
LLM-based works lack consensus on what constitutes the
most effective integration strategy, and the approaches adopted
so far remain relatively coarse. For example, some studies
bypass circuit knowledge entirely by invoking an optimization
algorithm directly, while others let the LLM propose candidate
sampling points or prune the search space based on the current
optimization state. A more systematic methodology is needed
to translate circuit understanding into design efficiency, along
with comparative studies to evaluate different intervention
strategies.
In response to the above challenges, we propose an end-to-
end framework that spans the analog front-end design flow,
taking a raw netlist as input and producing a sized circuit
as output, with circuit understanding accomplished in the
process and preserved as intermediate knowledge in textual
or annotated form. The framework begins with raw netlists
and performs robust circuit understanding based solely on
topology, assisted by graph algorithms and LLMs. This circuit
understanding is not only essential for guiding subsequent
design tasks within our framework but also tangible in the
sense that the extracted knowledge can be retained for potential
reuse in other applications. On top of this, sizing serves
both as a strengthened optimization stage and as a validation
mechanism: the acceleration achieved in sizing demonstrates
the practical benefits of circuit understanding, while the cor-
rectness of sizing outcomes in turn substantiates the reliability
of the proposed circuit understanding process.
Our main contributions are summarized as follows:
• We present a unified end-to-end framework for analog
front-end design that integrates LLMs into the work-
flow. Starting from raw netlists, the framework performs
topology-based circuit understanding and leverages the
extracted annotations to enhance downstream tasks, with
the knowledge preserved for potential reuse.
• We propose a graph-assisted methodology for reliable
circuit understanding. Circuit information is systemati-
cally extracted into a hierarchical representation across
three levels–component, module, and stage–and the LLM
executes iterative reasoning in a hypothesis-verification-
refinement loop, improving robustness while ensuring
interpretability.
• We introduce an efficient integration of LLM-based cir-
cuit understanding with BO. By incorporating LLM-
guided initial sampling and stagnation-triggered trust-
region updates, our approach improves BO efficiency
through simple yet effective mechanisms.
• We validate the framework on four real-world circuit
cases, where TopoSizing achieves 100% correctness in
both circuit understanding and parameter assignment. In
constraint satisfaction tasks, our method delivers 1.4×
to 4.8× higher sample efficiency and 1.2× to 3.5×
faster runtime compared with traditional optimization
baselines.

## Page 3

II. PRELIMINARIES
A. AMS Circuit Understanding and Sizing
From the above discussion, we note that our entire workflow
begins with establishing a reliable circuit understanding. To
evaluate this crucial step, we introduce two indicators: (i) the
classification accuracy of devices and modules, which reflects
whether circuit components are assigned to the correct func-
tional roles; and (ii) the correctness of design-constraint or
parameter assignment, which measures whether the extracted
circuit information can be faithfully translated into sizing-
related guidance.
To further demonstrate the practical value of circuit under-
standing, we take AMS circuit sizing as a downstream task.
AMS sizing refers to the process of tuning device-level param-
eters—such as transistor widths, lengths, or bias currents—so
that the resulting circuit meets all performance specifications
such as gain, bandwidth, power consumption, and stability.
Successful sizing requires searching over a high-dimensional
design space, and its efficiency can be significantly improved
when guided by accurate circuit understanding. Thus, sizing
serves both as a key design objective and as an auxiliary
validation of the usefulness of our framework.
B. Problem Formulation for Sizing
AMS circuit sizing can be cast as a constrained optimization
problem. For the feasibility scenario, the task is to identify a
design x ∈ X that meets all performance specifications:
find x ∈ X s.t. Fi(x) ≥ Ci, ∀i ∈ {1, . . . , Nc}, (1)
where Fi(x) is the i-th performance metric, Ci its specifica-
tion, and Nc the number of constraints.
For the single-objective constrained optimization, one target
metric Ft(x) is maximized while ensuring feasibility of all
other constraints:
max
x∈X
Ft(x) s.t. Fi(x) ≥ Ci, ∀i ̸= t. (2)
This formulation highlights the two experimental settings
considered in our framework: (i) pure feasibility search for
rapid constraint satisfaction, and (ii) performance-driven opti-
mization under design constraints.
C. LLMs for AMS EDA
Existing approaches of utilizing LLMs in AMS EDA can
be broadly divided into two categories.
The first line of work develops domain-specific LLMs tai-
lored for AMS design tasks. These methods aim to incorporate
circuit-specific knowledge into pretraining or fine-tuning, but
they are constrained by the scarcity of large, high-quality
datasets. Despite auxiliary efforts such as extracting netlists
from schematics or images [19], [20], automatically generating
sizing scripts [21], and constructing test benches [22], ensuring
correctness in circuit interpretation and parameter allocation
still requires substantial manual validation. This bottleneck
limits both scalability and reliability.
The second line of work attempts to directly apply general-
purpose LLMs within the design process. In the context of
sizing, for example, retrieval-augmented generation (RAG)
and multi-agent discussion have been explored to provide
design guidance [16], [23]. Other studies let LLMs intervene
in optimization directly [13], [14]. However, these approaches
typically process raw netlists without additional structure,
which restricts applicability to simple topologies. Moreover,
their interpretations are rarely verified, leaving the process
opaque and lacking transparency. Even in optimization integra-
tion, strategies remain coarse and inconsistent: some pipelines
bypass model intervention entirely [16], while others only
allow the LLM to propose a few candidate samples per
iteration [13].
Nevertheless, these efforts highlight the larger potential of
LLM-in-the-loop workflows: as commercial models continue
to improve, such pipelines could, in principle, achieve even
stronger design performance. The key challenge, however,
lies in ensuring interpretability and reliability. In particular,
existing approaches rarely verify whether the LLM’s circuit
understanding is correct or whether its design guidance is
appropriate—both of which are crucial for trustworthy automa-
tion. This gap motivates our work, where we propose a frame-
work that not only leverages LLMs for circuit understanding
but also incorporates mechanisms to validate and refine their
interpretations.
In summary, while these works demonstrate the potential
of LLMs in AMS EDA, they also reveal clear limitations in
scalability, generality, and verifiability. Addressing these gaps
motivates the systematic framework proposed in this paper.
D. Circuit Representation: Requirements and Choices
A central question in enabling LLMs for AMS EDA is
how to represent circuits in a form that is both information-
preserving and LLM-compatible. On the one hand, the rep-
resentation must preserve all relevant structural and parametric
information, since any loss of connectivity or device attributes
would compromise the fidelity of subsequent analysis. On
the other hand, the representation must be interpretable by
LLMs, whose pretraining distribution is dominated by natural
language rather than specialized graph data or domain-specific
syntaxes.
From the perspective of information completeness, electrical
circuits are naturally graphs: devices and terminals can be
modeled as nodes, while electrical connections form edges.
Such graph-based representations are widely used in learning-
based electronic design automation (EDA), particularly with
graph neural networks (GNNs) [9], [10], [12]. They provide a
lossless encoding of circuit topology and attributes. However,
these representations are not well aligned with LLMs. Raw
adjacency matrices or edge lists are high-dimensional, order-
sensitive, and difficult to tokenize effectively; linearizing them
introduces artificial ordering biases; and functional relation-
ships such as differential pairs or current mirrors remain
implicit [17]. Consequently, although graph representations

## Page 4

excel at preserving information, they do not naturally support
LLM reasoning.
Another direction is to augment the raw netlist with seman-
tic annotations. For instance, designers may explicitly mark
functional groups. Such annotations improve interpretability
but typically require substantial manual effort, making them
difficult to scale across large design spaces. A related approach
is to employ code-like domain-specific languages (DSLs)
for circuit description. These languages introduce specialized
syntax to represent modules and hierarchical structure in
a more formal and programmable way, akin to hardware
description languages [?]. While such DSLs can provide
concise and structured descriptions, they rely on conventions
that pretrained LLMs are not exposed to, and thus may
not be easily understood without additional fine-tuning or
task-specific adaptation. In summary, both directions have
demonstrated effectiveness within their respective application
scenarios, but neither fully satisfies the dual requirement of
being information-encompassing and LLM-compatible. This
gap motivates our approach, which aims to balance these trade-
offs by transforming graph-extracted circuit information into
structured natural-language descriptions that preserve essential
topology while remaining aligned with LLMs’ strengths.
III. METHODOLOGY
A. Overview
In this section, we describe the detailed implementation
of TopoSizing, a fully automated framework that integrates
graph-based processing with LLMs to achieve reliable circuit
understanding and sample-efficient optimization (Fig. 1). The
workflow is carried out in three stages. First, raw netlists
are transformed into hierarchical graph representations that
capture component, module, and stage-level structure. Based
on this representation, LLM agents iteratively perform circuit
understanding, producing explicit and verifiable annotations of
functional roles. Finally, this circuit understanding is coupled
with an adapted BO, where LLM guidance refines the design
space and gives instructions on trust region update. By moving
from structural encoding to functional interpretation and ulti-
mately to optimization, TopoSizing integrates all stages into a
coherent end-to-end pipeline.
B. Topological Information Extraction
The first step toward reliable circuit understanding is to
choose an LLM- and algorithm-friendly circuit representation
that is both expressive and easy to process. Therefore, we
attempt to construct a structured container that captures the
circuit at three hierarchical levels: component, module, and
stage. This organization preserves both fine-grained details
and higher-level functional context, making the representation
both expressive and easy to query. An additional advantage
is that hierarchical abstraction effectively reduces the con-
nection complexity inherent in graph-structured netlists. Both
advantages of this structured graph container are particularly
beneficial in improving the robustness of LLM-based circuit
understanding [24]. The design of this hierarchy is inspired by
how analog designers analyze circuits: starting from the supply
to divide the circuit into stages, then identifying common
module sub-circuits to quickly infer functionality, and finally
inspecting individual component connections to verify details.
1) Circuit to graph transformation: Circuits naturally
exhibit the properties of a graph: devices and nets form nodes,
and their connections define edges. Leveraging this intrinsic
structure, we transform a raw netlist into a structured, semanti-
cally enriched graph in a fully automated way. This process is
almost “free” since it reuses the connectivity already encoded
in the netlist and produces a unique, lossless representation
that preserves all original information.
G = (V, E),
V = Vdev ∪ Vnet,
Vdev = { vi | τ(vi) ∈ {NMOS, PMOS, R, C, I, V } },
Vnet = { vj | τ(vj) ∈ {net, GND, VDD} },
E ⊆ { (vdev
i , vnet
j , ℓ) | ℓ ∈ L },
L = {G, S, D, R, C, I, V }.
Here, G is the circuit graph with node set V and edge set
E. The node set is partitioned into device nodes Vdev and net
nodes Vnet, where τ(vi) denotes the node type. Each edge
ε ∈ E connects a device node to a net node with label l ∈ L,
where l indicates terminal roles or device types.
Edges ε ∈ E connect device nodes to net nodes with a label
ℓ ∈ L. For MOSFETs, labels indicate terminal roles: Gate (G),
Source (S), Drain (D). For other devices, the label corresponds
to the device type. Since connections only occur between
devices and nets, the resulting graph is bipartite, capturing
both topology and port-level semantics.
This structured encoding offers two key benefits: (1) it
is lossless, preserving every device, connection, and func-
tional role, and (2) it is directly compatible with graph-
based algorithms. As a result, it serves as an ideal foundation
for subsequent topology analysis, module recognition, and
optimization.
2) Sub-circuit Matching: To further simplify the circuit
graph while retaining meaningful functional structure, we
take advantage of a characteristic unique to analog circuits:
the frequent reuse of well-established basic modules such as
differential pairs, current mirrors, and cascode stages. This
is where our graph-based representation shows its strength:
once a circuit is represented as a graph, we can directly apply
subgraph algorithms to identify and manipulate these recurring
patterns.
In our approach, we first construct a library of commonly
used analog building blocks. Each building block is a labeled
subgraph in which both node types and edge types are
explicitly specified, encoding canonical topological patterns.
These templates not only guide the matching process, but also
link specific transistors and devices to their functional roles
within a module, enabling us to later reason about how design
parameters influence performance. The initial library covers
differential pairs, current mirrors, cascode stages, cascode

## Page 5

Sub-circuit Matching
Component Level Info
Stage-level Grouping
Module Level Info
Stage Level Info
Topological Information Extraction LLM-guided Optimization
LLM-guided Initial Sampling
LLM-guided TuRBO
Input
AMS Circuit
Graph Representation
Circuit Netlist
Netlist
subckt. OpAmp Vdd Vss
NM1 net0 net1 Vss Vss
NM2 net2 net2 net2 Vss
NM3 net3 Vin net4 VSS
......
Agent 1
Summarized Circuit Information
Design Parameters Assignment
Design Specifications
Design Information
Circuit Understanding
Agent 3
Design Space
Pruning
Trust Region
Update
Settings
Agent 1: Agent for Iterative Circuit Understanding
Agent 2: Agent for Initial Design Space Pruning
Agent 3: Agent for Trust Region Update Guidance
Agent 2
LLM Agents Design Specifications
Loop_Gain > 60 dB
Loop_Gain_UGB > 50MHz
Phase Margin > 60 degree
......
Simulation Settings
PDK: SMIC 55nm
Corner: TT
Temperature: 27 ℃
Proposed Sizings Simulation Results
Simulator
Fig. 1: The work flow of TopoSizing
current mirrors, Class-AB stages, and diode-connected MOS
devices. These modules are chosen because they constitute the
fundamental building blocks of analog design, are repeatedly
reused across circuits, and together span almost all practical
topologies [25], [26]. The library can also be readily extended
to more complex structures.
Based on the graph representation of the circuit, we perform
subgraph isomorphism to detect template matches. When a
match is found, we replace the matched devices with a sin-
gle supernode representing the identified module. Net nodes,
which define the electrical interconnection points, are pre-
served to maintain global connectivity. This replacement does
more than reduce complexity, it also embeds new semantic
information into the graph. By renaming the supernode and
adjusting the edge types at its boundary, we encode both
the module’s identity and its port-level roles directly into
the graph’s structure. This makes later queries trivial—design
tools or LLM agents can retrieve functional details simply by
inspecting node and edge labels.
3) Stage-level Grouping: Even after subcircuit matching
and module abstraction, the circuit graph can remain highly
complex: it may still contain a large number of nodes, no
clear hierarchical organization, and many connections that
a designer would not consider functionally important. To
further organize this structure, we take inspiration from a
common principle in analog circuit design, dividing the circuit
into stages, and formulate a rule-based method for grouping
components based on whether a continuous current can flow
through them.
The core idea is that components sharing a continuous
current supply path typically form a unified functional stage,
operating under the same biasing conditions and serving a
specific electrical role in the overall system. To implement this,
we first reduce the circuit graph by removing edges that do not
conduct current, such as those corresponding to capacitors or
MOSFET gate connections. This results in a pruned subgraph:
Gi = (Vi, Ei) ⊆ G.
Within this current-conduction graph, a conduction path is
defined as a sequence of alternating device and net nodes that
connect the supply node vVDD ∈ Vnet to the ground node
vGND ∈ Vnet:
Pk = (v1, e1, v2, . . . , vn), with v1 = vVDD, vn = vGND.
All such paths are enumerated and collected:
P = {P1, P2, . . . , PK}.
Rather than treating each conduction path independently, we
merge any two paths that share an internal node:
Pi ∼ Pj ⇐⇒ ∃v ∈ Pi ∩ Pj, v /
∈ {vVDD, vGND}.
This defines an equivalence relation over P, and the union of
equivalent paths forms a single stage:
Sl =
[
Pk∈[P]l
Pk, l = 1, . . . , L.
Once the stages {S1, . . . , SL} are identified, we define a
stage-level interconnection graph:
Gstage = (S, Einter),
where stages are divided and edges capture valid inter-stage
signal connections. Formally:
(Si, Sj) ∈ Einter iff













∃ v ∈ Vnet,
∃ ui ∈ Si ∩ Vdev,
∃ uj ∈ Sj ∩ Vdev,
(ui, v) ∈ E ∧ (uj, v) ∈ E.
This ensures that stages are connected only through shared

## Page 6

CS_gate
Vin
common_mode
Vout
Vin
Vin
CS_gate
CG_gate
Mid
Vout
Vin Vout
Vout
Vin Vin
common_mode Vref
Vref
Vout
CS_gate
CG_gate
Mid
differential pair current mirror cascode cascode current mirror lvt
Vref
Vref
CG_gate
CG_gate
CG_Vref
CS_Vref CG_Vref
CS_Vref
Mid Mid
Sub-circuit Template
cascode current mirror diode classAB stage resistor network capacitor network
diff_pair_n/p
Vout
common_mode
Vin
CM_n/p
Vref
S
mid
cascode_n/p
Vout
S
CG_gate cascode_CM_n/p
Vref
S
CG_gate
cascode_CM_n/p
Vref
S
CG_Vref
CS_Vref Diode_n/p
D
S
classAB
Vout
bottom
Vin
top
R/C_network
end
end
mid
Node Collapsing
Sub-circuit Template
Basic Components
Node Collapsing
No Node Collapsing Step needed
Vref
Fig. 2: The sub-circuit templates and supernode collapsing. The connection can be depicted accurately in text by embedding
names into the edges and nodes.
nets, with no device-level overlap, aligning with the modular
and hierarchical nature of analog circuit design. We also
annotate stages containing primary input or output terminals,
enabling the generation of a stage-level signal flow model. This
hierarchy serves as a bridge between low-level device repre-
sentation and higher-level functional architecture, providing
a clear, query-friendly structure for subsequent analysis and
LLM interaction.
The outcome of this step is a hierarchical graph-based
container for circuit information. At the lowest level, it records
individual transistors and their precise electrical connections;
at the next level, these devices are grouped into recognized
sub-circuits; and at higher levels, sub-circuits are organized
into functional stages. This layered organization preserves
complete connectivity while making it easy to query relation-
ships—whether between specific ports, within a module, or
across stages. In this way, the graph serves as an accessible and
structured repository from which later stages of the framework
can retrieve exactly the information they need, without losing
any of the detail present in the original netlist.
C. Circuit Understanding
1) Iterative Circuit Understanding via LLM Agents: To
achieve an accurate and robust understanding of the circuit’s
functional behavior, our framework employs dedicated LLM
agents that iteratively analyze and refine circuit interpretations.
Initially, an LLM agent processes structured circuit representa-
tions, including graph-based topology and hierarchical module
information, to generate a functional analysis.
Crucially, the agent incorporates an internal confidence
assessment mechanism, whereby it evaluates the certainty of
its interpretations for each node, connection, or functional
relationship. To ensure reliability, our model further applies
a simple checklist [27] during recognition, ensuring that all
device functions are successfully identified, that the func-
tional roles hypothesized for devices within each stage are
consistent with their interconnections, and that the overall

## Page 7

diff_pair_n_0
CM_n_0
CM_p_0
G
diff_pair_n_0
CM_n_0
CM_p_0 P
Vout
Vref G
GND
N
S
Vref G
N
P
VDD
S
S
S
Vout
D
common_mode
D
S S
I
I
I
C
C
C
D
D
diff_pair_n_0
CM_n_0
CM_p_0 P
Vout
Vref
GND
N
S
N
P
VDD
S
S S
Vout
D
common_mode
D
S S
I
I
I
C
D
D
VDD VDD
GND GND
Original Circuit Disable Non-current Edges Divided into Separated Stages
Stage 0 Input Stage Output Stage Other
N
N
N
GND
N N
P P P
S S
D
D
D D
D
S
G G
G
G
VDD
S
S
G
G
S S
G
I
I
I
S
C
G
C
C
D
D
D
Circuit in Graph Representation
with Sub-circuits matched
Grouping:
Stage 0: [ I_0, CM_n_0 ]
input Satge: [ In, CM_p_0, diff_pair_0, NM_0, PM_0 ]
Output Stage: [ Out, NM_1, PM_1 ]
Other: [C_0]
Supply Paths Info:
Satge 0: VDD-I-I_0-I-net-Vref-CM_n_0-S-GND
Input Stage: [VDD-S-CM_p_0-Vref-net, VDD-S-PM_0-S-
net]-Vout-diff_pair_0-common_mode-net-D-NM_0-S-GND
Output Stage: VDD-S-PM_1-D-Vout-D-NM_1-S-GND
Inter-stage Connection:
Stage 0-Input Stage: CM_n_0-Vref-net-G-NM_0
Satge 0-Output Stage: CM_n_0-Vref-net-G-NM_1
Input Stage-Stage 2: diff_pair_0-Vout-net-G-PM_1, PM_0-D-net-G-PM_1
Input Stage-Other: diff_pair_0-Vout-net-C-C_0, PM_0-D-net-C-C_0
Output Stage-Other: PM_1-D-net-C-C_0, PM_1-G-net-C-C_0, NM_1-D-net-C-C_0
Fig. 3: Example of the grouping process of the matched circuit. The circuit topology can be then presented in text as shownn
in the lower half of the figure.
Agent for circuit
understanding
G
N
N
N
GND
N N
P P P
S S
D
D
D D
D
S
G G
G
G
VDD
S
S
G
G
S
S
G
I
I
I S
C
G
C
C
D
D
diff_pair_n_0
CM_n_0
CM_p_0 P
Vout
Vref
G
GND
N
S
Vref G
N
P
VDD
S
S
S
Vout
D
common_mode
D
S
S
I
I
I
C
C
C
D
D
Stage 0 Stage 1 Stage 2
IN OUT
Graph-based
Representation
Sub-circuit
Matching
Stage-level
Grouping
Circuit Netlist
Component Level
Module Level
Stage Level
N
Y
Confidence > threshold ?
Ask for more
information
Assign Design
Parameters
Summarize Design
Information
Topological Infromation Extraction Circuit Understanding
Work Flow of LLM Perfoming Circuit Understanding
Fig. 4: LLM agent performs circuit understanding based on
the hierarchical topology information extracted and stored in
the previous steps.
circuit hierarchy and connectivity align with the inferred
functional assignments. Only after these checks are satisfied
does the agent output its result as confident. When the agent
detects ambiguities or low-confidence regions, it generates
targeted queries to disambiguate these uncertain elements.
These queries are answered through subsequent interactions,
enabling the model to iteratively refine its understanding.
The iterative process continues until the agent’s confidence
surpasses a predefined threshold, at which point the analysis
is considered sufficiently accurate and the iteration terminates.
This confidence-driven stopping criterion ensures that the
model dedicates more effort to complex or uncertain circuit
regions, while avoiding unnecessary computations on well-
understood portions.
Throughout this workflow, a coordination agent collects and
organizes the progressively refined insights into structured
prompts. These comprehensive representations encode the
hierarchical and functional properties of the circuit, facilitating
downstream tasks such as optimization guidance and design
space exploration. By integrating confidence-aware iterative
analysis, the system achieves a high-fidelity understanding of
complex circuit structures in an efficient and scalable manner.
2) LLM-Assisted Design Parameter Assignment: In addi-
tion to functional understanding, the framework leverages
LLM agents to assign design parameters in a symmetry-aware
manner. Many analog circuits contain inherently symmetric or
replicated structures, such as differential pairs, current mirrors,
or cascaded bias branches, where corresponding devices share
identical or proportionally related sizing parameters. By auto-
matically detecting these symmetric or functionally equivalent
components from the hierarchical circuit representation, the
LLM enforces consistent parameter assignments across them.
This process significantly reduces the effective dimensionality
of the design space, as multiple variables can be tied together
under symmetry constraints rather than optimized indepen-
dently. Such parameter tying mirrors common practices in
manual sizing, where designers pre-define equality or ratio
relationships to maintain matching characteristics and simplify
the optimization process. Incorporating this capability into
the automated framework not only accelerates convergence

## Page 8

but also ensures that the generated designs adhere to well-
established analog design principles.
D. LLM-guided Optimization
Our ultimate goal is to leverage the circuit knowledge
learned by the LLM to accelerate device sizing. BO is a
classical and highly efficient search algorithm in device sizing,
making it a natural choice as our optimization backbone.
To inject domain knowledge, we leverage the LLM’s cir-
cuit reasoning to intervene at two critical factors influencing
BO performance: enhancing initial sampling quality through
conservative space pruning, and refining trust region updates
via selective guidance when stagnation occurs. This targeted
integration preserves BO’s efficiency while steering the search
with domain-informed direction.
1) LLM-Guided Initial Sampling: The proposed LLM-
guided optimization framework addresses the device sizing
problem in analog circuit design, where optimal sizing pa-
rameters are strongly dependent on the specific technology
library. Due to the inherent variations in device models, sizing
solutions cannot be directly transferred across technologies.
Consequently, the LLM cannot generate high-performance
design points without prior knowledge; instead, it must rely
on representative samples to infer promising regions of the
design space.
To this end, the optimization process begins with an initial
sampling stage. Based on the structural and functional infor-
mation extracted in the previous stages, the LLM performs
a loosely constrained search space pruning. The pruning is
intentionally conservative to avoid excluding potentially viable
designs. For example, only basic feasibility constraints are
imposed, such as ensuring that the width-to-length ratio (W/L)
of differential pairs exceeds 5. This eliminates parameter
configurations that are clearly infeasible while retaining a
broad search range.
The initial sampling is conducted primarily within the LLM-
pruned subspace, ensuring that most samples are concentrated
in regions more likely to yield good performance. However, to
preserve global exploration capability, a controlled fraction of
samples is drawn from outside the pruned region. This hybrid
sampling strategy provides the optimizer with an informed and
more effective initial dataset, while avoiding premature over-
restriction of the search domain.
Algorithm 1 Initial Sampling
1: Input:
D, DP - original and pruned design space
α - Pruned space sampling ratio
2: Sample {xp
i } ∼ LHS(P)
3: Sample {xr
i } ∼ LHS(D \ P)
4: xi ← {xp
i } ∪ {xr
i } with ratio α : (1 − α)
5: Evaluate FoMi ← {FoM(xi)}
6: return (xi, FoMi)
Algorithm 2 LLM-Guided TuRBO
1: Input: Ninit, Niter, K, αinc > 1, αdec ∈ (0, 1), rmin <
rmax
2: Randomly sample Ninit points, fit GP0; set x⋆
=
arg max FoM, FoM⋆
= max FoM, r0 ∈ [rmin, rmax],
no imp= 0
3: for t = 0 to Niter − 1 do
4: Within T Rt = {x : ∥x − x⋆
∥∞ ≤ rt}, simulate and
update GPt+1
5: ymax = max FoM(Xt), xnew = arg max FoM(Xt)
6: If ymax > FoM⋆
: (x⋆
, FoM⋆
) = (xnew, ymax), rt+1 =
min(αincrt, rmax), no imp=0;
7: else: rt+1 = max(αdecrt, rmin), no imp+=1
8: If no imp≥ K: query LLM, receive (x̃, r̃), set x⋆
←
x̃, rt+1 ← clip(r̃; rmin, rmax), no imp=0
9: end for
10: Return: (x⋆
, FoM⋆
)
2) Stagnation Triggered Trust Region Update: Following
the initial sampling, the optimization proceeds using the
TuRBO (Trust Region BO) algorithm [28], which adap-
tively explores the high-dimensional design space in parallel.
While TuRBO efficiently exploits local structure, it may be-
come trapped in sub-regions when performance improvements
plateau. To address this, the LLM is incorporated as an
adaptive guide for trust region adjustment.
The LLM is not invoked at every iteration to reduce
computational and resource overhead. Instead, its intervention
is triggered by a stagnation criterion, specifically, when the
best observed performance fails to improve over a predefined
number of consecutive iterations. Upon activation, the LLM
analyzes the accumulated performance data together with
the topological and functional circuit information. It then
generates refined constraints or directional guidance to adjust
the trust region boundaries, steering the optimizer toward
unexplored yet promising regions of the parameter space.
This selective intervention strategy strikes a balance be-
tween the exploitation capacity of TuRBO and the global
redirection enabled by the LLM. By limiting LLM queries
to moments of optimization stagnation, the framework mini-
mizes unnecessary model calls while maintaining the ability
to dynamically reorient the search toward high-performance
regions.
IV. EXPERIMENTAL RESULTS
A. Testcases
We select four real-world analog circuits to evaluate the
design efficiency of TopoSizing: a two-stage operational
transconductance amplifier (OTA), a folded-cascode opera-
tional transconductance amplifier (FCOTA), a StrongArm latch
comparator (SACMP), and a low-dropout regulator (LDO). All
circuits are implemented using the commercial SMIC 55nm
CMOS process and are derived from practical engineering
applications, ensuring their real-world relevance.

## Page 9

tca
Two-stage OTA Folded Cascode OTA StrongArm Latch Comparator Low Dropout Voltage Regulator
Fig. 5: Schematics of four real-world circuits.
As illustrated schematically in Fig. 5, these circuits are
carefully chosen to cover a wide range of topologies, per-
formance specifications, and design space complexities. The
following sections detail the rationale behind the selection of
these benchmarks and describe the corresponding experimental
setups.
OTA: This circuit includes 11 design parameters: 4 tran-
sistor widths, 4 transistor lengths, 2 transistor ratios, and 1
capacitance value. The target specifications are:
C =
(
Gain ≥ 40 dB, GBW ≥ 50 MHz
Phase margin ≥ 60◦
, Power ≤ 0.5 mW
(3)
FCOTA: This circuit contains 20 design parameters: 7 transis-
tor widths, 7 lengths, 4 ratios, and 2 capacitance values. The
circuit comprises 33 components, including both core stages
and biasing networks. The target specifications are :
C =









Gain ≥ 100 dB, GBW ≥ 30 MHz
Phase margin ≥ 60◦
, Power ≤ 1 mW
PSRR ≥ 100 dB, CMRR ≥ 100 dB
Noise ≤ 30 mV
(4)
LDO: The LDO test case features 27 design parameters:
10 transistor widths, 10 lengths, 4 ratios, 2 resistances, and
1 capacitance. With a design space of approximately 1069
sampling points, it is the largest among all test cases. The
target specifications are:
C =





∆V ≤ 0.1 V, Setup Time ≤ 15 ns
PSRR ≥ 60 dB, Noise ≤ 5 uV/
√
Hz
Dropout ≤ 150 mV
(5)
SACMP: The StrongArm latch comparator is a dynamic
circuit with 14 design parameters: 6 transistor widths, 6
lengths, and 2 capacitances. It includes both symmetrical and
cross-coupled structures. The target specifications are:
C =
(
Power ≤ 40 uW, Set Delay ≤ 4 ns
Reset Delay ≤ 4ns, Noise ≤ 120 uV
(6)
We apply TopoSizing to all four test cases and record
both the number of simulation samples and the runtime. To
provide a comprehensive performance comparison, we also
evaluated commercial auto-sizing tools from Cadence Virtuoso
[29], as well as two academic baselines: TuRBO [28] and a
reinforcement learning–based approach [4], under the same
design specifications.
B. Implementation Details
We evaluate the proposed LLM-guided circuit optimization
framework through a series of experiments encompassing
baseline comparisons, recent LLM-aided approaches, and an
ablation study. All optimization tasks are performed using
TuRBO implemented in PyTorch, with circuit simulations
conducted in Cadence Spectre under the SMIC 55 nm PDK.
Each circuit is optimized over ten independent runs to mitigate
stochastic effects, and the average results are reported. The
language model adopted in all LLM-related settings is GPT-
4o [30] accessed via the OpenAI API, with the generation
temperature fixed at 0.5. Unless otherwise specified, no fine-
tuning or task-specific pretraining is applied.
For circuit understanding, we rely on graph-based process-
ing. Circuit netlists are first converted into graph represen-
tations, where devices and terminals correspond to nodes and
electrical connections to edges. We employ networkx to sup-
port sub-circuit matching via subgraph isomorphism, as well
as path-finding algorithms for identifying connectivity patterns
between modules. This enables systematic construction of the
component–module–stage hierarchy. All intermediate graph
structures are stored in a standardized JSON-like format,
which preserves device attributes, connectivity, and hierarchy
for subsequent reasoning and optimization.
The allocation of design parameters (e.g., widths, lengths,
finger numbers) to each device is supposed to be done by
LLMs. Within our framework, this assignment is automatically
inferred during circuit understanding and achieves complete
correctness across all benchmarks. In contrast, prior LLM-
based approaches that directly process raw netlists often fail
to achieve correct parameter assignment, leading to ambiguous
or inconsistent sizing configurations. To ensure fairness in our
comparisons, we manually complete the parameter assignment
step before optimization when evaluating these baselines. This
guarantees that all methods begin the sizing stage from an
equally valid configuration, and that the performance dif-
ferences reflect only the efficiency of the LLM intervention
strategy.
For non-LLM baselines, we consider a pure TuRBO im-
plementation without language model guidance, a reinforce-
ment learning–based optimization strategy [4], and the built-
in optimizer provided by Cadence Virtuoso. For LLM-based
baselines, all methods employ GPT-4o in combination with

## Page 10

TuRBO to ensure fairness, while differing only in the interven-
tion strategy. Specifically, we compare our proposed adaptive
intervention with a variant where the model intervenes at every
optimization iteration [14], as well as the ADO-LLM approach
in which two candidate points are proposed at each round [13].
To better reflect practical design requirements, experiments
are conducted under two distinct scenarios: constraint satisfac-
tion optimization, which focuses on rapidly identifying feasi-
ble solutions, and single-objective optimization, which aims to
achieve progressive performance improvement over multiple
iterations. In the ablation study, we assess the contributions
of two key components by selectively removing either the
topology analysis module or the LLM-guided enhancement
to the TuRBO process, while keeping all other settings fixed.
C. Optimization Formulation
We now present the mathematical formulation of the op-
timization problems considered in our experiments. We first
introduce the notation for the design parameters, performance
metrics, performance specifications, and optimization direc-
tions:
X = (X1, . . . , Xn) ∈ D ⊆ Rn
,
F(X) = (F1(X), . . . , Fm(X)),
C = (C1, . . . , Cm),
φ = (φ1, . . . , φm), φi ∈ {−1, 1}.
Here, X denotes the n-dimensional vector of design param-
eters, and D is the corresponding design space. F(X) is the
m-dimensional vector of performance metrics obtained from
circuit simulation, and C is the m-dimensional vector of per-
formance specifications. The sign φi encodes the optimization
direction:
φi =
(
1, for metrics to be maximized,
−1, for metrics to be minimized.
1) General normalized score: For unified evaluation of
metrics with different optimization directions, we define the
normalized score of the i-th metric as:
ri(X) = φi ·
Fi(X) − Ci
max(|Fi(X)|, |Ci|)
, (7)
where the numerator adjusts the sign according to the opti-
mization direction, and the denominator normalizes the scale.
A positive ri means the i-th metric exceeds its specification,
while a negative value indicates violation.
D. Multi-constraint feasibility optimization
For a pure feasibility problem, we focus on minimizing
violations. The FoM is defined as:
FoMfeas(X) =
m
X
i=1
min 0, ri(X)

, (8)
so that each constraint contributes its violation magnitude
if unsatisfied, and 0 otherwise. The optimal feasible point
achieves FoMfeas = 0.
1) Single-objective constrained optimization: When opti-
mizing a specific target metric Ft(X) while satisfying all
constraints, we separate the non-target penalties from the target
reward. We first quantify the penalties from all non-target
metrics:
Rnon-target(X) =
m
X
i=1
i̸=t
min 0, ri(X)

, (9)
where each non-target metric contributes its violation magni-
tude if unsatisfied, and 0 otherwise.
The contribution of the target metric Ft(X) depends on
whether all non-target constraints are satisfied:
Rtarget(X) =
(
rt(X), if ri(X) ≥ 0 ∀i ̸= t,
min 0, rt(X)

, otherwise.
(10)
This ensures that the target metric receives a positive reward
only when all other constraints are satisfied; otherwise it is
also penalized if it fails to meet its own specification.
Finally, the overall figure of merit is obtained by combining
the two terms:
FoMsingle(X) = Rnon-target(X) + Rtarget(X). (11)
E. Constraints Satisfaction
Tab. III summarizes the optimization results, averaged over
ten independent runs. Compared with traditional sizing meth-
ods, our framework achieves clear improvements in both
sampling efficiency and convergence speed across most test
cases. Compared to other LLM-aided approaches, TopoSizing
attains higher optimization efficiency while requiring substan-
tially fewer LLM calls. Although our framework introduces
additional preprocessing and circuit-understanding steps, the
overall runtime remains lower, making this trade-off both
practical and acceptable.
F. Single-object Optimization
We further evaluate each test case by selecting a single
performance metric for single-objective optimization, in order
to examine whether the proposed framework can not only
accelerate optimization but also establish a correct mapping
from design parameters to performance metrics. This experi-
ment is designed to test whether LLMs, when guided through
our framework, can capture such relationships more reliably
and thereby enhance performance in larger-scale optimization
tasks. As baselines, we compare against Bayesian Optimiza-
tion (BO) with 400 samples, an LLM-based method with 400
samples, and BO with 800 samples. The experimental results
confirm our expectation: except for the relatively simple 2stage
case—where the LLM’s interpretation is already sufficiently
accurate and thus improvements are less pronounced—our
framework consistently achieves comparable or better opti-
mization outcomes with fewer simulations.
Closer inspection of the curves reveals three important
observations. (1) Our framework starts from a higher FoM
immediately after the initial sampling stage, indicating that

## Page 11

Fig. 6: FoM versus sample count in the single-objective optimization, with curves representing different methods and shaded
areas showing variation across runs.
the LLM-guided initialization provides a strong advantage.
(2) Long stagnation phases rarely occur, as our intervention
strategy effectively invokes the LLM to adjust the search
space once stagnation is detected. (3) Comparing the baselines
ADO and LEDRO further illustrates the role of understanding
quality: in cases where the LLM’s interpretation is reliable
(e.g., 2stage and FC), region-based pruning of the design space
outperforms point-wise proposal strategies; in contrast, when
the interpretation is less reliable (e.g., LDO), region-based
modification may amplify errors and lead to performance
degradation. These observations together validate both the
effectiveness and the limitations of LLM-guided optimization,
while highlighting the advantages of our proposed integration
strategy.
G. Ablation Study
Tab. II presents the results of the ablation study, which
demonstrate the effectiveness of the proposed TopoSizing
framework. The analysis shows that both the topological
information extraction and the iterative understanding by LLM
contribute to improving circuit understanding accuracy. Fur-
thermore, the enhanced understanding translates into higher
optimization efficiency, thereby validating the overall effec-
tiveness of our framework.
H. Case Study
Finally, we conduct a case study on the most complex
FCOTA circuit to evaluate the reliability of circuit under-
standing in a concrete setting. As shown in Fig. 7, when
directly reading the raw netlist, the LLM produces incorrect
functional interpretations, failing to capture key structural rela-
tionships. In particular, for the FCOTA with a common-mode
feedback (CMFB) module, the model—without any auxiliary
structure—misidentifies devices around the output node(s),
conflating CMFB-related elements with the main signal path.
This leads to an erroneous assignment of roles in the first stage
(e.g., mislabeling the load/sleeve devices and the input pair),
which propagates downstream. In contrast, after applying
our framework (Fig. 8), the hierarchical preprocessing and
TABLE I: Optimization results on four real-world circuits:
Comparison With Conventional Methods
Category Algorithm
Test Cases
OTA FCOTA SACMP LDO
Sample #
Proposed 18 32 18 22
TuRBO 25 66 107 37
RL 49 86 86 60
Virtuoso 197 142 108 400
Eff. impr. 1.4× 2.2× 4.8× 1.4×
Runtime (s)
Proposed 167 608 273 234
TuRBO 234 1270 1907 412
RL 2178 4705 4407 6553
Virtuoso 324 947 950 1141
Speed up 1.2× 1.5× 3.5× 1.4×
TABLE II: Results of Ablation Study
Category Algorithm
Test Cases
OTA FCOTA SACMP LDO
Sample #
Proposed 18 32 18 22
W/O IU∗
19 52 69 39
W/O TIE†
37 66 107 37
Accuracy of
Classification
Proposed 100% 100% 100% 100%
W/O IU 100% 87% 89% 86%
W/O TIE 100% 48% 58% 59%
* Iterative understanding by LLM
† Topological Information Extraction
iterative verification (topological extraction, conduction-graph
filtering, port-role inference, and grouping) enable the model
to disentangle the CMFB loop from the differential signal
path and to recover a complete and correct stage-level inter-
pretation. This comparison highlights that our framework not
only improves accuracy but also ensures robustness, eventually
reaching 100% correctness through repeated verification.

## Page 12

TABLE III: Optimization results on four real-world circuits:
Comparison With LLM-aided Methods
Category Algorithm
Test Cases
OTA FCOTA SACMP LDO
Sample #
Proposed 18 32 18 22
LEDRO 18 59 60 33
ADO-LLM 24 55 72 35
Eff. impr. 1.0× 1.6× 2.8× 1.5×
Runtime (s)
Proposed 167 608 273 234
LEDRO 172 1328 836 407
ADO-LLM 233 1081 972 427
Speed up 1.0× 1.8× 3.6× 1.7×
LLM Calls #
Proposed 1.1 3.3 2.2 2.1
LEDRO 2.3 7.4 7.5 4.1
ADO-LLM 3.0 7.0 9.0 4.4
Result of LLM Analyzing Raw Netlist
W/O Topological Information Extraction & Iterative Understanding
Correct Classification Wrong Classification
Fig. 7: LLM processing raw netlist
V. CONCLUSION
We propose TopoSizing, a stable and efficient topology-
aware framework for analog circuit sizing. TopoSizing ac-
celerates the optimization process through LLM integration
for circuit topology comprehension and design space pruning.
Experiments across four real-world test cases demonstrate its
superior sampling efficiency and time efficiency compared to
prior sizing tools from both industry and academia.
This fully automated and efficient design framework demon-
strates significant practical value, offering a novel approach
to accelerate traditional analog sizing processes through the
integration of advanced AI capabilities.

## Page 13

Now I can confirm that PM_4 and PM_5 are common source
transistors for amplification in the output stage. R_0 and R_1 forms a
resistor network that provides the average common-mode voltage for
the common-mode feedback amplifier, of which diff_pair_p_0 is the
input pairs. Capacitors C_0 and C_1 are connected in a special
configuration of cascode Miller-compensation.
The functions of each component are:
diff_pair_n_0: input pair of the input stage, ......
Here is the final prompt with assigned parameters:
This is a fully differential two-stage folded cascode amplifier. The first
stage consists of a differential input pair with a telescopic cascode
structure. ......
W1: Gate width of input pairs of the input stage
L1: Gate length of the input pairs of the input stage
W/ Topological Information Extraction & Iterative Understanding
Circuit Understanding Round 1
Stage 0 Stage 1 Stage 2 Stage 3 Output Stage Output Stage
Input Stage Input Stage
Circuit Representation with Hierarchical Information
(W,L)1
(W,L)5
(W,L)3
(W,L)2
(W,L)4
(W,L)6
C
R
(W,L)7
Circuit Understanding W/ Design Parameters Assigned
Conversation
Prompt
Agent
Topological
Information
Topological
Information
Agent
The following presents the classification of an analog circuit
according to its current supply branches. Please infer the overall
function of the circuit and explain the role of each component.
Stage-level Grouping:
Input Stage: [ diff_pair_n_0, NM_0, cascode_n_0, ...], Supply Paths:
VDD-{S}-cascode_p_0-{Vout}-net-{Vout}-cascode_n_0-{S}-GND, ......
Output Stage : ……
......
Between Stage Connections:
Input Stage-Output Stage : cascode_p_0-{Vout}-net-{G}-PM_5, ......
......
Ask for more information about connections of unsure nodes. List
them in the following format:
Unsure nodes: [node_name_1, node_name_2]
The circuit appears to be a two-stage amplifier featuring a folded-
cascode input. Bias networks and common mode feedback circuit
are also included.
Input stage is a folded-cascode structure. Output stage seems to
have a current mirror and a transistor as output driver. Stage 0 likely
serves as the biasing stage with a cascode current mirror and an
ideal current source. ......
Unsure nodes:
[PM_4, PM_5, R_0, R_1, C_0, C_1, diff_pair_p_0]
Connection Info:
PM_4: PM_4-{G}-net-{Vout}-cascode_p_0, PM_4-{G}-net-{Vout}-
cascode_n_0, PM_4-{S}-VDD, PM_4-{D}-Vout-{D}-NM_5, ......
......
Stage-level Information
Query made by LLM Agent
Detailed Conncetion Inforamation
Summarized Circuit Understanding
Matched Sub-circuits Correct Classification Connection Info Required
Stage Partition
Assigned
Parameters
Fig. 8: The detailed steps of our iterative circuit understanding process and final output with parameter assigned.

## Page 14

REFERENCES
[1] R. A. Vural and T. Yildirim, “Analog circuit sizing via swarm intelli-
gence,” AEU-International journal of electronics and communications,
vol. 66, no. 9, pp. 732–740, 2012.
[2] W. Lyu, P. Xue, F. Yang, C. Yan, Z. Hong, X. Zeng, and D. Zhou, “An
efficient bayesian optimization approach for automated optimization of
analog circuits,” IEEE Transactions on Circuits and Systems I: Regular
Papers, vol. 65, no. 6, pp. 1954–1967, 2017.
[3] K.-E. Yang, C.-Y. Tsai, H.-H. Shen, C.-F. Chiang, F.-M. Tsai, C.-A.
Wang, Y. Ting, C.-S. Yeh, and C.-T. Lai, “Trust-region method with
deep reinforcement learning in analog design space exploration,” in 2021
58th ACM/IEEE design automation conference (DAC). IEEE, 2021, pp.
1225–1230.
[4] Z. Kong, X. Tang, W. Shi, Y. Du, Y. Lin, and Y. Wang, “Pvtsizing:
A turbo-rl-based batch-sampling optimization framework for pvt-robust
analog circuit synthesis,” in Proceedings of the 61st ACM/IEEE Design
Automation Conference, 2024, pp. 1–6.
[5] A. F. Budak, P. Bhansali, B. Liu, N. Sun, D. Z. Pan, and C. V. Kashyap,
“Dnn-opt: An rl inspired optimization for analog circuit sizing using
deep neural networks,” in 2021 58th ACM/IEEE Design Automation
Conference (DAC). IEEE, 2021, pp. 1219–1224.
[6] A. F. Budak, D. Smart, B. Swahn, and D. Z. Pan, “Apostle:
Asynchronously parallel optimization for sizing analog transistors using
dnn learning,” in Proceedings of the 28th Asia and South Pacific
Design Automation Conference, ser. ASPDAC ’23. New York, NY,
USA: Association for Computing Machinery, 2023, p. 70–75. [Online].
Available: https://doi.org/10.1145/3566097.3567880
[7] A. F. Budak, M. Gandara, W. Shi, D. Z. Pan, N. Sun, and B. Liu, “An
efficient analog circuit sizing method based on machine learning assisted
global optimization,” IEEE Transactions on Computer-Aided Design of
Integrated Circuits and Systems, vol. 41, no. 5, pp. 1209–1221, 2021.
[8] S. Kim, Z. Wang, S. Lee, Y. Oh, H. Zhu, D. Kim, and D. Z. Pan, “Ppaas:
Pvt and pareto aware analog sizing via goal-conditioned reinforcement
learning,” arXiv preprint arXiv:2507.17003, 2025.
[9] H. Wang, K. Wang, J. Yang, L. Shen, N. Sun, H.-S. Lee, and S. Han,
“Gcn-rl circuit designer: Transferable transistor sizing with graph neural
networks and reinforcement learning,” in 2020 57th ACM/IEEE Design
Automation Conference (DAC), 2020, pp. 1–6.
[10] Z. Li and A. C. Carusone, “Design and optimization of low-dropout
voltage regulator using relational graph neural network and reinforce-
ment learning in open-source sky130 process,” in 2023 IEEE/ACM
International Conference on Computer Aided Design (ICCAD). IEEE,
2023, pp. 01–09.
[11] M. Liu, W. J. Turner, G. F. Kokai, B. Khailany, D. Z. Pan, and H. Ren,
“Parasitic-aware analog circuit sizing with graph neural networks and
bayesian optimization,” in 2021 Design, automation & test in Europe
conference & exhibition (DATE). IEEE, 2021, pp. 1372–1377.
[12] S. Lee, Z. Wang, S. Kim, T. Lee, Y. Lai, and D. Z. Pan, “Dice: Device-
level integrated circuits encoder with graph contrastive pretraining,”
arXiv preprint arXiv:2502.08949, 2025.
[13] Y. Yin, Y. Wang, B. Xu, and P. Li, “Ado-llm: Analog design bayesian
optimization with in-context learning of large language models,” in Pro-
ceedings of the 43rd IEEE/ACM International Conference on Computer-
Aided Design, 2024, pp. 1–9.
[14] D. V. Kochar, H. Wang, A. P. Chandrakasan, and X. Zhang, “Ledro: Llm-
enhanced design space reduction and optimization for analog circuits,”
in 2025 IEEE International Conference on LLM-Aided Design (ICLAD).
IEEE, 2025, pp. 141–148.
[15] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“Analogcoder: Analog circuit design via training-free code generation,”
in Proceedings of the AAAI Conference on Artificial Intelligence, vol. 39,
no. 1, 2025, pp. 379–387.
[16] J. Shen, Z. Chen, J. Zhuang, J. Huang, F. Yang, L. Shang, Z. Bi,
C. Yan, D. Zhou, and X. Zeng, “Atelier: An automated analog circuit
design framework via multiple large language model-based agents,”
IEEE Transactions on Computer-Aided Design of Integrated Circuits
and Systems, pp. 1–1, 2025.
[17] J. Guo, L. Du, H. Liu, M. Zhou, X. He, and S. Han, “Gpt4graph: Can
large language models understand graph structured data? an empirical
evaluation and benchmarking,” arXiv preprint arXiv:2305.15066, 2023.
[18] A. Dăescu, A. Guzu, G. Nicolae, and C. Dan, “Assessing the capabilities
of large language models to comprehend analog integrated circuits via
netlist analysis,” in 2025 3rd Cognitive Models and Artificial Intelligence
Conference (AICCONF), 2025, pp. 1–7.
[19] Z. Tao, Y. Shi, Y. Huo, R. Ye, Z. Li, L. Huang, C. Wu, N. Bai, Z. Yu,
T.-J. Lin, and L. He, “Amsnet: Netlist dataset for ams circuits,” in 2024
IEEE LLM Aided Design Workshop (LAD), 2024, pp. 1–5.
[20] H. Xu, C. Liu, Q. Wang, W. Huang, Y. Xu, W. Chen, A. Peng,
Z. Li, B. Li, L. Qi, J. Yang, Y. Du, and L. Du, “Image2net: Datasets,
benchmark and hybrid framework to convert analog circuit diagrams
into netlists,” in 2025 International Symposium of Electronics Design
Automation (ISEDA), 2025, pp. 807–816.
[21] W. Sun, Y. Han, B. Lan, Q. Peng, and J. Wan, “Anasizecoder: Code
generator for analog integrated circuit sizing automation via large lan-
guage model,” in 2025 International Symposium of Electronics Design
Automation (ISEDA), 2025, pp. 817–822.
[22] W. Chen, C. Liu, W. Huang, J. Lyu, M. Yang, Y. Du, L. Du, and J. Yang,
“Analogtester: A large language model-based framework for automatic
testbench generation in analog circuit design,” in 2025 International
Symposium of Electronics Design Automation (ISEDA), 2025, pp. 201–
207.
[23] C. Liu, W. Chen, A. Peng, Y. Du, L. Du, and J. Yang, “Ampagent: An
llm-based multi-agent system for multi-stage amplifier schematic design
from literature for process and performance porting,” arXiv preprint
arXiv:2409.14739, 2024.
[24] R. Ye, C. Zhang, R. Wang, S. Xu, and Y. Zhang, “Language is all a
graph needs,” arXiv preprint arXiv:2308.07134, 2023.
[25] B. Razavi, Design of analog CMOS integrated circuits, 2005.
[26] R. Harjani, R. Rutenbar, and L. Carley, “Oasys: a framework for analog
circuit synthesis,” IEEE Transactions on Computer-Aided Design of
Integrated Circuits and Systems, vol. 8, no. 12, pp. 1247–1266, 1989.
[27] J. Cook, T. Rocktäschel, J. Foerster, D. Aumiller, and A. Wang,
“Ticking all the boxes: Generated checklists improve llm evaluation and
generation,” 2024. [Online]. Available: https://arxiv.org/abs/2410.03608
[28] D. Eriksson, M. Pearce, J. Gardner, R. D. Turner, and M. Poloczek,
“Scalable global optimization via local bayesian optimization,” in
Advances in Neural Information Processing Systems, H. Wallach,
H. Larochelle, A. Beygelzimer, F. d'Alché-Buc, E. Fox, and R. Garnett,
Eds., vol. 32. Curran Associates, Inc., 2019.
[29] Cadence Design Systems, Inc., “Cadence virtuoso analog design envi-
ronment,” 2024, accessed: 2025-08-28.
[30] J. Achiam, S. Adler, S. Agarwal, L. Ahmad, I. Akkaya, F. L. Aleman,
D. Almeida, J. Altenschmidt, S. Altman, S. Anadkat et al., “Gpt-4
technical report,” arXiv preprint arXiv:2303.08774, 2023.

