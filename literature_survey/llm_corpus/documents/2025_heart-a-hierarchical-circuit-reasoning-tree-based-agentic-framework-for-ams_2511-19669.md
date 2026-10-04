---
title: "HeaRT: A Hierarchical Circuit Reasoning Tree-Based Agentic Framework for AMS Design Optimization"
authors: "Poddar, Souradip; Ho, Chia-Tung; Wei, Ziming; Cao, Weidong; Ren, Haoxing; Pan, David Z."
date: "2025/11/24"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2511.19669"
pdf: "papers/2025_heart-a-hierarchical-circuit-reasoning-tree-based-agentic-framework-for-ams_2511-19669.pdf"
sha256: "b2a107a00890cd5283746d417a921227bd132178def3a8bfe04a5238a83121a0"
pages_extracted: 9
---

# HeaRT: A Hierarchical Circuit Reasoning Tree-Based Agentic Framework for AMS Design Optimization

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

HeaRT: A Hierarchical Circuit Reasoning Tree-Based Agentic
Framework for AMS Design Optimization
Souradip Poddar1
, Chia-Tung Ho2
, Ziming Wei1
, Weidong Cao3
, Haoxing Ren2
, David Z. Pan1
1
ECE Department, The University of Texas at Austin 2
NVIDIA Corporation 3
The George Washington University
{souradippddr1@, dpan@ece.}utexas.edu
Abstract—Conventional AI-driven AMS design automation algorithms
remain constrained by their reliance on high-quality datasets to capture
underlying circuit behavior, coupled with poor transferability across
architectures, and a lack of adaptive mechanisms. This work proposes
HeaRT, a hierarchical circuit reasoning-based agentic framework for
automation loops and a step toward adaptive, human-style design opti-
mization. HeaRT consistently improves F1,subcircuits by ≥ 13.5% and
F1,loops by ≥ 37.8% over few-shot prompting baselines across multiple
LLM backbones on our 40-circuit AMS benchmark of flattened SPICE
netlists, even as circuit complexity increases. Our experiments further
show that HeaRT achieves ≥ 3× faster convergence in incremental design
adaptation tasks under specification shifts across diverse optimization
approaches, supporting both topology reconfiguration and sizing.
I. INTRODUCTION
Analog and mixed-signal (AMS) circuit design remains challeng-
ing to automate due to its fully custom flows, intricate trade-offs in
deep sub-micron technologies, and the high cost of re-optimization
when specifications change. Conventional Bayesian Optimization
(BO) methods [1–3] offer strong sample efficiency but struggle to
scale effectively in complex, high-dimensional design spaces. Recent
learning-based approaches, particularly reinforcement learning (RL)
[4–13], demonstrate improved scalability for larger circuits, yet incur
prohibitive simulation costs. Moreover, their reliance on manually
defined functional decompositions or handcrafted sub-block reward
formulations limits autonomy and scalability [12, 13]. Being purely
data-driven, these models exhibit an inherent black-box nature [14],
hindering their ability to capture circuit intuition or physical causality.
Consequently, they struggle to adapt to incremental architectural
changes, often requiring new data collection, and the lack of explain-
ability of the design trade-offs considered erodes designer confidence
in the quality of the results [15] (Fig. 1(a)).
Most recently, Large Language Model (LLM)-based methods [14,
16–26] have shown promise in advancing AMS design automation.
By leveraging reasoning and agentic capabilities, these models can
emulate key aspects of human design workflows. However, current
vanilla LLM-based approaches fail to incorporate the hierarchical
cognitive structure underlying expert AMS design [27–29], resulting
in unstructured and inconsistent reasoning that limits both their
credibility and practical effectiveness (Fig. 1(b)). Moreover, existing
automation methods lack explicit mechanisms to adaptively balance
design reuse and redesign within topology-sizing co-optimization,
thereby treating each specification update as a new problem. In turn,
they re-optimize entire circuits from scratch, leading to catastrophic
forgetting [30] of previously acquired design knowledge. In real AMS
workflows, many subcircuits are already layout-planned, variation-
optimized [31–35], or even silicon-proven, making such full re-
optimization impractical. This lack of architectural and contextual
awareness causes redundant computation, degraded sample efficiency,
and inconsistent reliability, limiting the practical deployability of
current automation frameworks in industrial design flows.
To fully harness the potential of LLMs for AMS automation,
we propose HeaRT, a multi-level agentic reasoning framework that
enables reasoning-grounded downstream applications. HeaRT draws
(b) Vanilla LLM-Based Optimization
✓ Structured and Consistent Reasoning
✓ Context-Grounded and Agentic Decision-Making
✓ Transparent and Traceable Reasoning Paths
✓ Token-Efficient Inference
Vanilla LLMs / Reasoning Models
(e.g., GPT-4o, GPT-5, Gemini-2.5Pro etc.)
Hierarchical Circuit Reasoning Tree Construction
(Offline)
AMS Design Tasks
(Online)
Agentic Retrieval
Raw SPICE Netlist
.subckt ….
……….
……….
……….
……….
.ends
u u … u b ck t
sc t …
ts t su
su ts …
s ts …
c …. st c s t b u ct sc t s
b ck s b ck su t
… u t
u ct u s …
s c s
…
c t ctu c t t … ck s u s c t ctu
s u t s … u t b s t t t t
Top-level Module
Raw SPICE Netlist
.subckt ….
……….
……….
……….
……….
.ends
(c) Hybrid Approach: Agentic Reasoning Grounded
in a Hierarchical Circuit Reasoning Tree
HeaRT
(a) Traditional Black-Box Optimization
u t
u t
ct t c
u
Reinforcement
Learning
Model-Free Algorithms
(e.g., Genetic Algorithms) Bayesian Optimization
u t
‼ High Sample-Inefficiency
‼ Lack of Adaptive Decision-Making
‼ Lack of Explainability
a
‼ Hallucination Risk
‼ Inconsistent Explanations
‼ Black-Box Reasoning and Limited Debuggability
‼ High Token Overhead
Fig. 1: An illustration of comparisons of traditional black-box optimiza-
tion algorithm, vanilla LLM-based approach, and the proposed HeaRT
framework for the AMS design task.
inspiration from the hierarchical abstraction principles inherent to hu-
man circuit design, constructing a hierarchical circuit reasoning tree
to enable structured, context-aware reasoning with query-conditioned
traversal paths for improved interpretability and debugging [36]
(Fig. 1(c)). Our key contributions are summarized as follows:
• We develop HeaRT, a multi-level agentic reasoning framework
that performs graph-guided top-down circuit decomposition with
hierarchical organization, followed by bottom-up knowledge
consolidation into a hierarchical circuit reasoning tree, enabling
structured, context-aware LLM reasoning for AMS design tasks.
• We construct a 40-circuit AMS benchmark repository spanning
multiple circuit families and complexity tiers, together with
expert-annotated ground-truth labels. The benchmark will be
publicly released upon acceptance.
• We evaluate HeaRT across multiple LLM backbones on this
benchmark and show that it consistently boosts F1,subcircuits
by ≥13.5% and F1,loops by ≥37.8% over few-shot prompting
baselines. When integrated with existing optimization algo-
rithms, HeaRT further enables ≥ 3× faster convergence in
incremental design-adaptation tasks involving both sizing and
topology reconfiguration.
II. PRELIMINARIES
A. LLM-Aided AMS Design Automation
The emergence of reasoning-enabled LLMs endowed with multi-
modal understanding and agentic decision-making capabilities [37–
45] has redefined autonomous problem-solving across diverse do-
mains, motivating their evaluation within EDA. Critically, expert
AMS designers reason about unseen circuit structures through hierar-
chical cognitive processes of abstraction, analysis, and bottom-up in-
terpretation [27–29]. However, existing LLM-based AMS approaches
arXiv:2511.19669v2
[cs.AI]
26
Mar
2026

## Page 2

Hierarchical Circuit Reasoning Tree Construction (Offline Phase)
Input: Raw SPICE Netlist
(Textual Representation)
Downstream Query / Task
Simulator
Priority-Guided Sizing
(Topology Preserved)
Identify Candidate Topologies
Objective-Driven
Topology Retrieval
Objective Set (𝑆)
Constraint Set (𝑈)
User Query 𝑄
✓ Incremental Design Adaptation
✓ Q&AAssistant
Query-Conditioned
Traversal
(Structured JSON Representation)
Hierarchical Circuit Reasoning Tree
LLM Reasoning &
Query Interpretation
LLM-Based Rank
Maintenance
Metric Schema +
ISSCC/JSSC Grounding
Tree Traversal
Priority Regions
Graph-Guided Reasoning
for Tree Construction
Functional Subcircuit
Decomposition
Structural Cues
Sdecomp+ metadata
Hierarchical Structure
Formation
DC Connectivity
Analysis
Bottom-Up Knowledge
Consolidation
Decomposed Subckts +
Hierarchical Structure
Agentic Retrieval (Online Phase) Topology Knowledge Database
Q: "Improve DNL/INL linearity"
8-bit Fully Differential SAR ADC
Fig. 2: Overall workflow of the HeaRT framework. Left: offline hierarchical circuit reasoning tree construction from SPICE netlists via top-down
graph-guided subcircuit identification, hierarchical abstraction, and bottom-up knowledge consolidation. Middle: query-conditioned traversal and
agentic retrieval for AMS design tasks. Right: topology knowledge database with metric-wise ranking for objective-driven topology retrieval.
do not explicitly exploit this structured reasoning paradigm, leading to
inconsistencies, hallucinations [46], and limited reasoning traceability
that conflict with the precision, determinism, and verifiability required
in circuit design [36]. Moreover, recent LLM-aided efforts [14, 16–
18, 20–26] remain task-specific, often deploying LLMs within nar-
rowly scoped optimization roles rather than leveraging their broader
reasoning capabilities. EEsizer [47] employs Chain-of-Thought (CoT)
reasoning [37] with simulation-in-the-loop, operating primarily at the
parameter level for sizing rather than structural circuit interpretation.
Similarly, [48] uses metric-driven identification of local transistor
patterns (e.g., diff pairs, current mirrors), in contrast to the topology-
centric hierarchical abstraction used by human designers, where struc-
tural reasoning precedes consideration of performance metrics. These
limitations highlight the need for structured reasoning frameworks
that align LLM capabilities with human design reasoning.
B. Structural Interpretation of AMS Circuits
Flattened SPICE netlists represent circuits as a single connectivity
graph where devices share arbitrary interconnections, obscuring the
structural organization designers rely on to interpret circuit func-
tionality. Structural abstraction is therefore necessary for modular
analysis and optimization, thereby better unlocking the capabilities
of LLM in AMS design. Existing subcircuit identification methods
rely on hand-crafted rule-based [49], template library-based [50],
learning-based [51, 52], or class-specific few-shot approaches [53].
These methods remain constrained by template coverage or training
data distribution and primarily focus on identifying local transistor
patterns. In this paper, we use the term subcircuits to denote mod-
ular functional blocks corresponding to the block-level abstractions
commonly used by analog designers. Examples include gain stages
in amplifiers; resistor dividers, error amplifiers, and pass devices in
LDO regulators [27]. However, given the diversity of analog circuit
architectures, automated structural partitioning to recover high-level
functional abstractions remains challenging [53], since electrically
coupled fragments hinder independent functional interpretation and
disrupt feedback loop relationships. When correctly identified, such
abstractions preserve subcircuit-level functional integrity and expose
the modular structure required for hierarchical circuit reasoning and
systematic interpretation of circuit blocks and their feedback interac-
tions [28, 29]. To quantitatively evaluate the correctness of subcircuit
decomposition and feedback loop identification, we introduce two
F1-score-based metrics:
F1,subcircuits =
2|S ∩ S∗
|
|S| + |S∗|
, F1,loops =
2|L ∩ L∗
|
|L| + |L∗|
, (1)
where S∗
and S denote the ground-truth and identified subcircuit sets,
respectively, and L∗
and L denote the ground-truth and identified
feedback loop sets. A subcircuit match requires agreement in device
membership and internal net connectivity, whereas a feedback loop
match requires agreement in participating devices (or hierarchical
blocks), loop polarity, and functional role.
C. Performance Figure of Merit (FoM)
Given m normalized performance metrics {fr(x)}m
r=1 defined over
their acceptable bound intervals Sbound
r = [Lr, Hr], with fr(x) ≥ 0
indicating satisfaction, we define the performance Figure of Merit
(FoM) as follows:
FoM(x) =
m
X
r=1
wr min(1, max(0, fr(x))) , (2)
where wr is the weighting factor. The max(·) operator truncates
negative values arising from constraint violation, while the min(·)
operator bounds each metric contribution, preventing any single term
from dominating the FoM beyond full satisfaction.
D. Design Effort Reuse Under Specification Shift
To complement the performance-centric FoM, it is also essential to
assess how effectively an optimization process preserves prior design
effort in scenarios where incremental design adaptation is preferred
over full re-optimization. In practical AMS workflows, re-optimizing
solely for performance under updated specifications, without con-
textual awareness, may inadvertently discard valuable prior efforts,
such as robustness across PVT corners, device mismatch, and layout
parasitics. Hence, analogous to parameter drift induced by fine-tuning
in machine learning literature [30, 54, 55], we introduce a design
effort reuse score, denoted by Reffort, to quantify deviation of design
variables under specification shifts within a fixed architecture relative
to a previously validated reference design, defined as:
Reffort = 1/M
M
X
j=1
h
1 − min

1, log10(xj/x
(0)
j )
 i
. (3)
Here, M is the number of design variables, and x
(0)
j and xj
represent the initial and modified values of the j-th design variable,
respectively. The min(·, 1) cap prevents single large deviations from
dominating the score. Reffort ∈ [0, 1], where 1 indicates complete
parameter reuse within the fixed architecture and smaller values
reflect substantial deviation from the validated reference design.
III. THE HEART FRAMEWORK
Current LLM-based approaches for AMS design fall short of the
explainability and structured reasoning exhibited by human design
flows, and their opaque, inconsistent reasoning limits trustworthiness

## Page 3

Algorithm 1 : Line #2
Residual Component(s)
Algorithm 1 : Line #12
Algorithm 1 : Line #7
DC Connected Components (SDC constraints)
Minimum Hard Connectivity Constraints
{M1, M2, M3, M4, M5} {M6, M7}
{M8}
bias_root bias_root
bias_root
Algorithm 1 : Line #20
Algorithm 1 : Line #6
Fig. 3: An illustrative example of HeaRT’s circuit decomposition.
and practical deployment. Inspired by the cognitive process through
which human designers analyze, abstract, and interpret circuits, we
propose HeaRT as a hierarchical reasoning framework that leverages
human-inspired design philosophy to achieve transparent, context-
consistent circuit understanding for downstream agentic tasks. As
depicted in Fig. 2 (left), HeaRT begins with graph-guided LLM-
assisted subcircuit extraction, followed by hierarchical organization
and bottom-up architectural knowledge consolidation. By combining
top-down circuit decomposition with bottom-up reasoning, the frame-
work mirrors the hierarchical abstraction principles used in circuit
design. Local metadata provides informative cues, while access to
the full netlist as global context maintains semantically consistent
multi-level circuit reasoning. The framework of HeaRT operates
through two complementary phases: an offline, one-time knowledge-
building stage that constructs the hierarchical circuit reasoning tree
(Section III-A), and an online, real-time agentic retrieval stage
that leverages standard LLM reasoning capabilities while grounding
responses in this tree, enabling structured, context-grounded inference
beyond single-shot full-circuit reasoning (Section III-B). The fol-
lowing subsections describe each phase in further detail. To promote
reproducibility and further research, we will release HeaRT as an
open-source framework upon acceptance.
A. Hierarchical Circuit Reasoning Tree Construction
This phase constitutes a one-time offline process that constructs a
hierarchical circuit reasoning tree directly from raw SPICE netlists,
grounding subsequent agentic retrieval inference tasks.
1) Graph-Guided Reasoning for Tree Construction: We first trans-
form the SPICE-level circuit netlist into a device–net bipartite graph
G = (VD, VN , E), where device nodes VD represents circuit com-
ponents (e.g., transistors, resistors, capacitors), and net nodes VN
correspond to unique net names. Each net node is labeled by its
type as SUPPLY_PORT, SIGNAL_PORT, or INTERNAL_NET, and
each edge e ∈ E encodes a unique device-terminal-net connection.
For decomposition purposes, MOSFETs are simplified to three-
terminal devices (D, G, S), omitting body connections. From this
graph, we extract DC-connected components that capture conductive
paths between supply rails, exposing the circuit’s biasing structure
commonly used by designers to reason about functional organization.
These components serve as structural cues for LLM-assisted circuit
decomposition into the functional subcircuits defined in Section II-B,
with metadata generated via Chain-of-Thought (CoT) reasoning [37]
and in-context examples [56] as described in Algorithm 1. Fig. 3 il-
lustrates an example of this decomposition. The extracted subcircuits,
Sdecomp, together with their metadata (e.g., subcircuit_ID,
role_hint, architectural context, port annotations, etc.), are pro-
vided to a second LLM stage to construct the hierarchical reasoning
tree for subsequent bottom-up knowledge consolidation.
2) Bottom-Up Knowledge Consolidation: In this step, the LLM
performs bottom-up hierarchical reasoning from leaf to root, ag-
gregating subcircuit-level reasoning and propagating structural and
functional insights upward through the hierarchy. Local and global
metadata guide contextual consistency, enabling the model to interpret
each subcircuit’s role relative to its parent and the broader circuit.
At each node, the LLM identifies local feedback loops among its
immediate children and assigns unique loop IDs where applicable. As
reasoning progresses upward, annotated netlists with corresponding
signal and supply ports are synthesized and stored at each node,
forming consistent representations across abstraction levels. Finally,
all inferred functional relations are consolidated into the hierarchical
circuit reasoning tree for subsequent retrieval during inference.
Algorithm 1 LLM-Assisted Functional Subcircuit Decomposition
Require: Raw SPICE netlist N
Ensure: Subcircuits Sdecomp enriched with contextual metadata
1: function EXTRACTDCSUBCIRCUITS(G)
2: GDC ← remove all non–DC-conductive terminals (MOS gates, capacitor nodes),
then drop isolated nodes
3: Initialize RVDD ← ∅, RGND ← ∅, DDCAlive ← ∅
4: Perform multi-source BFS on GDC from {VDD, GND}
5: RVDD, RGND ← terminals reachable from VDD and GND, respectively
6: DDC alive ← { d = (n1, n2) ∈ D | (n1 ∈ RVDD ∧ n2 ∈ RGND) ∨
(n2 ∈ RVDD ∧ n1 ∈ RGND) }
7: SDC ← connected components formed by DC-alive devices
8: return SDC
9: end function
10: function PROCESSRESIDUALANDACCOMPONENTS(G, SDC )
11: Gres ← G \ SDC
12: SAC ← connected components of Gres
13: return SAC
14: end function
15: G(VD, VN , E) ← NETLISTTOBIPARTITE(N ) ▷ device–net bipartite graph
construction
16: Annotate each n ∈ VN as SUPPLY_RAIL, SIGNAL_PORT, or INTERNAL_NET
▷ initial net-level metadata
17: SDC ← EXTRACTDCSUBCIRCUITS(G) ▷ DC Connectivity Cues
18: SAC ← PROCESSRESIDUALANDACCOMPONENTS(G, SDC )
19: P ← BUILDPROMPT(N, SDC , SAC , CoT, In-Context Exemplars)
20: OLLM ← LLM(P) ▷ LLM Reasoning for Circuit Decomposition
21: Sdecomp ← Parse(OLLM)
22: return Sdecomp
B. Agentic Retrieval
This phase performs online, real-time agentic traversal and re-
trieval, grounding LLM reasoning in the hierarchical circuit rea-
soning tree built during the offline setup. The retrieval operations
are bounded by the tree depth, reducing exploration overhead and
supporting consistent, context-grounded inference. To reduce token
usage while maintaining reasoning accuracy, we employ context engi-
neering strategies [45], including context compression, local–global
context coordination, and selective context reuse.
1) Topology Knowledge Database Management: As illustrated
in Fig. 2 (right), we maintain a topology knowledge database
in which circuits are organized by functional category. Each row
corresponds to a circuit topology T represented by its normalized
circuit graph, while columns correspond to the family-specific per-
formance metrics defined in our circuit-family metric schema (e.g., 3-
dB bandwidth, DC gain, and phase margin for OPAMPs; temperature
coefficient and startup margin for bandgap references, and so on).
Each topology entry stores a metric-wise rank ri(T ) that reflects

## Page 4

TABLE I: Comparison of LLM-Based Methods for Analog and Mixed-Signal (AMS) Design.
Framework Multiple Hierarchical Performance Objective- Targeted Opt. Commercial
Types1
Circuit Reasoning Driven Topology Opt.2
Tasks PDKs3
Artisan [16] – – • TR & Sizing –
ADO-LLM [17] • – – Sizing Only •
AnalogCoder [18] • – – TG –
Atelier [20] – – ◦ TG†
& Sizing •
AnalogXPert [21] – – – TG‡
–
AnalogGenie/Lite [22, 23] • – – TG & Sizing –
LEDRO [24] – – – Sizing Only –
MenTeR [59] • – • TG & Sizing –
ADO-KT [48] • – – Sizing Only –
AnaFlow [14] – – – Sizing Only –
AnalogCoder-Pro [19] • – ◦ TG & Sizing –
HeaRT (This Work) • • • TR & Context-Aware Sizing •
1
Demonstrates support across multiple AMS circuit families, 2
Supports target performance-conditioned topology optimization, 3
Demonstrates evaluation using commercial
foundry PDKs, †
Topology generation via retrieval followed by template-based mutation rather than synthesizing from scratch, ‡
Retrieved subcircuits from circuit library
combined to generate larger composite architectures, TR: Topology Retrieval/Selection, TG: Topology Generation, • Supported, ◦ Partial, – Not Supported.
its relative performance capability along metric i. The database is
initialized and updated using LLM reasoning grounded in analog
design literature, including representative circuits and design insights
from ISSCC and JSSC publications, as well as standard textbooks
[27, 57], in conjunction with the circuit-family metric schema. For
each topology, the LLM assigns metric-wise rankings along with
concise justification cues reflecting expected performance trade-
offs. During updates, the LLM conditions on the current repository
context to provide additional grounding and ensure consistency across
entries. New topologies are inserted into existing metric-wise ranking
lists via ordered-list insertion, while newly introduced performance
metrics are added as new ranking columns and populated accordingly.
New circuit families follow the same ranking procedure under their
respective metric schemas. To support fast lookup, each topology
entry additionally stores a Weisfeiler–Lehman (WL) graph fingerprint
[58] as an auxiliary hash index. This database supports task-specific
rank-based retrieval during downstream applications.
2) Reasoning-Guided and Query-Conditioned Traversal: In this
stage, the agent performs a query-conditioned traversal of the hi-
erarchical circuit reasoning tree. A single LLM pass first assigns
query-conditioned relevance weights to every edge using CoT and
in-context reasoning. Before traversal, each node v is evaluated using
the following branch-admission criterion, which determines whether
its children should be enqueued during BFS:
Branch Cut(v) =







max
u∈Children(v)
w(v,u) < τstop
or max
u∈Children(v)
w(v,u) − min
u∈Children(v)
< ϵ







,
(4)
where w(v,u) is the query-conditioned relevance of child u, τstop
halts expansion when all children are weak, and ϵ stops expansion
when all children are similarly strong (i.e., no dominant direction).
If condition 4 is satisfied at a node v, all of its children are
marked as non-admissible and therefore never enqueued into the
BFS queue. A BFS from the root then enqueues children only
when the Branch Cut criterion is not met. Traversal thus naturally
terminates either at suppressed nodes or true leaves. The result-
ing root-to-terminal traversal paths define query-conditioned priority
search regions, guiding the framework toward the most influential
design variables for downstream tasks such as sizing optimization or
objective-driven retrieval while maintaining full circuit context.
3) Objective-Driven Retrieval for Topology Optimization: Given
a topology optimization task, HeaRT performs query-conditioned
BFS traversal to identify candidate subcircuits for objective-driven
retrieval. For each identified subcircuit, the set of performance
objectives S is determined through LLM reasoning over the user’s
system-level natural language query and the corresponding circuit
family-specific performance metric schema defined in the knowledge
database, with the remaining metrics forming the constraint set U.
Through an agentic function call, HeaRT locates the corresponding
database entries of the identified topologies using their stored WL
hash indices and computes an aggregate rank-based Figure of Merit
(FoMrank) as a fast ranking proxy, defined as:
min
T ∈ϕ(U)
FoMrank(T ) =
X
i∈S
wi × ri(T ), (5)
where wi denotes the metric weight and ϕ(U) the feasible search
region constrained by U:
ϕ(U) = {T | |rj(T ) − r̄j| ≤ ϵ, ∀j ∈ U} , (6)
where r̄j denotes the prior rank on the j-th metric and ϵ is a
small rank tolerance (set to 3 in our experiments). If a prior rank is
unavailable, the constraint is ignored, yielding the full search space
along that metric dimension. The top-k topologies that minimize
this score represent promising candidates for the given design goal,
enabling objective-driven retrieval for topology selection.
IV. EXPERIMENTAL RESULTS
A. Experimental Setup
We evaluate HeaRT’s structural circuit decomposition and feedback
loop identification on a curated repository of 40 AMS circuits
spanning diverse types and complexity levels (Fig. 4), provided as
flattened SPICE netlists. Circuits are categorized into three tiers
based on transistor count (excluding digital logic components) as a
rough proxy for structural complexity: Simple (< 20 transistors),
Medium (20–40 transistors), and Hard (> 40 transistors). The
repository will be open-sourced upon acceptance. For this task, we
evaluate HeaRT across a diverse set of LLM backbones spanning
open-source LLMs (LLaMA-3.3-70B-Instruct [60, 61], DeepSeek-
V3.2 [62]) and proprietary frontier models (Gemini-2.5 Pro [63],
GPT-5 [64], and Claude-4.6-Sonnet [65]), using the F1-based metrics
defined in Eq. 1. To illustrate HeaRT’s downstream applicability
beyond structural interpretation, we further evaluate two system-
level circuits: (i) a supply-insensitive relaxation oscillator and (ii)
an analog front-end (AFE), each under its corresponding design-
adaptation scenario (Fig. 5(a)-(b)). We use Eq. (2) to compute the
performance FoM for both scenarios and Eq. (3) to evaluate design
effort reuse for the fixed-architecture case in Scenario 1 (Fig. 5(a)).
Note that since Reffort measures preservation of prior design effort
rather than performance improvement, it is evaluated separately
from the FoM. All experiments use a 200-simulation budget. The
one-time offline tree construction phase employs Claude-4.6-Sonnet,
while downstream interaction through the Interface Agent uses lighter
LLM variants (e.g., GPT-4o [66]) to reduce LLM inference overhead
during online queries. Schematic simulations are performed using

## Page 5

TABLE II: Evaluation of HeaRT across different LLM backbones.
LLM Backbone Approach
F1,subcircuits F1,loops
S M H S M H
LLaMA-3.3-
70B-Instruct†
HeaRT 0.724 0.513 0.336 0.561 0.362 0.191
Few-shot (Text) 0.571 0.452 0.167 0.396 0.153 0.135
Few-shot (Image) – – – – – –
Gemini-2.5 Pro
HeaRT 0.979 0.952 0.914 0.948 0.891 0.876
Few-shot (Text) 0.697 0.641 0.532 0.569 0.471 0.414
Few-shot (Image) 0.681 0.612 0.469 0.591 0.554 0.397
DeepSeek-V3.2
HeaRT 0.993 0.974 0.946 0.975 0.917 0.892
Few-shot (Text) 0.729 0.672 0.539 0.688 0.621 0.497
Few-shot (Image) 0.718 0.637 0.493 0.702 0.626 0.464
GPT-5
HeaRT 0.993 0.973 0.949 0.972 0.931 0.906
Few-shot (Text) 0.722 0.669 0.578 0.691 0.638 0.523
Few-shot (Image) 0.725 0.663 0.541 0.704 0.659 0.478
Claude-4.6-
Sonnet
HeaRT 0.997 0.988 0.955 0.984 0.962 0.937
Few-shot (Text) 0.762 0.709 0.684 0.741 0.675 0.614
Few-shot (Image) 0.759 0.693 0.657 0.744 0.698 0.581
S, M, and H denote Simple (<20), Medium (20–40), and Hard (>40) circuit
complexity tiers in our benchmark repository, measured by transistor count excluding
digital logic. “Text” denotes SPICE netlist input, while “Image” denotes schematic
image input. †
Vision capability not supported for this model.
the TSMC180nm process technology with SPICE in Cadence Analog
Design Environment (ADE) on a 4-core 3.3 GHz CPU Linux system.
B. Evaluation of Structural Decomposition and Loop Identification
Table I summarizes the capabilities and limitations of existing
LLM-based AMS design frameworks. We evaluate HeaRT’s ability to
perform structural decomposition and feedback loop identification on
flattened netlists, assessing how effectively it recovers circuit structure
from raw connectivity, thereby enabling the hierarchical circuit rea-
soning paradigm described in Section II-B. Using the metrics in Eq. 1,
the predicted subcircuits and feedback loops are manually verified
against expert-provided ground-truth annotations from experienced
analog circuit designers for the circuits in the benchmark repository
(Fig. 4). The benchmark repository, including the expert-annotated
ground-truth labels used for evaluation, will be publicly released upon
acceptance to facilitate reproducibility and further research. Table II
reports F1 scores averaged over 10 independent trials across multiple
LLM backbones, comparing HeaRT against two CoT-based few-shot
prompting baselines using netlists and schematic images as inputs,
respectively. As shown, performance degrades from Simple to Hard
circuits across both tasks as structural complexity increases. While
schematic image-based reasoning yields marginal gains over netlists
on simpler circuits, particularly for loop identification where visual
feedback paths are clearer, schematic images introduce connectivity
ambiguity from wire crossings and overlapping nets on larger circuits,
degrading loop and subcircuit recovery. Across all circuit complexity
tiers, HeaRT consistently improves the F1 scores relative to the base-
lines, achieving ≥ 13.5% improvement in F1,subcircuits and ≥ 37.8%
Fig. 4: Statistical summary of our curated dataset repository.
in F1,loops, demonstrating its ability to infer meaningful structure
from otherwise unstructured netlist representations. These gains stem
from the combination of our graph-guided partitioning heuristics and
LLM reasoning. As expected, models with stronger reasoning and
instruction-following capabilities yield improved performance, with
the best results obtained using Claude-4.6-Sonnet.
C. Integration within Optimization Loops
The experimental evaluation comprises two incremental design-
adaptation scenarios. Scenario 1 investigates query-conditioned
priority-guided sizing optimization. We integrate HeaRT with the
TuRBO optimizer [67], which has been widely adopted for analog
sizing [68], and benchmark it against standalone TuRBO and the
LLM-based sizing framework LEDRO [24]. Scenario 2 then evalu-
ates HeaRT’s optimizer-agnostic and context-aware decision-making
capability by integrating it with several established optimization
algorithms commonly used for analog sizing, namely Differential
Evolution (DE) [69], the RL-inspired DNN-Opt [7], and the multi-
agent RL-based MA-RL [10, 13], alongside TuRBO. These base-
lines were chosen to represent different categories of optimization
approaches. In the experiments that follow, we limit HeaRT to a one-
time invocation at initialization. For a fair convergence comparison
under consistent initialization conditions, all optimizers are initialized
from the same design point. In incremental design adaptation tasks, a
uniform starting point reflects practical workflows where an existing
design is refined rather than redesigned from scratch. A detailed
analysis of HeaRT’s token usage across the offline and online phases
versus circuit complexity tiers is provided in the Appendix A (Fig. 8).
1) Scenario 1: Performance-Shift–Driven Priority-Guided Sizing
Optimization: As shown in Fig. 5(a), this design adaptation scenario
targets a 10% reduction in frequency error in a supply-insensitive
relaxation oscillator while maintaining oscillation frequency, relaxing
the IQ tradeoff, and keeping all other key specifications approxi-
mately unchanged. During inference-time query-conditioned traversal
on the hierarchical circuit reasoning tree, HeaRT accurately identi-
fies and prioritizes the comparator front-end design variables. The
corresponding traversal path and its reasoning trace are illustrated in
Fig. 6(a). The TuRBO optimizer-based sizing procedure is invoked,
with HeaRT-derived priority variables P defining the initial bounds:
boundsinit[P] = full range, boundsinit[¬P] = nominals. This
priority-guided optimization accelerates convergence, achieving an
FoM of ∼1.93× and a Reffort of ∼ 3.6× compared to the standalone
TuRBO (Fig. 7(a)-(b)). Table III summarizes the comparative results
for Scenario 1 in terms of FoM, convergence speed, and real-
time token usage. Unlike approaches that repeatedly invoke LLM
reasoning within the optimization loop, HeaRT performs a single
real-time invocation to prioritize the search space.
2) Scenario 2: Knowledge-Based, Performance-Driven Retrieval-
Guided Topology Reconfiguration and Sizing: This scenario targets
lower input-referred noise (higher SNR) and reduced area under
relaxed gain and offset constraints in an analog front-end (Fig. 5(b)).
As shown in Fig. 6(b), HeaRT identifies the first gain stage as
the dominant noise contributor, with the associated reasoning trace
shown alongside. Its rank-based retrieval, guided by S = {IRN
TABLE III: Scenario 1: HeaRT+TuRBO vs. baselines.
Method FoM
#Sims to
Converge
#Tokens
(Real-Time)
TuRBO [67] 320.71 516 N/A
LEDRO [24] 395.4 354 75,212
HeaRT+TuRBO 618.97 165 7,194
GPT-4o used for HeaRT online phase and LEDRO. All results averaged
over 10 independent trials.

## Page 6

Scenario 1 …
For the given sized netlist and performance metrics, reduce the frequency error by at least 10% without
changing the output frequency. Allow up to a 5% trade-off in IQ if necessary, while keeping all other
performance metrics and the circuit topology unchanged.
(a)
Initial Performance Metrics:
DC Gain = 19.9879 dB, Unity_Gain_Frequency = 110 KHz, IQ = 10.7983 uA, DC_Offset = 120 uV, 3dB_Bandwidth = 6.923 KHz, CMRR = 76.82,
PSRR = 84.47, IRN_Density@1KHz = 283.89 nV/√Hz, Phase_Margin = 80.4°, Area = 1.76 µm²
Initial Performance Metrics:
Output Frequency = 407.34 KHz, RMS Frequency Error = 2.1 KHz, IQ = 21 uA, PSRR = 64 dB, Line Regulation = 0.08%/V,
DC Supply Sensitivity = 6 ppm/mV, Area = 1.98 µm²
SS
SS SS
SS
SS SS
SS SS
SS
SS
Scenario 2 …
For the given sized netlist and performance metrics, improve the SNR and area, with up to 10% allowable trade-off in DC gain
and input-referred offset. Keep all other performance metrics unchanged. Topology adaptation may be performed if beneficial.
(b)
Fig. 5: Schematics of (a) Relaxation oscillator and (b) Analog front-end circuit with their corresponding design scenarios.
Fig. 6: Query-conditioned traversal and associated reasoning for (a)
Scenario 1 and (b) Scenario 2.
TABLE IV: Scenario 2: Impact of HeaRT across different optimizers.
Optimizer Eval. Metric Baseline
HeaRT Optimized
Sizing
Topo+
Sizing
TuRBO [67]
FoM 240.21 413.78 555
#Sims to Converge 401 37 43
DE [69]
FoM 230.21 381.82 551
#Sims to Converge 479 63 68
DNN-Opt [7]
FoM 269.73 467.93 556.6
#Sims to Converge 286 31 38
MA-RL [10, 13]
FoM 293.07 463.42 556.3
#Sims to Converge 314 28 39
density, Area}, selects an inverter-based topology as a higher-ranked
alternative and generates sized netlists for both designs, thereby
overcoming architectural bottlenecks inherent in conventional sizing-
only workflows. Fig. 7(c) shows the resulting FoM profiles rela-
tive to standalone TuRBO, while Table IV summarizes the impact
across different baseline optimizers. As shown, HeaRT achieves an
FoM improvement of ≥ 58% through sizing-only optimization, and
≥ 89% through its retrieval-guided topology reconfiguration across
all standalone baselines, while also achieving significantly faster
convergence. Table V reports the Spearman rank-order correlation
coefficient (ρs) between LLM-derived and human expert rankings for
10 representative topologies per family in OPAMPs and comparators,
comparing Base (few-shot prompting) and +Context (ISSCC/JSSC-
grounded) settings to assess the reliability of LLM-assigned rankings
during topology knowledge database population (Section III-B1).
V. CONCLUSION
In this paper, we propose HeaRT, a novel hierarchical circuit rea-
soning tree-based agentic framework for AMS design optimization.
HeaRT combines top-down graph-guided subcircuit identification
TABLE V: Expert–LLM Rank Agreement (Spearman’s ρs Correlation).
Circuit
Family
GPT-4o
Gemini-
2.5 Pro
DeepSeek-
V3.2
GPT-5
Claude-4.6-
Sonnet
OPAMP 0.78 / 0.86 0.84 / 0.90 0.87 / 0.92 0.88 / 0.92 0.93 / 0.95
Comparator 0.74 / 0.81 0.79 / 0.88 0.89 / 0.92 0.86 / 0.91 0.92 / 0.93
Results averaged across corresponding family-specific metrics over 10 trials. Each cell
reports Base / +Context agreement. Base: few-shot prompting without external
grounding; +Context: with knowledge retrieved from ISSCC/JSSC corpus.
(a)
(b)
(c)
Fig. 7: Performance profiles for (a) FoM and (b) Reffort in Scenario
1 and (c) FoM in Scenario 2, plotted against simulation count. Shaded
regions indicate variation across 10 independent runs.
and hierarchical abstraction with bottom-up knowledge consolida-
tion, enabling structured, context-aware reasoning for AMS design
tasks. The framework allows natural-language instructions to steer
real-time retrieval while remaining grounded in the hierarchical
reasoning structure. Our experiments validate HeaRT’s ability to
recover meaningful structure from flattened netlists across a diverse
AMS benchmark. We further demonstrate its effectiveness in practi-
cal design-adaptation scenarios involving both topology and sizing
optimization, two critical steps in AMS design. Future directions
include enriching the reasoning tree with simulation metadata and
exploring applications such as layout-constraint extraction. Overall,
HeaRT represents a step toward context-aware, practical AMS design
automation. Upon acceptance, the HeaRT framework and the bench-
mark repository, including the expert-annotated ground-truth labels
used for evaluation, will be publicly released as open source.

## Page 7

REFERENCES
[1] Wenlong Lyu et al. Batch Bayesian optimization via multi-objective
acquisition ensemble for automated analog circuit design. In
Proc. ICML, 2018.
[2] Shuhan Zhang, Fan Yang, et al. An Efficient Batch-Constrained
Bayesian Optimization Approach for Analog Circuit Synthesis via
Multiobjective Acquisition Ensemble. IEEE TCAD, 2022.
[3] Xilu Wang et al. Recent Advances in Bayesian Optimization. ACM
Computing Surveys, 55(13s):1–36, 2023.
[4] Keertana Settaluri et al. AutoCkt: Deep Reinforcement Learning of
Analog Circuit Designs. In Proc. DATE, pages 490–495, 2020.
[5] Hanrui Wang et al. GCN-RL Circuit Designer: Transferable
Transistor Sizing with Graph Neural Networks and Reinforcement
Learning. In Proc. DAC, 2020.
[6] Yaguang Li, Yishuang Lin, Meghna Madhusudan, et al. A Circuit
Attention Network-Based Actor-Critic Learning Approach to Robust
Analog Transistor Sizing. In Proc. MLCAD, 2021.
[7] Ahmet F. Budak et al. DNN-Opt: An RL inspired optimization for
analog circuit sizing using deep neural networks. In Proc. DAC,
2021.
[8] Ahmet F Budak et al. APOSTLE: Asynchronously parallel op-
timization for sizing analog transistors using DNN learning. In
Proc. ASPDAC, pages 70–75, 2023.
[9] Minjeong Choi et al. Reinforcement Learning-based Analog Circuit
Optimizer using gm/ID for Sizing. In Proc. DAC, 2023.
[10] Jinxin Zhang et al. Automated Design of Complex Analog Circuits
with Multiagent based Reinforcement Learning. In Proc. DAC,
2023.
[11] Youngmin Oh et al. CRONuS: Circuit Rapid Optimization with
Neural Simulator. In Proc. DATE, pages 1–6, 2024.
[12] Handa Sun et al. EVDMARL: Efficient Value Decomposition-based
Multi-Agent Reinforcement Learning with Domain-Randomization
for Complex Analog Circuit Design Migration. In Proc. DAC, 2024.
[13] Jiarui Bao et al. Multiagent Based Reinforcement Learning(MA-
RL): An Automated Designer for Complex Analog Circuits. IEEE
TCAD, 2024.
[14] Mohsen Ahmadzadeh, Kaichang Chen, et al. (Invited Paper)
AnaFlow: LLM-based Workflow for Reasoning-Driven Explainable
and Sample-Efficient Analog Circuit Sizing. In Proc. ICCAD, 2025.
[15] Georges GE Gielen. A Brief 20-Year History and Future Perspec-
tives on Sizing and Layout Synthesis of Analog/RF ICs. IEEE
Design & Test, 42(6):63–74, 2025.
[16] Zihao Chen et al. Artisan: Automated Operational Amplifier Design
via Domain-specific Large Language Model. In Proc. DAC, 2024.
[17] Yuxuan Yin, Yu Wang, Boxun Xu, and Peng Li. ADO-LLM: Analog
Design Bayesian Optimization with In-Context Learning of Large
Language Models. In Proc. ICCAD, 2024.
[18] Yao Lai et al. AnalogCoder: Analog Circuit Design via Training-
Free Code Generation. In Proc. AAAI, 2025.
[19] Yao Lai, Souradip Poddar, Sungyoung Lee, et al. AnalogCoder-Pro:
Unifying Analog Circuit Generation and Optimization via Multi-
modal LLMs. IEEE TCAD, 2026.
[20] Jinyi Shen, Zihao Chen, Ji Zhuang, et al. Atelier: An Automated
Analog Circuit Design Framework via Multiple Large Language
Model-Based Agents. IEEE TCAD, 2025.
[21] Haoyi Zhang et al. AnalogXpert: Automating Analog Topology
Synthesis by Incorporating Circuit Design Expertise into Large
Language Models. In Proc. ISEDA, 2025.
[22] Jian Gao et al. AnalogGenie: A Generative Engine for Automatic
Discovery of Analog Circuit Topologies. In Proc. ICLR, 2025.
[23] Jian Gao, Weidong Cao, and Xuan Zhang. AnalogGenie-Lite:
Enhancing Scalability and Precision in Circuit Topology Discovery
through Lightweight Graph Modeling. In Proc. ICML, 2025.
[24] Dimple Vijay Kochar et al. LEDRO: LLM-Enhanced Design Space
Reduction and Optimization for Analog Circuits. In Proc. ICLAD,
2025.
[25] Haohang Xu, Chengjie Liu, Qihang Wang, et al. Image2Net:
Datasets, Benchmark and Hybrid Framework to Convert Analog
Circuit Diagrams into Netlists. In Proc. ISEDA, 2025.
[26] Weiyu Chen et al. AnalogTester: A Large Language Model-Based
Framework for Automatic Testbench Generation in Analog Circuit
Design. In Proc. ISEDA, 2025.
[27] Behzad Razavi. Design of Analog CMOS Integrated Circuits.
McGraw-Hill, Inc., 2000.
[28] Ken Kundert. Principles of Top-Down MixedSignal Design. The De-
signer’s Guide Community http://www. designers-guide. org, 2003.
[29] Georges GE Gielen. Modeling and analysis techniques for system-
level architectural design of telecom front-ends. IEEE Transactions
on Microwave Theory and Techniques, 50(1):360–368, 2002.
[30] James Kirkpatrick et al. Overcoming catastrophic forgetting in
neural networks. Proceedings of the national academy of sciences,
2017.
[31] Ahmet F Budak et al. Joint Optimization of Sizing and Layout for
AMS Designs: Challenges and Opportunities. In Proc. ISPD, 2023.
[32] Ahmet F Budak et al. Practical Layout-Aware Analog/Mixed-
Signal Design Automation with Bayesian Neural Networks. In
Proc. ICCAD, 2023.
[33] Wei Shi et al. RobustAnalog: Fast Variation-Aware Analog Circuit
Design Via Multi-task RL. In Proc. MLCAD, 2022.
[34] Jian Gao et al. RoSE: Robust Analog Circuit Parameter Op-
timization with Sampling-Efficient Reinforcement Learning. In
Proc. DAC, 2023.
[35] Weidong Cao et al. RoSE-Opt: Robust and Efficient Analog Circuit
Parameter Optimization With Knowledge-Infused Reinforcement
Learning. IEEE TCAD, 2024.
[36] Chengjie Liu and Jun Yang. IC + AI: a no-human-in-loop design
paradigm? Science China Information Sciences, 2025.
[37] Jason Wei et al. Chain-of-Thought Prompting Elicits Reasoning in
Large Language Models. Proc. NeurIPS, 35:24824–24837, 2022.
[38] Lei Wang et al. A survey on large language model based au-
tonomous agents. Frontiers of Computer Science, 2024.
[39] Caigao Jiang et al. LLMOPT: Learning to Define and Solve
General Optimization Problems from Scratch. arXiv preprint
arXiv:2410.13213, 2024.
[40] Zhiheng Xi et al. The rise and potential of large language model
based agents: A survey. Science China Information Sciences, 2025.
[41] Junde Wu, Jiayuan Zhu, Yuyuan Liu, et al. Agentic Reasoning:
A Streamlined Framework for Enhancing LLM Reasoning with
Agentic Tools. arXiv preprint arXiv:2502.04644, 2025.
[42] Fengli Xu, Qianyue Hao, Zefang Zong, et al. Towards Large
Reasoning Models: A Survey of Reinforced Reasoning with Large
Language Models . arXiv preprint arXiv:2501.09686, 2025.
[43] Qiguang Chen, Libo Qin, Jinhao Liu, Dengyun Peng, et al. Towards
Reasoning Era: A Survey of Long Chain-of-Thought for Reasoning
Large Language Models. arXiv preprint arXiv:2503.09567, 2025.
[44] Daya Guo, Dejian Yang, Haowei Zhang, Junxiao Song, Zhang, et al.
DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via
Reinforcement Learning. arXiv preprint arXiv:2501.12948, 2025.
[45] Philipp Schmid. The New Skill in AI is Not Prompting, It’s Context
Engineering, 2025.
[46] Lei Huang et al. A Survey on Hallucination in Large Language
Models: Principles, Taxonomy, Challenges, and Open Questions.
ACM Transactions on Information Systems, 2025.
[47] Chang Liu and Danial Chitnis. EEsizer: LLM-Based AI Agent for
Sizing of Analog and Mixed Signal Circuit. IEEE Transactions on
Circuits and Systems I: Regular Papers, 2025.
[48] Karthik Somayaji NS and Peng Li. AI Analog Circuit Design
Agents: On Knowledge Extraction and Transfer with Knowledge
Graphs. In 2025 IEEE/ACM International Conference On Computer
Aided Design (ICCAD), pages 1–9. IEEE, 2025.
[49] Inga Abel, Maximilian Neuner, and Helmut Graeb. A functional
block decomposition method for automatic op-amp design. Inte-
gration, 85:108–120, 2022.
[50] Bohao Li, Shizhang Wang, Tinghuan Chen, Qi Sun, and Cheng

## Page 8

Zhuo. Efficient subgraph matching framework for fast subcircuit
identification. In Proceedings of the 2024 ACM/IEEE International
Symposium on Machine Learning for CAD, pages 1–7, 2024.
[51] Kishor Kunal, Tonmoy Dhar, Meghna Madhusudan, Jitesh Poojary,
Arvind K Sharma, Wenbin Xu, Steven M Burns, Jiang Hu, Ramesh
Harjani, and Sachin S Sapatnekar. GANA: Graph Convolutional
Network Based Automated Netlist Annotation for Analog Circuits.
In DATE, pages 55–60, 2020.
[52] Kishor Kunal, Tonmoy Dhar, Meghna Madhusudan, Jitesh Poojary,
Arvind K Sharma, Wenbin Xu, Steven M Burns, Jiang Hu, Ramesh
Harjani, and Sachin S Sapatnekar. GNN-Based Hierarchical Anno-
tation for Analog Circuits. IEEE Transactions on Computer-Aided
Design of Integrated Circuits and Systems, 42(9):2801–2814, 2023.
[53] Phuoc Pham, Arun Venkitaraman, Chia-Yu Hsieh, Andrea Bonetti,
Stefan Uhlich, Markus Leibl, Simon Hofmann, Eisaku Ohbuch,
Lorenzo Servadei, Ulf Schlichtmann, et al. GENIE-ASI: Generative
Instruction and Executable Code for Analog Subcircuit Identifica-
tion. In 2025 ACM/IEEE 7th Symposium on Machine Learning for
CAD (MLCAD), pages 1–21. IEEE, 2025.
[54] Sanghwan Kim, Lorenzo Noci, Antonio Orvieto, and Thomas
Hofmann. Achieving a Better Stability-Plasticity Trade-off via
Auxiliary Networks in Continual Learning. In Proceedings of the
IEEE/CVF conference on computer vision and pattern recognition,
pages 11930–11939, 2023.
[55] Friedemann Zenke, Ben Poole, and Surya Ganguli. Continual
Learning Through Synaptic Intelligence. In International conference
on machine learning, pages 3987–3995. Pmlr, 2017.
[56] Tom Brown et al. Language Models are Few-Shot Learners.
Proc. NeurIPS, 2020.
[57] Paul R Gray, Paul J Hurst, Stephen H Lewis, and Robert G Meyer.
Analysis and design of analog integrated circuits. John Wiley &
Sons, 2024.
[58] Nino Shervashidze, Pascal Schweitzer, Erik Jan Van Leeuwen, Kurt
Mehlhorn, and Karsten M Borgwardt. Weisfeiler-Lehman Graph
Kernels. Journal of Machine Learning Research, 12(9), 2011.
[59] Pin-Han Chen, Yu-Sheng Lin, Wei-Cheng Lee, Tin-Yu Leu, Po-
Hsiang Hsu, Anjana Dissanayake, Sungjin Oh, and Chinq-Shiun
Chiu. Menter: A fully-automated multi-agent workflow for end-to-
end rf/analog circuits netlist design. In Proc. ICLAD, pages 124–
132. IEEE, 2025.
[60] Aaron Grattafiori et al. The Llama 3 Herd of Models. arXiv preprint
arXiv:2407.21783, 2024.
[61] Meta AI. Llama-3.3-70B-Instruct Model Card, 2024. URL: https:
//huggingface.co/meta-llama/Llama-3.3-70B-Instruct.
[62] DeepSeek-AI. DeepSeek-V3.2: Pushing the Frontier of Open Large
Language Models. arXiv preprint arXiv:2512.02556, 2025.
[63] Google DeepMind. Gemini 2.5: Pushing the Frontier with Advanced
Reasoning, Multimodality, Long Context, and Next Generation
Agentic Capabilities. arXiv preprint arXiv:2507.06261, 2025.
[64] Aaditya Singh, Adam Fry, Adam Perelman, Adam Tart, Adi Ganesh,
Ahmed El-Kishky, Aidan McLaughlin, Aiden Low, AJ Ostrow,
Akhila Ananthram, et al. OpenAI GPT-5 System Card. arXiv
preprint arXiv:2601.03267, 2025.
[65] Anthropic. Introducing Claude Sonnet 4.6, 2026. URL:
https://www.anthropic.com/news/claude-sonnet-4-6.
[66] Aaron Hurst, Adam Lerer, Adam P Goucher, Adam Perelman,
Aditya Ramesh, Aidan Clark, AJ Ostrow, Akila Welihinda, Alan
Hayes, Alec Radford, et al. GPT-4o System Card. arXiv preprint
arXiv:2410.21276, 2024.
[67] David Eriksson, Michael Pearce, Jacob Gardner, Ryan D Turner,
and Matthias Poloczek. Scalable Global Optimization via Local
Bayesian Optimization. Advances in neural information processing
systems, 32, 2019.
[68] Konstantinos Touloupas, Nikos Chouridis, and Paul P Sotiriadis.
Local Bayesian Optimization For Analog Circuit Sizing. In 2021
58th ACM/IEEE design automation conference (DAC), pages 1237–
1242. IEEE, 2021.
[69] Cătălin Vişan, Octavian Pascu, Marius Stănescu, Elena-Diana
Şandru, Cristian Diaconu, Andi Buzo, Georg Pelz, and Horia Cucu.
Automated circuit sizing with multi-objective optimization based
on differential evolution and Bayesian inference. Knowledge-Based
Systems, 258:109987, 2022.
[70] Jerry Liu et al. LlamaIndex, 2023. URL: https://github.com/
run-llama/llama index.
APPENDIX A
TOKEN COST ANALYSIS
To quantify HeaRT’s overall token cost, we report the total
token usage across circuits in our benchmark repository at different
stages of the offline and online phases (Fig. 8), grouped by circuit
complexity tiers. As shown, token usage generally increases with
circuit complexity across stages, with the increase most pronounced
in the top-down graph-guided subcircuit decomposition and tree con-
struction stage. The subsequent bottom-up knowledge consolidation
stage shows comparatively smaller variation across complexity tiers.
In contrast, online reasoning stages exhibit noticeably lower variation.
This two-phase design amortizes the higher offline construction cost
across downstream applications, reducing per-task inference overhead
compared to single-shot prompting, particularly for more complex
circuits, while preserving circuit-level reasoning fidelity.
Simple
Medium
Hard
16000
14000
12000
10000
8000
6000
4000
Graph-Guided Reasoning
for Tree Construction
Bottom-Up Knowledge
Consolidation
Reasoning Traversal
Total
Token
Usage
Hierarchical Circuit Reasoning Tree Construction
(Offline, one-time: Claude-4.6-Sonnet)
Downstream Task Reasoning
(Online inference: GPT-4o)
Fig. 8: Token usage across different stages of the HeaRT workflow.
For the experiments reported in Table III (Scenario 1) and Table IV
(Scenario 2), the total real-time token usage was approximately 7,194
and 10,216 tokens, respectively.
APPENDIX B
ADDITIONAL IMPLEMENTATION DETAILS
This appendix details the external grounding mechanism for topol-
ogy knowledge database updates (Fig. 2, Section III-B1). We use
LlamaIndex [70] to index a curated corpus of ISSCC and JSSC
publications, alongside standard analog design textbooks [27, 57],
with each document parsed into structured chunks capturing (i) the
document title, (ii) circuit family, (iii) architectural description, and
(iv) reported performance characteristics and trade-offs. Retrieval
queries are constructed from the circuit family label and an LLM-
generated architectural description of the candidate topology. The
retrieved context, together with the circuit-family metric schema and
current repository state, is provided as input to the LLM to guide
metric-wise ranking updates. Future extensions could incorporate
schematic image-to-graph conversion for richer structural retrieval
from analog circuit literature.

## Page 9

Furthermore, while Table V evaluates the reliability of LLM
ranking in a bulk setting (all topologies are ranked simultaneously),
it does not capture the incremental insertion scenario described in
Section III-B1, where a new topology is slotted into an existing
repository whose entries already carry metric-wise rankings and
concise justification cues reflecting expected performance trade-offs.
To evaluate this, we adopt a leave-one-out protocol over the 10
representative topologies per family used in Table V: in each trial, one
topology is held out and inserted into a repository of the remaining 9,
with ρs measured against human expert ground truth. This is repeated
across all 10 topologies, with results averaged over 10 independent
LLM runs. Table VI reports ρs under two conditions: without and
with conditioning on the existing repository state, both grounded in
ISSCC/JSSC literature.
TABLE VI: Incremental Topology Insertion: LLM–Expert Ranking Cor-
relation (Spearman’s ρs, mean ± std).
Condition
Claude-4.6-Sonnet GPT-5
OPAMP Comparator OPAMP Comparator
w/o Repository
Context
0.74 ± 0.09 0.70 ± 0.11 0.72 ± 0.10 0.69 ± 0.11
w/ Repository
Context
0.92 ± 0.04 0.90 ± 0.03 0.89 ± 0.06 0.85 ± 0.04
Both conditions are grounded in ISSCC/JSSC literature. Results averaged over 10
independent trials under a leave-one-out protocol over 10 topologies per family.
APPENDIX C
EFFICACY OF FoMrank FOR TOPOLOGY RETRIEVAL
To further validate the effectiveness of our top-k FoMrank-based
topology retrieval strategy, we evaluate the performance profiles
across candidates at different rank positions for Scenario 2, under
identical optimization settings using TuRBO. All results are averaged
over 10 independent trials. As shown in Fig. 9, higher-ranked
candidates generally achieve higher FoM, supporting the utility of our
FoMrank-based retrieval as an effective proxy for topology selection.
Figure
of
Merit
(FoM)
Number of Simulations
Fig. 9: Performance FoM Profiles Across Retrieved Topology Ranks.

