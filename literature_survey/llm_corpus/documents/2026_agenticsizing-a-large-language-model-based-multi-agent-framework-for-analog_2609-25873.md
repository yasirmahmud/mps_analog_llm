---
title: "AgenticSizing: A Large Language Model-based Multi-Agent Framework for Analog Circuit Sizing"
authors: "Hao, Yijia; Verma, Pratibha; Guo, Dongxu; Sestito, Cristian; O'Boyle, Michael; Bouganis, Christos-Savvas; Prodromakis, Themis"
date: "2026/09/22"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2609.25873"
pdf: "papers/2026_agenticsizing-a-large-language-model-based-multi-agent-framework-for-analog_2609-25873.pdf"
sha256: "c1e5daa0afcc4ba3463f77651ef71bc96fb25664c039cdb828524cef632d9268"
pages_extracted: 13
---

# AgenticSizing: A Large Language Model-based Multi-Agent Framework for Analog Circuit Sizing

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

1
AgenticSizing: A Large Language Model-based
Multi-Agent Framework for Analog Circuit Sizing
Yijia Hao , Pratibha Verma , Member, IEEE, Dongxu Guo ,
Cristian Sestito , Member, IEEE, Michael O’Boyle , Fellow, IEEE,
Christos-Savvas Bouganis , Senior Member, IEEE, and Themis Prodromakis , Senior Member, IEEE
Abstract—Analog circuit sizing remains a challenging and
time-consuming task due to the large design space, strong per-
formance trade-offs, and increasing circuit complexity in scaled
technologies. Although recent large language model (LLM)-
based methods show promise in improving sample efficiency and
interpretability, existing approaches often lack explicit circuit-
topology understanding and are mainly evaluated on relatively
simple analog building blocks. This paper presents a multi-
agent LLM-based framework for complex analog circuit sizing.
The proposed framework first analyzes the circuit topology and
decomposes the netlist into functional blocks and substructures.
It also extracts lightweight design knowledge for reuse. Based
on the extracted topology and knowledge, a planner coordinates
multiple role-specialized sizing agents to update design variables
and achieve global performance specifications. This workflow
mimics the collaborative process of an expert analog design team
and provides a structured, interpretable, and simulation-driven
optimization procedure. The framework was validated on eight
circuits, with the largest design containing up to 55 transistors
and 60 sizing variables. Notably, for the LDO benchmark,
the proposed method achieved a 60% success rate with an
average of 83 iterations, where classical optimizers failed to
find feasible solutions. Further, ablation studies demonstrate that
topology understanding, design-knowledge infusion, and agent
specialization provide complementary benefits. The source code
is available to support reproducibility.
Index Terms—Agentic AI, Large Language Models (LLMs),
Analog Design Automation, Analog Circuit Sizing, LangGraph.
I. INTRODUCTION
ANALOG circuits work as an essential interface between
digital processing systems and the physical world. De-
spite their critical importance, the transistor sizing process
remains time-consuming due to the vast design space and
the inherent trade-offs among performance metrics, such as
gain, bandwidth, power consumption, noise, and linearity.
Consequently, the design process still heavily relies on human
This work was supported by the Engineering and Physical Sciences
Research Council (EPSRC) AI Hub for Productive Research and Innovation
in eLectronics (APRIL) under Grant No. EP/Y029763/1, and by the Royal
Academy of Engineering (RAEng) Chair in Emerging Technologies under
Grant No. CiET1819/2/93. (Corresponding author: Cristian Sestito).
Y. Hao, D. Guo, C. Sestito and T. Prodromakis are with the Centre for
Electronics Frontiers, Institute for Integrated Micro and Nano Systems, School
of Engineering, The University of Edinburgh, UK (email: csestito@ed.ac.uk).
P. Verma contribued to this work while affiliated with the Centre for
Electronics Frontiers, Institute for Integrated Micro and Nano Systems,
School of Engineering, The University of Edinburgh, UK. Current affiliation:
Department of Electrical Engineering, Indian Institute of Technology Indore,
India.
M. O’Boyle is with the School of Informatics, The University of Edinburgh,
UK.
C.S. Bouganis is with the Department of Electrical and Electronic Engi-
neering, Imperial College London, UK.
intuition and experience. Furthermore, as technology nodes
continue to scale down, the complexity of analog circuit design
increases correspondingly, posing significant challenges for
designers to simultaneously meet aggressive time-to-market
schedules and stringent performance specifications.
To improve the efficiency of the analog design flow, elec-
tronic design automation (EDA) techniques have evolved
alongside conventional manual design methodologies [1]–[3].
Early approaches were primarily optimization-based methods,
such as evolutionary algorithms [4]–[7], which incorporate
AI into the transistor-sizing process. More recently, surro-
gate model-based approaches have been explored, in which
machine learning models are trained online to predict circuit
performance and guide the search process. For example,
Bayesian optimization (BO) with Gaussian process regres-
sion (GPR) has been widely investigated [8]–[10], as it can
reduce the number of expensive circuit simulations required
by evolutionary algorithms. However, its efficiency degrades
as the design space expands and the number of performance
trade-offs increases. Reinforcement learning (RL) [11]–[13]
has also been explored as an alternative framework, in which
neural networks learn sizing policies by interacting with the
simulator and receiving rewards based on performance met-
rics, with the goal of maximizing the expected cumulative
reward. Although such policies may be transferable across
related design tasks, RL methods generally suffer from low
sample efficiency [14]. Overall, both surrogate model-based
optimization and RL-based frameworks require a large number
of simulations to obtain high-quality designs, particularly for
complex analog circuits. In addition, their limited transparency
remains a significant challenge, as the underlying models are
often treated as black boxes, making it difficult for designers
to assess the reliability of the obtained solutions.
More recently, large language model (LLM)-based ap-
proaches [15], [16] have attracted wide interest due to their
few-shot learning, in-context reasoning capabilities and the
prior knowledge embedded in pre-trained cloud-based models.
These methods exhibit advantages in terms of sample effi-
ciency and interpretability over alternative frameworks. How-
ever, LLMs still face two challenges that hinder their applica-
tion in analog circuit sizing. First, the circuit understanding of
existing LLM-based approaches remains limited. The circuit
topology is typically provided in the form of a netlist without
annotations, which makes it difficult to determine whether
the LLM truly comprehends the underlying sizing problem.
Consequently, the optimization process often resorts to random
or heuristic exploration. Furthermore, the general knowledge
arXiv:2609.25873v1
[cs.AI]
22
Sep
2026

## Page 2

2
embedded in these models lacks the specific domain expertise
required for analog circuit design, thereby constraining their
design efficiency. Second, existing LLM-based sizing frame-
works have been primarily evaluated on basic analog building
blocks. However, practical analog systems can readily exceed
50 transistors [17], [18]. Liu et al. reported a 60% success rate
for a medium-complexity amplifier with 20 transistors [19].
This result indicates that direct LLM-based sizing may have
limited scalability to highly integrated circuits.
To address these challenges, this work introduces Agen-
ticSizing, a multi-agent optimization framework for analog
circuit sizing. Inspired by existing analog design workflows,
the framework decomposes complex circuits into functional
blocks and coordinates role-specific agents to meet global
performance objectives. It integrates circuit topology infor-
mation and reusable design knowledge to guide optimization,
improving both sizing efficiency and interpretability. Rather
than using the LLM as a standalone optimizer, AgenticSizing
investigates how structured agent coordination and circuit-
level context can support effective analog sizing across both
commercial EDA and open-source environments. We evaluate
the proposed framework on eight circuits spanning different
levels of complexity and compare its performance with three
baseline methods. For the most complex circuit, the proposed
framework achieves a 60% success rate with an average of
89.7 simulation evaluations, whereas all baseline methods fail
to find a feasible design within the six-hour time limit. The
key contributions of this work are as follows:
• We propose an LLM-based decomposition tool that
leverages circuit topology information to decom-
pose complex analog circuits into functional blocks
and substructures. The software is open-source at
https://github.com/aprilaihub/agentic-analog-sizing.
• The framework leverages an LLM-based knowledge ex-
traction tool that captures performance trade-offs, func-
tional role–performance relationships, and design pa-
rameter–performance trade-offs, enabling the extracted
knowledge to be reused across different circuits.
• We introduce a novel multi-agent LLM workflow that
emulates the collaborative process of expert analog design
teams by incorporating structured design knowledge and
topological understanding.
• We demonstrate the superior performance of the proposed
framework over competitive optimization approaches on
medium- to high-complexity benchmarks and quantify
the individual impact of each contribution.
The remainder of the paper is organized as follows. Section
II reviews LLM-related work in analog circuit sizing. Section
III illustrates the methodology and components. Section IV
presents experimental results from low- to medium-complexity
circuits, extending to a high-complexity case, along with an
ablation study. Section V concludes the paper.
II. RELATED WORK IN LLM-BASED ANALOG CIRCUIT
SIZING
Analog circuit sizing aims to determine transistor dimen-
sions and component values that satisfy target performance
specifications. Several LLM-based multi-agent frameworks
have recently been proposed as alternatives or complements
to conventional optimization routines. AnaFlow [21] uses four
agents, namely explainer, DC reviewer, specs reviewer, and
sizer, to support reasoning-driven and explainable sizing with
iterative simulation feedback. Liu et al. [25] adopt a two-agent
hierarchy consisting of an expert for high-level reasoning and
an employee for data extraction, focusing on extracting sizing
relationships from circuit documentation. AmpAgent [22] uses
three sequential agents: a literature analysis agent retrieves
formulas and stability conditions, a mathematics reasoning
agent derives stage-level design targets, and a device sizing
agent uses conventional optimizers to size multi-stage ampli-
fiers. LLM-USO [24] is a hybrid agentic framework in which
an LLM agent and a BO algorithm propose design points
through a shared simulation buffer. It uses a working LLM
generates transferable knowledge summaries, while a critique
LLM refines them for reuse across circuits. Circuit-Agent and
EEsizer [16], [26] also use multi-agent architectures, including
selector, extractor, calculator, composer, and validator roles,
with an emphasis on integrating LLM reasoning with external
EDA tools for circuit evaluation and verification.
As summarized in Table I, existing LLM-enabled analog
design frameworks commonly support closed-loop simulation,
but differ substantially in reasoning capability, benchmark
scale, and deployment support. Methodologically, multi-agent
coordination is supported by only a subset of frameworks,
and explicit structural reasoning and knowledge bases are
rarely incorporated. In terms of benchmark scale, the reported
breadth ranges from 2 to 24 circuits, and the largest reported
circuit in prior work contains at most 20 transistors.
These studies show that LLMs can provide useful reasoning
interfaces for analog sizing and can be coupled with simulation
tools to support iterative refinement. The existing literature,
however, remains at an early stage. Prior work has primarily
explored how to organize LLM agents, prompt them with
circuit information, and connect them to external EDA tools.
Less attention has been given to circuit representation dur-
ing optimization and to the retention and transfer of design
knowledge across sizing tasks. Furthermore, the applicability
of LLMs to complex analog circuits remains largely unex-
plored. This limitation is significant because practical analog
circuits typically comprise multiple interconnected devices and
circuit blocks, resulting in a high-dimensional and strongly
coupled design space. Effective optimization therefore requires
representations that capture circuit topology, device roles, and
interconnections, together with mechanisms for retaining and
reusing design knowledge to guide the search process and
reduce redundant simulations.
Motivated by these observations, this work investigates
agentic LLMs for simulation-in-the-loop analog circuit sizing
under complex design settings. Compared with prior frame-
works, AgenticSizing evaluates eight circuits spanning three
representative analog circuit categories: operational transcon-
ductance amplifiers (OTAs), bandgap references (BGRs), and
low-dropout regulators (LDOs), with up to 55 transistors and
60 design variables.

## Page 3

3
TABLE I
COMPARISON OF REPRESENTATIVE LLM-ENABLED ANALOG DESIGN FRAMEWORKS.
Framework
Benchmark LLM-Enabled Methodology Deployment
Benchmark
Breadth
Largest Reported
Circuit
Max.
Variables
Closed-Loop
Simulation
Structural
Reasoning
Knowledge
Memory
Multi-Agent
Coordination
Adaptive
Search
Commercial
EDA
Open-
Source
Cross-
Node
EEsizer [16] 6 circuits 20 transistors N/R ✓ ✗ ✗ ✗ ✗ ✗ ✓ ✓
AnalogCoder [20] 24 circuits N/R N/R ✓ ✗ ✗ ✗ ✗ ✗ ✓ ✗
AnaFlow [21] 2 circuits 16 transistors N/R ✓ ✓ ✗ ✓ ✗ ✗ ✗ ✗
AmpAgent [22] 7 circuits N/R N/R ✓ ✗ ✓ ✓ ✓ ✓ ✗ ✓
LEDRO [23] 22 circuits 19 transistors 20 variables ✓ ✗ ✗ ✗ ✓ ✓ ✓ ✓
LLM-USO [24] 24 circuits N/R N/R ✓ ✗ ✓ ✓ ✓ ✗ ✗ ✗
This work
8 circuits
(3 categories)
55 transistors 60 variables ✓ ✓ ✓ ✓ ✗ ✓ ✓ ✗
✓: explicitly supported; ✗: unsupported or not reported; N/R: quantitative benchmark information not reported.
Inputs
• Circuit Topology / Netlist
• Design Variables
• Target Specifications
• Maximum Runtime
Two-Step Decomposition
1 Functional Role Tags
2 Substructure Tags
Phase 1
Structured Representation
Main Signal: DifferentialPair (M1-M2)
Main Signal: CurrentMirror/ActiveLoad (M3-M4)
Bias Network: CurrentMirror (M0, M0b)
Bias Network: TailCurrentSource (M0b)
Spectre Simulation
• DC
• Transient
• AC
• ......
Initial & Iterative Sizing
High-Level Planner
• Ranks roles / selects target
• Evaluates violations
• Dispatches role tasks
• Checks constraints
Role-Specific Sizer
• Initializes/updates owned variables
• Respects bounds and constraints
• Predicts performance effects
• Performance
• Role
• Predictions
• Variables
• Instructions
Knowledge Base (4 Views)
• Performance coupling
dominant performance trade-offs
((P1, P2), (P1, P3), (P2, P3), ...)
• Critical parameters
substructure/parameter -> performance
S1: ((X1, P1), (X1, P2), ...)
• Role–performance influence
role -> performance relationships
((R1, P1), (R1, P2), (R2, P1), ...)
• Transferable heuristics
reusable design strategies
evidence-backed guidance for future sizing tasks
Outputs
• Final design X*
• Design summary
Committed simulation record
e.g., Gain = 80dB, PM = 60
Initial / bounded design update
e.g., W1 = 1um, L1 = 60nm, NF1 = 2
`
Specifications:
Gain >= 80 dB,
CMRR >= 60dB,
PM >= 60o
,
......
M5
M0c
M3
M0
M4
M1 M2
M0b
vip vin
vdd
vo
Phase 2 & 3 Phase 4
Rz Cc
Fig. 1. Four-phase agenticSizing framework for automated analog IC sizing. The proposed method integrates LLM decomposer (phase 1), LLM planner and
sizer (phase 2 and phase 3), and LLM knowledge extractor (phase 4).
III. METHODOLOGY
In this work, we adopt agentic LLM principles [27], [28]
for analog circuit sizing by coordinating role-specific agents
through structured state transitions and tool interactions. Since
no LLM is specifically developed for analog circuit design,
and training such a model would require substantial domain-
specific data, ChatGPT-5.2 is used as the underlying LLM
agent, following prior LLM-based EDA research [15], [16].
A. The AgenticSizing Workflow
The AgenticSizing workflow, illustrated in Fig. 1, is a multi-
agent framework requiring only the circuit netlist, design-
variable bounds, target specifications, simulator interface, and
runtime budget. Circuit-specific prompt content is generated
automatically from topology tagging, simulation history, and
knowledge-base retrieval, allowing the same agent templates
to be reused without circuit-specific prompt engineering. The
workflow partitions the circuit into functional roles, gener-
ates an initial design candidate, iteratively refines it through
planner–sizer interactions, and extracts reusable knowledge
from the simulation results. The following subsections de-
scribe these stages in detail.
1) Two-step decomposition: The first stage uses a decom-
poser agent to convert the transistor-level circuit netlist into a
structured representation for use by subsequent agents. Details
of the decomposition are presented in Section III-B. The
output of this stage defines the ownership of each design
variable and provides the structural vocabulary used by the
planner, the sizers, and the knowledge base.
2) Initial Sizing: After decomposition, the framework con-
structs an initial design point. A high-level planner initiates the
sizing process by coordinating the role-specific sizing agents
according to the design specifications, decomposed circuit
structure, and available prior knowledge. The sizers indepen-
dently assign initial values to their role-scoped design variables
while meeting the specified bounds and constraints. These
role-level assignments are then assembled into a complete
design vector and evaluated by the simulator. The resulting
performance vector is recorded as the initial simulation re-
sult, providing the starting point for the subsequent iterative
optimization process.
3) Iterative Sizing: The iterative sizing stage performs
closed-loop refinement of the initial design. At each iteration,
the current design is evaluated against the target specifications,
and the planner coordinates the selection of a role-specific
sizing action to address the remaining performance gaps. The
selected sizer then updates only its assigned design variables
within the specified bounds and submits the resulting candidate
design for simulation. The newly obtained performance vector
is appended to the simulation history and used to guide the
next iteration. This process alternates between planner-guided
design updates and simulator-based evaluation until the design
satisfies the target specifications or the optimization process

## Page 4

4
terminates.
4) Knowledge extraction: Knowledge extraction converts
empirical sizing experience into reusable symbolic knowl-
edge. After the iterative optimization completes, the LLM
knowledge extractor analyzes the relationship between design-
variable changes, circuit substructures, functional roles, and
performances. The extracted knowledge is then stored in four
complementary views. More details can be found in Section
III-D. This knowledge base is then reused by the planner
and sizers in subsequent optimization tasks. In this way,
the framework accumulates structured design experience that
improves future role selection, variable tuning, and trade-off
reasoning.
B. LLM decomposer
The AgenticSizing workflow uses four specialized agents.
The LLM decomposer agent translates transistor-level analog
netlists into a normalized symbolic representation of circuit
function and topology, serving as a bridge between low-
level device connectivity and high-level design reasoning.
The tool follows a two-stage LLM-assisted pipeline. In the
first stage, the decomposer agent partitions the circuit into
predefined high-level function roles, such as main gain stage,
bias network, feedback network, output stage, startup circuitry,
etc.. Each device is strictly assigned to one role. In the second
stage, the tool decomposes each functional role into canonical
circuit substructures drawn from a controlled ontology. Each
substructure is represented by a canonical label, base type,
modifiers, connectivity evidence, and associated design vari-
ables (Fig. 2). The ontology is defined once using common
analog primitives, including differential pairs, current mirrors,
resistor dividers, and cross-coupled pairs, while modifiers such
as cascode, folded, and tail provide further specialization. For
example, reference and tail current sources share the base
type current source but use the reference and tail modifiers,
respectively. Restricting the LLM to these predefined terms
normalizes synonymous descriptions and prevents unsupported
labels, thereby supporting consistent knowledge retrieval and
transfer across circuits. The ontology is extensible and requires
no circuit-specific modification for the evaluated benchmarks.
C. LLM Planner and Sizers
The sizing is performed by LLM planner and sizers. The
system follows a 1 + N planner-sizer pattern, where a single
planner agent performs global planning and coordination,
while multiple role-scoped sizer agents propose parameter
updates for the circuit roles identified by the decomposer tool.
The planner acts as a high-level decision agent. During
initialization, it ranks all available functional roles and pro-
duces an ordered execution plan, where each role receives
a priority, a set of focus metrics, and a worker instruction.
During iterative sizing, it evaluates the latest simulation results,
identifies unsatisfied specifications, and dispatches one role
sizer to optimize one target metric (Table II). Before prompt
construction, the knowledge base is filtered according to the
active specifications, such that only relevant role–performance
relations, performance tradeoffs, and design heuristics are
Functional-Role Partitioning Prompt
Context prompt
You are an analog circuit analysis assistant.
Task
• Partition the netlist into high-level functional roles.
• Focus on system purpose; include only roles that exist.
Typical functional roles
main signal path, bias network, feedback network
cmfb, compensation network, reference generation
output stage, ptat/ctat core, startup circuit
Rules
• Put each device in one role; preserve exact netlist names.
• Do not invent devices.
• Use “output stage” only for a clearly separate power-driving stage.
Input
Transistor-level netlist: {netlist}
(a)
Functional-Role Partition JSON
Main signal path
devices: [M1, M2, M3,
M4, M5]
Differential first stage and
common-source second stage.
Bias network
devices: [M0, M0b, M0c]
Compensation network
devices: [Rz, Cc]
Check: unique devices ✓ exact names ✓ no invented devices ✓
Output
{ "functional_roles": [
{ "name": "Main signal path",
"devices": ["M1", "M2", "M3", "M4", "M5"],
"description": "Two-stage signal path" },
{ "name": "Bias network",
"devices": ["M0", "M0b", "M0c"] },
{ "name": "Compensation network",
"devices": ["Rz", "Cc"] }
] }
(b)
Substructure-Tagging Prompt
Context prompt
A netlist and its Step 1 functional partition are provided.
Task
Identify substructures inside each role using the canonical ontology.
Canonical ontology
Base types
differential_pair, current_source, current_mirror
gain_stage, source_follower, resistor_divider
miller_compensation, capacitive_feedback_network
cross_coupled_pair, diode_connected_device
Modifiers
active_load, bias, cascode, common_gate, tail
common_source, folded, output, reference, testbench
Decision rules
• Match each type to its base_type and modifiers.
• Label by primary function and use the most specific valid label.
• Keep devices in their Step 1 role; require connectivity evidence.
• Record symbolic design variables, not parameter names.
Inputs
Netlist: {netlist}
Functional partition: {functional_partition_json}
(c)
Substructure-Tagging JSON
Output
Resolved structures
Main signal path
Differential pair [M1, M2]
Current mirror (active_load) [M3, M4]
Gain stage (common_source) [M5]
Bias network
Current mirror (bias) [M0, M0b, M0c]
Tail/output current sources [M0b, M0c]
Compensation network
Miller compensation [Rz, Cc]
{ "roles": [
{ "name": "Main signal path",
"substructures": [
{ "base_type": "differential_pair", ... },
{ "base_type": "current_mirror", ... },
{ "base_type": "gain_stage",
"modifiers": ["common_source"],
"devices": ["M5"] }] },
{ "name": "Bias network",
"substructures": [ ... ] },
{ "name": "Compensation network",
"substructures": [
{ "base_type": "miller_compensation",
"devices": ["Rz", "Cc"] }
] }
] }
(d)
Fig. 2. Overview of the two-step circuit decomposition process: (a) functional-
role partitioning prompt; (b) structured JSON representation of the identified
functional roles; (c) substructure-tagging prompt; and (d) structured JSON
representation of the constituent substructures.
TABLE II
PROMPT CONTEXT COMPONENTS FOR PLANNER AND SIZER AGENTS.
Agent Component Source Purpose
Planner
Design specifications YAML input Define optimization goals
Tagging results Tagging module Define role decomposition
knowledge base summary Knowledge base Provide heuristics and priors
Recent run history Simulation records Reflect search progress
Local state summary Planner memory Preserve decision context
Sizer
Role and target metric Planner output Define local objective
Role-scoped variables Tagging module Restrict editable variables
Variable bounds Config file Keep updates feasible
Knowledge base evidence Knowledge base Guide parameter updates
Improving examples Simulation records Ground candidate proposals
Local state summary Worker memory Preserve refinement context
provided to the planner. The planner also decides whether to
continue from the current design or revert to the best-known
design when recent evidence suggests an unproductive trajec-
tory. Each sizer performs local design-variable optimization
within its assigned role. It receives the planner instruction,
role substructures, variable bounds, recent simulation records,
improving-design examples, and knowledge filtered for the
selected target metric and functional role (Table II). Then the
sizers propose updates only for variables owned by its assigned
role. Fig. 3 illustrates representative prompts and responses of
the planner and sizer agents during iterative sizing.
An example illustrating the flow is shown in Fig. 4. The
example refers to a Miller-compensated OTA in which the
post-simulation phase margin is 45◦
, below the required spec-
ification of 60◦
, while the unity-gain bandwidth is 2 MHz,
exceeding its 1 MHz requirement. This means phase margin

## Page 5

5
Planner
<Context prompt>
Iterative planner for analog IC sizing.
Select one target metric and one role-scoped
worker.
Task
• Continue from the live design or revert to the best-known design.
• Provide a short goal-and-guardrail instruction for the target metric.
• Feasibility first: reduce any non-objective constraint violation.
• Use the strategic local summary as the primary run interpretation.
• Try the dominant role first; then consider other role–metric pairs.
Inputs
{case_name}, {settings_json}, {working_performances_json},
{unsatisfied_metrics_json}, {planner_local_summary_json}, {latest_record_json},
{recent_role_metric_selections_json}, {allowed_roles_json},
{role_perf_summary_block}, {perf_tradeoff_summary_block}, {critical_heuristics_block}
Output
{
"state_action": "continue_current|revert_best",
"target_metric": "...",
"selected_role_name": "...",
"worker_instruction": "...",
"selection_rationale": "...",
"recovery_rationale": "...",
"focus_kb_evidence": ["..."]
}
Sizer
<Context prompt>
Role-scoped iterative worker
for analog IC sizing.
Optimize one target using only the role’s variables.
Task
• Return the requested candidate count as strict JSON.
• Edit only role_substructures variables; remain within bounds.
• Prioritize KB parameter evidence; otherwise use the worker summary.
• Use oscillation alerts and target tradeoffs to avoid back-and-forth.
• Reopen a recovered spec only if net weighted violation improves.
Inputs
{case_name}, {role_name}, {target_metric}, {worker_instruction},
{working_state_json}, {role_substructures_json}, {variable_bounds_json},
{design_specs_json}, {latest_results_json}, {improving_best_json},
{worker_local_summary_json}, {kb_substruct_param_perf_summary_block}
Output
{
"role_name": "{role_name}",
"target_metric": "{target_metric}",
"candidates": [
{candidate_examples_block}
],
"rationale": "..."
}
Fig. 3. Overview of the AgenticSizing planner and sizer. Planner selects metric–role pairs based on the current design state, performance objectives, and
historical sizing records. Sizer generates role-specific sizing candidates using bounded variable updates, knowledge-base evidence, and historical worker
summaries.
HIGH-LEVEL PLANNING
KNOWLEDGE-GUIDED SIZING
1. Simulation Result
PM = 45 deg < 60 deg
UGB = 2 MHz > 1 MHz
2. High-Level Planner
Detect active PM violation
Select primary intervention
ROLE KB
PM → compensation
UGB → gain path
3. Constrained Delegation
Improve PM
Subject to preserving UGB
4. Compensation Sizer
Tune Miller network
Candidate variables: Cc, Rz
5. Candidate Proposal
Predicted: PM +15 deg
UGB -10%
6. Feasibility Check
UGB ≥ 1 MHz?
SUBSTRUCTURE KB
Miller compensation
Cc, Rz
Accept Candidate
Commit and re-simulate
Gain-Path Recovery
Restore UGB, then merge
with compensation update
Revised candidate
Fig. 4. The illustration of planner-sizer rounds for Miller-compensated OTA
phase margin recovery during iterative sizing.
is in active violation, whereas bandwidth is a satisfied but
potentially vulnerable constraint.
The high-level planner first interprets the violation using
the role-level knowledge base. It identifies the compensation
network as the primary functional role associated with phase-
margin recovery and the gain path as a secondary role that may
interact with bandwidth and stability. The planner therefore
formulates a role-scoped task for the compensation sizer:
improve phase margin while preserving sufficient unity-gain
bandwidth.
The compensation sizer then queries the substructure-level
knowledge base and identifies the Miller compensation ca-
pacitor Cc and the compensation zero-setting resistor Rz as
relevant tuning variables. Based on local knowledge relat-
ing compensation parameters to stability metrics, it proposes
increasing Cc and adjusting Rz. The predicted effect is an
increase in phase margin of approximately 15◦
, accompanied
by an estimated 10% reduction in unity-gain bandwidth.
The planner subsequently evaluates this proposal against
the global performance trade-off knowledge base. If the pre-
dicted bandwidth remains above its specification after the
compensation update, the planner accepts the proposal and
commits the candidate design to simulation. If the same update
were predicted to violate the bandwidth constraint, the planner
would instead reject the update or dispatch a subsequent task
to the gain-path sizer to recover bandwidth.
D. LLM knowledge extractor
In the proposed framework, the knowledge base enables
the planner to reuse prior sizing experience, reason about
trade-offs, and allocate tasks to suitable circuit roles. It also
constrains the sizers by providing role-relevant parameter-
performance knowledge, thereby reducing unnecessary explo-
ration before expensive circuit simulations are invoked.
The knowledge base organizes analog sizing experi-
ence into four structured categories: performance trade-

## Page 6

6
Performance–Performance Trade-Off Knowledge
Context
Settings · improving designs · structural tagging
Task
Extract sparse, high-value performance
tradeoff evidence.
Rules
• Target metrics only; exclude self-pairs.
• Keep sparse evidence; omit weak facts.
• Fixed relation/direction; exact topology; ≥3 facts.
Output
{"facts":[{
"type":"perf_perf_tradeoff",
"subject":"<metric>",
"relation":"trades_off_with",
"object":"<metric>",
"direction":"unspecified",
"confidence":0.0, "evidence_count":0,
"evidence_iters":[], "topology":"<case>"
}]}
(a)
Role–Performance Knowledge
Context
Common inputs · Step 1 and Step 2 JSON
Task
Extract role-level performance influence
grounded in prior steps.
Rules
• Stage 1 roles and target metrics only.
• Influence: dominant, significant, or weak.
• Exact topology; strict JSON; ≥4 facts.
Output
{"facts":[{
"type":"role_perf",
"functional_role":"<stage1_role>",
"metric":"<response>",
"influence":"dominant|significant|weak",
"confidence":0.0,
"evidence_count":0,
"evidence_iters":[], "topology":"<case>"}]}
(b)
Substructure–Parameter–Performance Knowledge
Context
Common inputs · Step 1 JSON
Task
Extract critical substructure–parameter–
performance facts.
Rules
• Stage 2 tags, des_vars, and target metrics only.
• Omit weak or speculative evidence.
• Direction unspecified; exact topology; ≥8 facts.
Output
{"facts":[{
"type":"substruct_param_perf",
"substructure":"<stage2_tag>",
"parameter":"<des_var>",
"metric":"<response>",
"direction":"unspecified",
"confidence":0.0, "evidence_count":0,
"evidence_iters":[],
"topology":"<case>"}]}
(c)
Design Heuristics
Context
Common inputs · all three extracted KB views
Task
Generate reusable, evidence-backed
analog design strategies.
Rules
• Mechanisms, sequencing, and guardrails only.
• No device/parameter names, values, or iterations.
• Target metrics only; produce 3–12 heuristics.
Output
{"heuristics":[{
"case_name":"<case>",
"condition":{
"target_metrics":["<response>"]},
"heuristic":"<mechanism and guardrail>",
"priority":"critical|high|medium|low",
"scope":"planner|worker|both"
}]}
(d)
Fig. 5. A pipeline that transforms design evidence into four knowledge modules: (a) performance–performance trade-off knowledge; (b) role–performance
knowledge; (c) substructure–parameter–performance knowledge; and (d) design heuristics.
offs, substructure–parameter–performance relations, role–
performance relations, and critical sizing heuristics. Their
predefined fields and relation types support automatic val-
idation and task-specific retrieval, allowing the planner to
obtain global optimization guidance and the sizers to retrieve
knowledge for local parameter updates. Formally, the knowl-
edge base abstracts simulation histories and structural circuit
annotations into symbolic facts as shown in Fig. 5.
The first category stores performance-performance trade-off
knowledge. It captures relationships among circuit metrics,
such as whether improving phase margin tends to reduce unity-
gain bandwidth or whether reducing noise tends to increase
power consumption. This view is mainly used by the planner to
reason about global specification interactions before assigning
a sizing task.
The second category stores role-performance knowledge. It
associates functional circuit roles, such as the compensation
network with the metrics they influence. The influence level
may be dominant, significant, or weak. This view supports
planner-level task allocation by helping the planner select the
most relevant role for a currently violated specification.
The third category stores substructure-parameter-
performance knowledge. It links circuit substructures
and design variables to performance metrics. For example, in
a Miller-compensated two-stage OTA, increasing Cc improves
the phase margin, while a separate fact records that increasing
Cc reduces the unity-gain bandwidth. This view is primarily
used by role-specific sizers to determine which local variables
should be modified for a given target metric.
The fourth category stores transferable heuristic knowledge.
It captures reusable, evidence-backed design strategies de-
rived from successful sizing trajectories, including general
mechanisms, adjustment sequences, and guardrails. Unlike
topology-specific knowledge, these heuristics abstract away
individual devices, parameter values, and iterations so they can
guide future sizing tasks. This view can support both planners
and role-specific sizers by providing general design guidance
across circuits.
TABLE III
SUMMARY OF BENCHMARKS. THE LOW-COMPLEXITY CIRCUITS ARE
USED TO CONSTRUCT THE INITIAL DESIGN KNOWLEDGE BASE.
Complexity Circuit # Variables # Transistors
Low
(initial knowledge
base source)
5T-OTA 7 6
Telescopic 17 12
Current mirror 10 10
Folded cascode 16 15
Medium
Two-stage FC 29 29
BGR 26 25
LDO 26 20
High LDO with BGR and FC 60 55
IV. EXPERIMENTS
A. Benchmarks
Table III summarizes the benchmark suite, organized by
circuit complexity. Four textbook circuits [29]–[31] with
fewer than 20 design variables are used to construct the
initial design knowledge base and are hence excluded from
evaluation. The whole AgenticSizing framework is evaluated
on medium-complexity analog blocks, including a two-stage
folded-cascode (FC) amplifier [32], bandgap reference (BGR)
[33], and low-dropout regulator (LDO) [34], and on a high-
complexity LDO integrating both the BGR and FC amplifier.
Existing LLM-based studies are usually validated only on
low- to medium-complexity circuits, leaving limited evidence
of their effectiveness on high-complexity designs. To address
this gap, we constructed a high-complexity benchmark by
integrating medium-complexity blocks into a complete LDO
system with a BGR (Fig. 6 (a)) and an FC error amplifier (Fig.
6 (b)). This benchmark contains more than 50 transistors and
involves over 50 design variables, shown in Fig. 6 (d).
B. Setup
Table IV summarizes the setup of the compared methods
used in our experiments. We compare the proposed method
with three representative optimization baselines: differential
evolution (DE), BO, and multi-agent RL (MA-RL). For DE,
we adopt the standard DE/rand/1/bin strategy [7]. For BO,

## Page 7

7
M12
M13
M10 M11
M15
M16
M17
M18
M19
MR1
M1 M2
M8 M9 M3
M4
M5
M6
M7
M14
cc2
cc1
cc3
VREF
VDD
SC · Startup circuit MS · Main signal path RG · Reference generation
MS · Differential pair
MS · Current mirror
MS · Tail current source
RG · Bias current source
RG · Cascode current source
RG · Cascode current source
RG · Diode-connected device
SC · Common-source stage
SC · Bias current source
SC · Diode-connected device
(a)
M11
M13
M4 M5
M3
M2
M0 M1
M6 M7
M10
M12
M14
M9
M8
MCAP MCAP
VIP VIN
VDD
23 M
M 24
M25 M26
M22
M15
M16
M18
M19
M20
M21
M17
VBP1
VBN
VBN2
VBP2
VCFMB
VREF
VOP
VBP2
VDD
VDD
VON
VCFMB
VBN
VBN2
VBP1
VBP2
VON
VOP
R
R
BN · Bias network MS · Main signal path CN · Compensation network OS · Output stage CM · Common-mode feedback
MS · Differential pair
MS · Folded cascode
BN · Tail source
BN · Cascode source
BN · Cascode source
BN · Bias source
BN · Diode-connected
BN · Diode-connected
BN · Diode-connected
BN · Diode-connected
CN · Miller compensation
OS · Common-source
OS · Bias source
CM · Resistor divider
CM · Common-source
CM · Reference source
CM · Bias source
CM · Bias source
(b)
M11
M12 M13
M14 M15
M16
M17 M18
Cm1
Cm2
M19 M20
M9
M21 M22
VIN
VOUT
VREF
VB
RF1
RF2
CF1
M10 M5 M6 M7
M3 M4
M1 M2
M8
MS · Main signal path BN · Bias network FB · Feedback network CN · Compensation network OS · Output stage
BN · Reference source
MS · Bias source
MS · Source followers
MS · Current mirror
MS · Common-source
MS · Diode-connected
MS · Current mirror
MS · Common-source
BN · Bias source
BN · Bias source
BN · Cascode source
MS · Current mirror
MS · Common-source stages
OS · Pass device
FB · Resistor divider
FB · Capacitive feedback
CN · Miller compensation
(c)
FC
+
-
-
RF1
RF2
CF1
COUT
IOUT
Re
VIN
VOUT
BGR
Power PMOS
(d)
Fig. 6. Topology and tagging results generated by the LLM. The LLM divides circuits into different functional blocks and identifies substructures (coloured)
within each block. (a) Bandgap reference circuit. (b) Two-stage folded cascode amplifier. (c) Low dropout regulator. (d) LDO with BGR and two-stage FC
amplifier.
we use a Gaussian process surrogate model with an RBF
kernel and the weighted expected improvement (wEI) ac-
quisition function [10]. For MA-RL, we use the MATD3
algorithm with an actor-critic network structure, where the
episode depth is set to 40 according to [35]. For MA-LLM,
we implement with LangGraph and use GPT 5.2 as the
backbone language model, with a temperature of 0.5 and a
maximum generation length of 5,000 tokens. These settings
are kept fixed across all benchmark circuits. The experiments
are conducted on a workstation equipped with AMD EPYC
9474F 48-Core Processor and 1.48T memory. The simulator
used is Cadence Spectre. Since the compared methods are
based on substantially different frameworks, we use wall-clock
time as the primary basis for comparison. Each experiment is
repeated five times to account for stochastic variability. We
compared the performance of our framework with respect to
the baselines in terms of success rate, average number of
simulations to achieve the best figure of merit (FoM) [10],
and wall-clock time. The wall-clock time includes the total
optimization time, the optimization time per iteration, and
the simulation time per iteration. For the integrated LDO,
an additional ablation study evaluates the contributions of
topology tagging, multi-agent specialization, and the design
knowledge base to optimization success and computational
efficiency. Each ablation configuration is evaluated using five
independent sizing attempts under the same time budget.
TABLE IV
SETUP OF COMPARED METHODS.
Method Hyper-parameter Value
DE Algorithm DE/rand/1/bin
BO
Kernel function RBF
Acquisition function wEI
MA-RL
Algorithm MATD3
Network Structure Actor–Critic
Episode depth 40
MA-LLM
LLM version GPT 5.2
Temperature 0.5
Max generation token length 5,000
C. Results on mid-complexity circuits
1) Results on BGR: The total runtime is set to 4 hours.
Table V compares the performance of different optimization
approaches on the BGR benchmark across five independent
attempts. Overall, DE achieves the highest reliability, satis-
fying all design specifications in all five attempts. However,
DE requires a relatively large number of simulations, with
an average of 936 simulations among successful runs. This
indicates that while DE is robust for this benchmark, it is
simulation-intensive. BO achieves valid designs in three out of
five attempts. Compared with DE, BO requires fewer simula-
tions for successful runs, with an average of 208 simulations.
However, its robustness is lower, as two attempts fail due to
excessive power consumption, and one of these also violates
the TC specification. This suggests that BO can be efficient

## Page 8

8
TABLE V
BGR: BEST PERFORMANCE SET FOUND BY DIFFERENT APPROACHES
(ACROSS FIVE ATTEMPTS). IF ANY SPECIFICATIONS ARE NOT MET WITHIN
FOUR HOURS, IT IS MARKED IN RED AND COUNTED AS A FAILURE.
Method Sim.
Power Vref PSR TC
(µW) (V) (dB) (ppm)
Target - ≤ 1 [0.5, 1.3] ≤ −40 ≤ 25
DE
726 0.460 0.62 −63.6 12.26
134 0.671 0.57 −72.5 13.29
1198 0.610 0.54 −60.3 22.74
1440 0.260 0.65 −46.7 15.66
1182 0.248 0.61 −46.2 2.56
BO
213 0.658 0.68 −82.1 18.24
fail 2.810 0.71 −76.6 23.42
265 0.417 0.53 −48.6 9.82
145 0.477 0.75 −53.6 0.23
fail 9.258 0.67 −72.5 113.32
MARL
fail 9.741 0.52 −8.7 261.81
fail 833.422 0.91 −10.5 34.54
fail 1.993 0.70 −69.6 26.99
fail 47.224 0.59 −62.3 52.17
fail 349.211 0.71 −12.6 64.42
AgenticSizing
33 0.308 0.58 −90.3 16.64
fail 0.231 0.53 −70.2 25.72
4 0.317 0.51 −80.7 9.24
45 0.106 0.61 −69.9 19.57
9 0.452 0.59 −73.8 19.89
Manual – 1.051 0.71 −89.0 9.69
for this circuit, but its performance is sensitive to the search
trajectory and surrogate model quality. MARL fails in all five
attempts. Although the output reference voltage remains within
the target range in all cases, MARL frequently violates the
power, PSR, and TC constraints. In particular, several runs
produce designs with power consumption far above the target
and insufficient PSR. These results indicate that the MA-
RL baseline struggles to handle the constrained analog sizing
problem for the BGR circuit within the given time budget.
AgenticSizing achieves four successful runs out of five and
requires substantially fewer simulations than both DE and BO.
Successful AgenticSizing runs find valid designs using only 4
to 45 simulations, with an average of 22.75 simulations. In
addition to its simulation efficiency, AgenticSizing also finds
high-quality designs, including the lowest power consumption
among all successful automated methods, 0.106 µW, and the
strongest PSR, −90.26 dB. The only failed AgenticSizing
run is close to feasibility, violating only the TC constraint
slightly, with 25.72 ppm compared with the 25 ppm target. It
is also important to note that the reduction in the number of
simulations is achieved at the expense of long LLM reasoning
time and significant token consumption, which should be
considered during practical application.
The manual design satisfies the voltage, PSR, and TC spec-
ifications but slightly exceeds the power constraint, with 1.051
µW compared with the 1 µW target. In contrast, AgenticSizing
identifies multiple feasible designs with lower power than the
manual baseline. These results demonstrate that AgenticSizing
provides a favourable balance between success rate, design
quality, and simulation efficiency on the BGR benchmark.
2) Results on two-stage FC: Table VI summarizes the re-
sults for the two-stage FC sizing task. DE, BO, and MARL fail
to satisfy all design specifications within the two-hour com-
putational budget in every trial. The most frequent violations
occur in the noise, rise time, UGB, and CMPM constraints.
In contrast, AgenticSizing achieves a 80% success rate. It
converges within only 11–29 circuit simulations while satis-
fying all target specifications. The sole failure case exhibits
a power value close to the target. The manual design meets
the specified requirements, except for the power requirement.
These results demonstrate that AgenticSizing is more reliable
and sample-efficient than both conventional optimization and
learning-based methods. Similar trends were also observed in
other amplifier sizing experiments, suggesting that the LLM-
based agent has high performance on analog amplifier sizing
tasks.
3) Results on LDO: Table VII summarizes the LDO sizing
results over five independent trials within a four-hour budget.
BO and AgenticSizing achieve comparable reliability with a
4/5 success rate, whereas DE and MARL succeed in only
1/5 trials. However, AgenticSizing demonstrates significantly
higher simulation efficiency, converging within 64–103 sim-
ulations (79 on average for successful runs), compared with
approximately 524 simulations for BO and 1131 simulations
for the successful DE trial.
The results reveal different optimization characteristics. DE
satisfies the regulation and PSRR constraints, but often fails
to achieve the target output voltage, indicating difficulty in
balancing the coupled specifications. BO provides robust con-
vergence but requires substantially more simulations, reflecting
the cost of black-box exploration. MARL exhibits poor robust-
ness. This behavior is likely attributed to the challenges of
RL in analog sizing, where sparse reward feedback and costly
environment interactions make it difficult to efficiently learn a
policy for highly constrained design spaces. In contrast, Agen-
ticSizing achieves high reliability with minimal simulation
overhead. This efficiency is attributed to the LLM’s in-context
learning capability and incorporation of prior domain-specific
knowledge, which enable more informed design exploration
and reduce unnecessary simulations. Its only failed trial vio-
lates the power constraint by 5.6% while satisfying all other
specifications. The manual design meets all specifications with
a power consumption of 81 µW, which is slightly higher than
the 33–74.5 µW achieved by AgenticSizing.
D. Full analysis on LDO with FC and BGR (with ablation
study)
Table VIII reports the full-LDO sizing results under a six-
hour budget. DE, BO, and MARL fail to satisfy all speci-
fications in every attempt, with failures primarily caused by
output-voltage, load-regulation, and PSRR violations. MARL
exhibits the largest performance variability, including severe
load-regulation and temperature-coefficient violations, high-
lighting the difficulty of RL in navigating the highly con-
strained analog design space with sparse performance feed-
back. In contrast, AgenticSizing achieves the highest success
rate, identifying feasible designs in three out of five trials
within 64–94 simulations, demonstrating superior robustness
and sample efficiency. The failures are limited to output-
voltage and PSRR violations, suggesting that the full-LDO

## Page 9

9
TABLE VI
TWO-STAGE FC: BEST PERFORMANCE SET FOUND BY DIFFERENT APPROACHES (ACROSS FIVE ATTEMPTS). IF ANY SPECIFICATIONS ARE NOT MET
WITHIN TWO HOURS, IT IS MARKED AS A FAILURE.
Method Sim.
Power CMRR ADM PSRR Output swing PM Noise Rise Time Slew UGB CMPM
(mW) (V) (dB) (dB) (V) (◦
) (µV) (ns) (V) (MHz) (◦
)
Target - ≤ 1 ≥ 100 ≥ 90 ≥ 100 ≥ 2.4 ≥ 60 ≤ 335 ≤ 30 ≤ −0.3 ≥ 30 ≥ 50
DE
fail 1.543 103 105 132 2.69 66 434.1 45.91 −0.53 37.67 27
fail 1.281 96 96 126 2.86 74 392.5 42.74 −0.57 27.51 15
fail 1.672 95 95 124 3.57 82 365.1 43.92 −0.51 24.67 20
fail 0.962 71 71 93 2.96 88 289.5 41.00 −0.51 27.23 23
fail 1.403 97 95 106 3.51 72 262.5 32.71 −0.55 43.84 25
BO
fail 0.921 99 87 89 2.94 68 241.2 47.69 −0.67 29.76 62
fail 0.966 104 105 143 2.81 84 365.9 73.79 −0.56 21.85 29
fail 0.676 110 110 134 2.57 78 348.1 56.63 −0.65 22.64 61
fail 1.283 101 97 104 3.53 76 430.8 27.53 −0.55 25.77 42
fail 0.974 114 114 154 2.83 47 328.4 79.08 −0.69 28.70 17
MARL
fail 1.185 100 100 125 3.51 91 472.6 59.63 −0.48 27.30 25
fail 0.533 88 92 138 2.73 27 393.4 116.6 −0.46 41.88 29
fail 1.250 98 98 123 3.31 91 444.4 64.56 −0.45 25.52 10
fail 1.490 90 90 131 3.56 76 427.2 35.37 −0.52 20.65 17
fail 1.403 101 101 127 3.48 70 356.8 52.02 −0.37 17.64 12
AgenticSizing
13 0.975 109 108 123 2.81 88 306.1 29.90 −0.53 56 51
11 0.788 105 100 106 3.28 92 332.8 29.95 −0.56 349 63
29 0.784 103 100 111 2.82 79 334.5 29.98 −0.56 217 61
fail 1.040 116 113 120 2.74 86 329.6 29.92 −0.57 272 52
24 0.946 109 109 130 2.79 80 313.6 28.37 −0.58 51.47 50
Manual - 1.272 107 109 131 3.11 73 332.6 26.39 −0.57 37.58 52
TABLE VII
LDO: BEST PERFORMANCE SET FOUND BY DIFFERENT APPROACHES
ACROSS FIVE ATTEMPTS. IF ANY SPECIFICATIONS ARE NOT MET WITHIN
FOUR HOURS, IT IS MARKED AS A FAILURE.
Method Sim.
Power Vout Load Regulation Line Regulation PSRR
(µW) (V) (V/A) (V/V) (dB)
Target – ≤ 100 [1.15, 1.25] ≤ 0.2 ≤ 0.2 ≥ 50
DE
fail 51.1 1.38 0.045 0.0003 68.0
fail 198.9 1.35 0.064 0.0018 62.9
1131 27.4 1.17 0.090 0.0007 73.5
fail 120.7 1.40 0.044 0.0007 62.9
fail 80.5 1.37 0.202 0.0016 55.6
BO
475 25.9 1.25 0.027 0.0004 68.0
fail 762.1 1.66 2.456 0.1682 50.6
563 35.9 1.22 0.035 0.0004 68.7
533 38.7 1.23 0.033 0.0007 62.6
526 17.1 1.15 0.066 0.0024 69.6
MARL
fail 69.1 0.81 90.613 0.0047 40.2
542 57.2 1.23 0.040 0.0005 65.2
fail 127.0 0.95 92.051 0.0278 29.7
fail 18.7 0.93 0.172 0.0126 37.6
fail 17.2 0.92 0.211 0.0003 72.8
AgenticSizing
77 33.0 1.16 0.050 0.0021 54.5
64 74.5 1.20 0.026 0.0062 51.2
fail 105.6 1.16 0.067 0.0045 54.0
73 44.2 1.15 0.026 0.0002 75.2
103 57.3 1.22 0.055 0.0021 53.9
Manual - 81 1.20 0.12 0.0005 55.3
sizing problem presents substantially tighter and more strongly
coupled design constraints than the standalone LDO bench-
mark. These results indicate that leveraging LLM in-context
learning together with domain-specific knowledge enables
more effective exploration of the constrained design space than
conventional black-box or RL-based optimization approaches.
Fig. 7 shows the optimization trajectories of five inde-
pendent runs. Attempts 1, 2, and 5 first achieved feasibility
TABLE VIII
FULL LDO: PERFORMANCE SETS FOUND BY DIFFERENT APPROACHES
ACROSS FIVE ATTEMPTS. IF ANY SPECIFICATION IS NOT MET WITHIN SIX
HOURS, THE ATTEMPT IS MARKED AS A FAILURE.
Method Sim.
Power Vout
Load
Reg.
Line
Reg.
TC PSRR
(µW) (V) (V/A) (V/V) (ppm) (dB)
Target – ≤ 700 [1.15, 1.25] ≤ 0.2 ≤ 0.2 ≤ 40 ≥ 50
DE
fail 863.8 1.19 0.178 0.0047 41.7 43.8
fail 642.4 0.83 0.532 0.0381 29.3 27.4
fail 1408.1 1.34 0.169 0.0287 24.1 74.7
fail 695.2 0.87 0.297 0.0196 36.0 29.5
fail 918.5 1.03 0.567 0.0349 39.8 27.5
BO
fail 739.8 1.17 0.208 0.0236 24.7 31.3
fail 968.9 1.01 0.196 0.0066 24.0 39.0
fail 895.0 1.12 0.172 0.0054 33.0 40.3
fail 290.7 0.77 0.023 0.0028 32.2 50.0
fail 821.2 0.93 0.187 0.0209 31.5 30.9
MARL
fail 867.8 0.77 0.367 0.0158 163.9 35.6
fail 823.8 1.24 0.801 0.0337 48.4 28.9
fail 214.1 0.65 3.025 0.1391 245.0 15.3
fail 486.6 0.61 0.051 0.0024 11.3 45.6
fail 776.9 1.37 0.901 0.0221 44.0 26.6
AgenticSizing
94 591.4 1.22 0.084 0.0028 28.8 58.3
83 748.0 1.15 0.161 0.0090 39.7 65.5
fail 772.8 1.11 0.343 0.0610 23.8 22.9
92 984.7 1.21 0.171 0.0022 30.9 50.4
fail 924.8 1.14 0.082 0.0015 34.7 51.7
at iterations 64, 83, and 81, respectively, while Attempts 3
and 4 remained infeasible. Attempt 3 rapidly approached the
feasible region, but subsequently oscillated near the specifi-
cation boundaries, particularly for minimum output voltage,
load regulation, and PSRR. This limit-cycle-like behavior
reflects coupled nonlinear trade-offs and the reduced reliability

## Page 10

10
Attempt 1 Attempt 2 Attempt 3 Attempt 4 Attempt 5 Specification
0.3
1
3
10
0 50 100 150
Simulation iteration
Power
(mW)
(a)
1
1.2
1.4
1.6
1.8
0 50 100 150
Simulation iteration
Minimum
output
(V)
(b)
1
1.2
1.4
1.6
1.8
0 50 100 150
Simulation iteration
Maximum
output
(V)
(c)
0.1
1
10
100
0 50 100 150
Simulation iteration
Load
regulation
(d)
0.001
0.01
0.1
1
0 50 100 150
Simulation iteration
Line
regulation
(e)
0
20
40
60
80
0 50 100 150
Simulation iteration
PSRR
(dB)
(f)
20
50
100
500
2000
0 50 100 150
Simulation iteration
Temperature
coefficient
(ppm/°C)
(g)
Fig. 7. Optimization trajectories of performance metrics over five independent attempts for the integrated LDO with FC and BGR. Three attempts reached a
specification-compliant design, while two remained infeasible within the time budget. Shaded regions denote specification-compliant ranges.
of qualitative LLM reasoning for fine-grained convergence.
Overall, the results demonstrate effective multi-specification
coordination while highlighting the run-to-run variability of
LLM-guided optimization.
1) Efficency analysis across complexity levels: Table IX
summarizes the computational efficiency of all methods across
benchmarks of increasing complexity, from standalone ana-
log building blocks to fully integrated LDO with FC and
BGR system. A clear trend emerges: as circuit complexity
and cross-block coupling increase, the robustness and sample
efficiency of conventional optimization methods deteriorate
rapidly, whereas AgenticSizing consistently maintains compet-
itive success rates with substantially fewer circuit simulations.
Among the baseline methods, DE is effective for relatively
simple problems but requires a large number of simulations
and fails to scale to more constrained benchmarks. BO gen-
erally provides higher robustness but at the cost of extensive
black-box exploration, resulting in significantly higher simula-
tion counts. MARL exhibits the weakest scalability, reflecting
the difficulty of learning effective policies from sparse reward
signals in expensive analog design environments.
In contrast, AgenticSizing consistently requires one to two
orders of magnitude fewer simulations while maintaining
comparable or higher success rates. A common characteristic
of LLM-based methods is that each optimization iteration
incurs additional reasoning overhead. This overhead becomes
less pronounced as system complexity increases and SPICE
simulation time dominates the total runtime. By leveraging
in-context learning together with domain-specific knowledge,
AgenticSizing efficiently guides the search toward promising
design regions, reducing unnecessary circuit evaluations. The
TABLE IX
COMPUTATION EFFICIENCY COMPARISON BETWEEN DIFFERENT
APPROACHES AVERAGED ACROSS FIVE ATTEMPTS.
Benchmark Method
Succ.
Rate
Avg.
Iter.
Total
Opt. Time
Opt. Time
per Iter.
Sim. Time
per Iter.
BGR
DE 5/5 936 392 ms 15.29 ms 9 min 45 s
BO 3/5 208 1 h 40 min 11 s 16.25 s 24.26 s
MARL 0/5 – 2 min 49 s 0.30 s 24.72 s
AgenticSizing 4/5 23 1 h 11 min 2 s 121.91 s 36.83 s
Two-stage
FC
DE 0/5 – 74 ms 10.63 ms 17 min 45 s
BO 0/5 – 52 min 32 s 19.34 s 15.73 s
MARL 0/5 – 4 min 4 s 0.34 s 19.69 s
AgenticSizing 4/5 19 1 h 37 min 42 s 186.53 s 22.18 s
LDO
DE 1/5 1131 97 ms 9.48 ms 25 min 44 s
BO 4/5 524 1 h 18 min 22 s 10.38 s 18.70 s
MARL 1/5 542 2 min 9 s 0.23 s 24.53 s
AgenticSizing 4/5 79 2 h 37 min 13 s 52.41 s 27.70 s
LDO with
FC and BGR
DE 0/5 – 192 ms 24.04 ms 48 min 7 s
BO 0/5 – 2 h 38 min 30 s 26.79 s 19.93 s
MARL 0/5 – 11 min 26 s 0.70 s 20.85 s
AgenticSizing 3/5 90 4 h 59 min 15 s 162.68 s 31.27 s
advantage becomes increasingly pronounced as the design
problem grows in dimensionality and constraint coupling,
where AgenticSizing remains the only method capable of
finding feasible solutions for the integrated benchmark.
Overall, these results demonstrate that AgenticSizing man-
ages good trade-off between robustness, scalability, and sam-
ple efficiency, making it promising for handling complex
analog circuit sizing tasks.
2) The impact of task decomposition: The effectiveness of
topology understanding is evaluated by comparing S0 and S1
in Table X, both of which use the same single-agent sizing

## Page 11

11
TABLE X
ABLATION STUDY OF THE PROPOSED FRAMEWORK ON LDO WITH FC
AND BGR AND COMPUTATION EFFICIENCY COMPARISON (AVERAGED
ACROSS FIVE ATTEMPTS).
Methods
Components Metrics
Topology
Tagging
Multi-Agent
Sizing
Design
Knowledge Base
Success
Rate
Avg.
Iter.
Tokens to
feasibility
Total
Tokens
S0 ✗ ✗ ✗ 0% – – 4.064M
S1 ✓ ✗ ✗ 40% 144 2.969M 3.579M
S2 ✓ ✓ ✗ 0% – – 3.370M
S3 ✓ ✗ ✓ 20% 136 3.221M 3.387M
S4 ✓ ✓ ✓ 60% 90 2.771M 3.714M
configuration without a design knowledge base and differ only
in the use of topology tagging. Without topology tagging,
S0 fails in all five attempts, whereas S1 achieves a 40%
success rate. Moreover, topology tagging reduces the average
total token consumption from 4.064 million to 3.579 million,
corresponding to an 11.9% reduction. Although the absolute
token usage remains nontrivial for this complex circuit, the
results indicate that topology tagging enables more targeted
parameter updates and improves both sizing effectiveness and
computational efficiency.
3) The impact of multi-agent sizing framework: The effect
of agent specialization can be seen from comparisons S1–S2
and S3–S4 in Table X. Without design knowledge, replacing
the single agent with specialized sizing agents reduces the
success rate from 40% to 0%, despite a 5.8% reduction
in total token consumption. This result suggests that task
decomposition alone may hinder the coordination of coupled
subblocks. In contrast, with the knowledge base enabled, agent
specialization increases the success rate from 20% to 60%.
Among successful runs, it reduces the average iterations to
feasibility from 136 to 90 and the tokens to feasibility from
3.221 million to 2.771 million, corresponding to reductions
of 33.8% and 14.0%, respectively. Although the average total
token consumption increases by 9.7%, the improved success
rate and lower cost to feasibility indicate that specialized
agents become effective when supported by appropriate design
knowledge.
4) The impact of knowledge infusion: From Table X, the
impact of knowledge infusion can be seen with comparisons
S1–S3 and S2–S4. In the single-agent configuration, adding
the knowledge base reduces the observed success rate from
40% to 20% and increases the tokens to feasibility by 8.5%,
indicating that knowledge injection alone does not ensure
improved optimization. In the multi-agent configuration, how-
ever, knowledge infusion increases the success rate from
0% to 60%, with a 10.2% increase in average total token
consumption.
Overall, these results demonstrate a complementary inter-
action between design knowledge and agent specialization:
the knowledge base provides role-specific guidance, while
the multi-agent decomposition enables such guidance to be
applied systematically to individual functional blocks.
V. CONCLUSION
This work presented a multi-agent LLM-based framework
for complex analog circuit sizing. The proposed workflow
integrates topology understanding, design-knowledge extrac-
tion, and role-specialized agents coordinated by a planner. By
decomposing circuits into functional blocks and incorporating
performance trade-offs and parameter–performance dependen-
cies, the framework enables targeted, interpretable optimiza-
tion with global specification awareness. Ablation studies
demonstrate that topology understanding improves success
rates, while agent specialization is most effective when sup-
ported by design knowledge. The results show that the frame-
work’s performance arises from the coordinated interaction
of structural understanding, knowledge-guided reasoning, and
multi-agent collaboration. Compared with black-box and sin-
gle LLM-based methods, the proposed approach offers a more
transparent and sample-efficient solution for high-dimensional
analog sizing. Future work will extend the framework to larger
analog and mixed-signal systems and advanced technology
nodes, while improving inter-agent communication, memory,
verification, and knowledge transfer across circuit families and
processes.
ACKNOWLEDGMENTS
We thank EDINA at the University of Edinburgh for sup-
porting the access to the OpenAI LLMs through the ELM
gateway. Specifically, GPT-5.5 was used during this work to
support the development of the AgenticSizing framework, as
well as for manuscript proofreading and language refinement.
REFERENCES
[1] G. Gielen and R. Rutenbar, “Computer-aided design of analog and
mixed-signal integrated circuits,” Proceedings of the IEEE, vol. 88,
no. 12, pp. 1825–1854, 2000.
[2] G. Gielen, H. Walscharts, and W. Sansen, “Analog circuit design
optimization based on symbolic simulation and simulated annealing,”
IEEE Journal of Solid-State Circuits, vol. 25, no. 3, pp. 707–713, 1990.
[3] Y. Hao, M. Gandara, S. Mitra, M. Strackx, S. Cochran, F. V. Fernández,
S. Li, and B. Liu, “From systematic to intelligent: Assessing ai-
empowered optimization techniques for analog building block sizing,”
IEEE Access, vol. 13, pp. 172 972–172 986, 2025.
[4] B. Liu, Q. Zhang, and G. G. E. Gielen, “A gaussian process surrogate
model assisted evolutionary algorithm for medium scale expensive opti-
mization problems,” IEEE Transactions on Evolutionary Computation,
vol. 18, no. 2, pp. 180–192, 2014.
[5] R. Martins, N. Lourenço, N. Horta, S. Zhong, J. Yin, P. I. Mak, and R. P.
Martins, “Design of a 4.2-to-5.1 ghz ultralow-power complementary
class-b/c hybrid-mode vco in 65-nm cmos fully supported by eda tools,”
IEEE Transactions on Circuits and Systems I: Regular Papers, vol. 67,
no. 11, pp. 3965–3977, 2020.
[6] B. Liu, Y. Wang, Z. Yu, L. Liu, M. Li, Z. Wang, J. Lu, and F. V.
Fernández, “Analog circuit optimization system based on hybrid evolu-
tionary algorithms,” Integration, vol. 42, no. 2, pp. 137–148, 2009.
[7] B. Liu, F. V. Fernández, G. Gielen, R. Castro-López, and E. Roca, “A
memetic approach to the automatic design of high-performance analog
integrated circuits,” ACM Trans. Des. Autom. Electron. Syst., vol. 14,
no. 3, Jun. 2009.
[8] S. Zhang, W. Lyu, F. Yang, C. Yan, D. Zhou, and X. Zeng, “Bayesian
optimization approach for analog circuit synthesis using neural network,”
in 2019 Design, Automation & Test in Europe Conference & Exhibition
(DATE), 2019, pp. 1463–1468.
[9] B. Liu, D. Zhao, P. Reynaert, and G. G. E. Gielen, “Gaspad: A general
and efficient mm-wave integrated circuit synthesis method based on
surrogate model assisted evolutionary algorithm,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 33,
no. 2, pp. 169–182, 2014.
[10] W. Lyu, P. Xue, F. Yang, C. Yan, Z. Hong, X. Zeng, and D. Zhou, “An
efficient bayesian optimization approach for automated optimization of
analog circuits,” IEEE Transactions on Circuits and Systems I: Regular
Papers, vol. 65, no. 6, pp. 1954–1967, 2018.

## Page 12

12
[11] W. Cao, J. Gao, T. Ma, R. Ma, M. Benosman, and X. Zhang,
“Rose-opt: Robust and efficient analog circuit parameter optimization
with knowledge-infused reinforcement learning,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 44,
no. 2, pp. 627–640, 2025.
[12] Y. Choi, S. Park, M. Choi, K. Lee, and S. Kang, “Ma-opt: Reinforcement
learning-based analog circuit optimization using multi-actors,” IEEE
Transactions on Circuits and Systems I: Regular Papers, vol. 71, no. 5,
pp. 2045–2056, 2024.
[13] M. Ahmadzadeh, J. Lappas, N. Wehn, and G. Gielen, “Anacraft:
Duel-play probabilistic-model-based reinforcement learning for sample-
efficient pvt-robust analog circuit sizing optimization,” IEEE Transac-
tions on Computer-Aided Design of Integrated Circuits and Systems,
vol. 45, no. 2, pp. 901–914, 2026.
[14] A. F. Budak, P. Bhansali, B. Liu, N. Sun, D. Z. Pan, and C. V. Kashyap,
“Dnn-opt: An rl inspired optimization for analog circuit sizing using
deep neural networks,” in 2021 58th ACM/IEEE Design Automation
Conference (DAC), 2021, pp. 1219–1224.
[15] Y. Yin, Y. Wang, B. Xu, and P. Li, “ADO-LLM: Analog Design Bayesian
Optimization with In-Context Learning of Large Language Models,”
in Proceedings of the 43rd IEEE/ACM International Conference on
Computer-Aided Design. Newark Liberty International Airport Marriott
New York NY USA: ACM, Oct. 2024, pp. 1–9.
[16] C. Liu and D. Chitnis, “EEsizer: LLM-Based AI Agent for Sizing of
Analog and Mixed Signal Circuit,” IEEE Transactions on Circuits and
Systems I: Regular Papers, vol. 73, no. 6, pp. 3825–3834, Jun. 2026.
[17] N. Utomo, B. C. T. Teo, X. Y. Lim, V. Navaneethan, Z. Liu, C. B.
Tan, and L. Siek, “An 85.1% peak efficiency, low power class h audio
amplifier with full class h operation,” IEEE Transactions on Circuits
and Systems I: Regular Papers, vol. 70, no. 12, pp. 4692–4704, 2023.
[18] X. Liao, Y. Zhang, S. Zhang, and L. Liu, “A 3.0 µ vrms, 2.4 ppm/°c
bgr with feedback coefficient enhancement and bowl-shaped curvature
compensation,” IEEE Transactions on Circuits and Systems I: Regular
Papers, vol. 71, no. 5, pp. 2424–2433, 2024.
[19] C. Liu and D. Chitnis, “Eesizer: Llm-based ai agent for sizing of analog
and mixed signal circuit,” IEEE Transactions on Circuits and Systems
I: Regular Papers, vol. 73, no. 6, pp. 3825–3834, 2026.
[20] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“AnalogCoder: Analog circuit design via training-free code generation,”
in Proceedings of the AAAI Conference on Artificial Intelligence, 2025.
[21] M. Ahmadzadeh, K. Chen, and G. Gielen, “(Invited Paper) AnaFlow:
Agentic LLM-based Workflow for Reasoning-Driven Explainable and
Sample-Efficient Analog Circuit Sizing,” in 2025 IEEE/ACM Interna-
tional Conference On Computer Aided Design (ICCAD). Munich,
Germany: IEEE, Oct. 2025, pp. 1–7.
[22] C. Liu, W. Chen, A. Peng, Y. Du, L. Du, and J. Yang, “Ampagent:
An llm-based multi-agent system for multi-stage amplifier schematic
design from literature for process and performance porting,” 2024.
[Online]. Available: https://arxiv.org/abs/2409.14739
[23] D. V. Kochar, H. Wang, A. P. Chandrakasan, and X. Zhang, “Ledro: Llm-
enhanced design space reduction and optimization for analog circuits,”
in 2025 IEEE International Conference on LLM-Aided Design (ICLAD),
2025, pp. 141–148.
[24] N. S. K. Somayaji and P. Li, “LLM-USO: Large Language Model-
Based Universal Sizing Optimizer,” IEEE Transactions on Computer-
Aided Design of Integrated Circuits and Systems, vol. 45, no. 4, pp.
1907–1920, Apr. 2026.
[25] C. Liu, W. Chen, H. Xu, Y. Du, J. Yang, and L. Du, “A Large
Language Model-based Multi-Agent Framework for Analog Circuits’
Sizing Relationships Extraction,” in 2025 International Symposium of
Electronics Design Automation (ISEDA). Hong Kong, China: IEEE,
May 2025, pp. 181–187.
[26] T. Yang, B. Li, Y. Li, W. Mao, and G. Han, “Circuit-Agent: A Large
Language Model Based Circuit Agent Framework for Analog/Mixed-
signal Circuit Design Automation,” in 2025 IEEE 7th International
Conference on Artificial Intelligence Circuits and Systems (AICAS).
Bordeaux, France: IEEE, Apr. 2025, pp. 1–5.
[27] S. Yao, J. Zhao, D. Yu, N. Du, I. Shafran, K. Narasimhan, and Y. Cao,
“React: Synergizing reasoning and acting in language models,” 2023.
[Online]. Available: https://arxiv.org/abs/2210.03629
[28] T. Schick, J. Dwivedi-Yu, R. Dessı̀, R. Raileanu, M. Lomeli,
L. Zettlemoyer, N. Cancedda, and T. Scialom, “Toolformer: Language
models can teach themselves to use tools,” 2023. [Online]. Available:
https://arxiv.org/abs/2302.04761
[29] J. Choi, S.-J. Kweon, and H. Jeon, “Single-stage cmos operational
transconductance amplifiers (otas): A design tutorial,” Electronics,
vol. 12, no. 18, 2023. [Online]. Available: https://www.mdpi.com/
2079-9292/12/18/3833
[30] B. Razavi, Design of Analog CMOS Integrated Circuits, 2nd ed. New
York, NY, USA: McGraw-Hill Education, 2017.
[31] P. R. Gray, P. J. Hurst, S. H. Lewis, and R. G. Meyer, Analysis and
Design of Analog Integrated Circuits, 5th ed. Hoboken, NJ, USA:
Wiley, 2009.
[32] A. F. Budak, M. Gandara, W. Shi, D. Z. Pan, N. Sun, and B. Liu, “An
efficient analog circuit sizing method based on machine learning assisted
global optimization,” IEEE Transactions on Computer-Aided Design of
Integrated Circuits and Systems, vol. 41, no. 5, pp. 1209–1221, 2022.
[33] K. Ueno, T. Hirose, T. Asai, and Y. Amemiya, “A 300 nw, 15 ppm/◦c,
20 ppm/v cmos voltage reference circuit consisting of subthreshold
mosfets,” IEEE Journal of Solid-State Circuits, vol. 44, no. 7, pp. 2047–
2054, 2009.
[34] K. N. Leung and P. Mok, “A capacitor-free cmos low-dropout regulator
with damping-factor-control frequency compensation,” IEEE Journal of
Solid-State Circuits, vol. 38, no. 10, pp. 1691–1702, 2003.
[35] J. Bao, J. Zhang, Z. Huang, Z. Bi, X. Feng, X. Zeng, and Y. Lu, “Multi-
agent based reinforcement learning (ma-rl): An automated designer for
complex analog circuits,” IEEE Transactions on Computer-Aided Design
of Integrated Circuits and Systems, vol. 43, no. 12, pp. 4398–4411, 2024.
Yijia Hao received the B.S. degree in electronic and
electrical engineering from the University of Elec-
tronic Science and Technology of China, Chengdu,
China, in 2021. She received the Ph.D. degree with
the University of Glasgow, Glasgow, U.K. in 2025.
She worked as a Postdoctoral Researcher with the
Centre for Electronics Frontiers, University of Ed-
inburgh, U.K. in 2026. Her current research interest
includes analog and mixed-signal IC design.
Pratibha Verma (Member, IEEE) is currently an
Assistant Professor with the Department of Elec-
trical Engineering, Indian Institute of Technology
Indore, India. She received the Ph.D. degree in
Electrical Engineering from Indian Institute of Tech-
nology Delhi, India. Prior to joining IIT Indore,
she worked as a Postdoctoral Researcher with the
Centre for Electronics Frontiers, University of Ed-
inburgh, U.K. Her research interests include analog
and mixed-signal integrated circuit design, energy-
harvesting and power-management circuits, and AI-
assisted electronic circuit design and automation.
Dongxu Guo received the B.S. degree in elec-
tronics engineering from Beijing Normal University,
Beijing, China, in 2021, and the M.Sc. degree in
electronics from The University of Edinburgh, Ed-
inburgh, U.K., in 2022. He is currently pursuing
the Ph.D. degree with the School of Engineering,
The University of Edinburgh, Edinburgh, U.K. His
research interests include analog and mixed-signal
integrated circuits, neuromorphic engineering, and
memristor-based front-end circuits.

## Page 13

13
Cristian Sestito (Member, IEEE) is a Research
Fellow at the Centre for Electronics Frontiers, Uni-
versity of Edinburgh, U.K. He earned his B.Sc.
(2016) and M.Sc. (2019) in Electronic Engineering,
followed by a Ph.D. in Information and Commu-
nication Technologies (2023), from the University
of Calabria, Italy. His PhD focused on the design
and FPGA implementation of convolutional neural
networks, with particular emphasis on dilated and
transposed convolutions. During 2021–2022, he was
a Visiting Scholar at Heriot-Watt University, Edin-
burgh, U.K. where he worked on neural network compression techniques.
His current research interests include digital AI accelerator design, software
simulators for neuromorphic AI, and AI-assisted electronic design automation
frameworks.
Michael O’Boyle (Fellow, IEEE) FBCS FRSE is
a Professor of Computer Science at the University
of Edinburgh. He is best known for his work in
automatic program optimisation, incorporating ma-
chine learning into compilation and parallelisation.
Over his career, he has been awarded three fellow-
ships, five international visiting positions and eleven
best paper awards. His current work focuses on
matching accelerator hardware to legacy code using
program synthesis and neural machine translation
Christos-Savvas Bouganis (Senior Member, IEEE)
is a Professor of Intelligent Digital Systems in
the Department of Electrical and Electronic En-
gineering, Imperial College London, U.K. He
is leading the iDSL group at Imperial College
(https://www.imperial.ac.uk/idsl), with a focus on
the theory and practice of reconfigurable computing
and design automation, mainly targeting the do-
mains of Machine Learning, Computer Vision, and
Robotics.
Themis Prodromakis (Senior Member, IEEE) re-
ceived the bachelor’s degree in electrical and elec-
tronic engineering from the University of Lincoln,
U.K., the M.Sc. degree in microelectronics and
telecommunications from the University of Liver-
pool, U.K., and the Ph.D. degree in electrical and
electronic engineering from Imperial College Lon-
don, U.K. He then held a Corrigan Fellowship in
nanoscale technology and science with the Centre
for Bio-Inspired Technology, Imperial College Lon-
don, and a Lindemann Trust Visiting Fellowship
with the Department of Electrical Engineering and Computer Sciences, Uni-
versity of California at Berkeley, USA. He was a Professor of nanotechnology
at the University of Southampton, U.K. He holds the Regius Chair of
Engineering at the University of Edinburgh. He is currently a Royal Academy
of Engineering Chair in emerging technologies and a Royal Society Indus-
try Fellowship. His background is in electron devices and nanofabrication
techniques. His current research interests include memristive technologies for
advanced computing architectures and biomedical applications. He is a fellow
of the Royal Society of Chemistry, the British Computer Society, the IET, and
the Institute of Physics.

