---
title: "AnalogSAGE: Self-evolving Analog Design Multi-Agents with Stratified Memory and Grounded Experience"
authors: "Wang, Zining; Gao, Jian; Fu, Weimin; Guo, Xiaolong; Zhang, Xuan"
date: "2025/12/27"
category: "end_to_end"
source_url: "https://arxiv.org/abs/2512.22435"
pdf: "papers/2025_analogsage-self-evolving-analog-design-multi-agents-with-stratified-memory_2512-22435.pdf"
sha256: "f571d8c5cd0551286ea8e929342753ca94a728255e39a940f0cc363402122c23"
pages_extracted: 7
---

# AnalogSAGE: Self-evolving Analog Design Multi-Agents with Stratified Memory and Grounded Experience

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AnalogSAGE: Self-evolving Analog Design Multi-Agents with
Stratified Memory and Grounded Experience
Zining Wang
Northeastern
University
Boston, USA
Jian Gao
Northeastern
University
Boston, USA
Weimin Fu
Kansas State
University
Manhattan, USA
Xiaolong Guo
Kansas State
University
Manhattan, USA
Xuan Zhang
Northeastern
University
Boston, USA
Abstract
Analog circuit design remains a knowledge- and experience-intensive
process that relies heavily on human intuition for topology genera-
tion and device parameter tuning. Existing LLM-based approaches
typically depend on prompt-driven netlist generation or predefined
topology templates, limiting their ability to satisfy complex specifi-
cation requirements. We propose AnalogSAGE, an open-source
self-evolving multi-agent framework that coordinates three-stage
agent explorations through four stratified memory layers, enabling
iterative refinement with simulation-grounded feedback. To sup-
port reproducibility and generality, we release the source code.
Our benchmark spans ten specification-driven operational ampli-
fier design problems of varying difficulty, enabling quantitative
and cross-task comparison under identical conditions. Evaluated
under the open-source SKY130 PDK with ngspice, AnalogSAGE
achieves a 10× overall pass rate, a 48× Pass@1, and a 4× reduction
in parameter search space compared with existing frameworks,
demonstrating that stratified memory and grounded reasoning sub-
stantially enhance the reliability and autonomy of analog design
automation in practice.
1 Introduction
Analog circuit design is regarded as one of the most knowledge-
intensive and experience-driven processes in electronic design au-
tomation (EDA) [9]. Unlike digital design, which benefits from
structured synthesis and verification pipelines [3], analog design
requires nuanced trade-offs among specification requirements that
are traditionally guided by human intuition. Despite decades of
research in automation, fully autonomous analog synthesis remains
elusive due to the enormous topology search space, nonlinear in-
teractions among device parameters, and the absence of scalable
reasoning frameworks capable of interpreting design intent [10, 16].
Recent advances in large language models (LLMs) enable human-
like reasoning in analog design automation as they can interpret tex-
tual specifications, retrieve knowledge, and propose design strate-
gies. However, directly applying LLMs to analog circuit design
still remains challenging [2, 9]. Prompt-based approaches such as
SPICEPilot [23] often generate structurally invalid circuits. For
example, they may produce incorrect device connections. A key
reason is that these methods lack feedback-driven refinement it-
erations. In contrast, template- and library-based methods such
as Artisan [6] and AnalogXpert [27] are generally more robust.
However, they still struggle to generalize beyond predefined pat-
terns, which limits their flexibility. Some frameworks [5, 11] in-
corporate simulation results into the feedback loop to mitigate
this issue, yet they rely on simplified device models, preventing
accurate performance evaluation under practical PDK conditions.
Figure 1: Overview of the proposed framework with three
main stages: topology selection, topology refinement, and
parameter optimization. The dashed line indicates closed-
loop feedback. Each stage agent is supported by four memory
layers that provide essential context for the LLM to perform
iterative design improvements.
More fundamentally, existing methods lack persistent cross-task
memory and simulation-grounded experience, leading to brittle
long-horizon reasoning capabilities and reduced reliability under
stringent analog specifications.
To address these challenges, this work presents AnalogSAGE,
which enables LLM to evolve its design capability by generating
new circuit topologies and learning from failed design tasks through
simulation feedback [24], similar to human designers. We design a
stratified context mechanism to selectively preserve stage-relevant
information while discarding irrelevant content, thereby maximiz-
ing effective context usage. Numerical optimization methods are
integrated with LLM-based reasoning to provide accurate sizing
and bias parameters under grounded simulation. We release the
implementation of the framework to facilitate fair comparison and
accelerate the research on LLM-driven analog design.1
The main contributions of this work are as follows:
• Self-evolving analog design multi-agent framework
that continuously enhances reasoning and design capabil-
ity through accumulated knowledge, enabling transferable
domain expertise and strategy refinement from both suc-
cessful and failed design iterations.
• A stratified memory architecture that organizes and
condenses essential design knowledge through four struc-
tured memory layers to support reasoning across different
design stages within limited context windows.
• A closed-loop framework with grounded experience
that unifies topology selection, refinement, and parameter
optimization under simulator feedback. The framework is
evaluated in a set of specification-driven design tasks using
the SKY130 PDK, achieving a 10× pass rate, a 48× Pass@1,
and a 4× reduction in the parameter search space.
1Source code is available at https://github.com/xz-group/AnalogSAGE.
arXiv:2512.22435v1
[cs.AR]
27
Dec
2025

## Page 2

Zining Wang, Jian Gao, Weimin Fu, Xiaolong Guo, and Xuan Zhang
2 Background
2.1 Analog design automation
Traditional analog circuit automation has long depended on ex-
pert knowledge and manual optimization. Early design automation
tools relied on predefined topology libraries selected by human de-
signers, focusing mainly on parameter sizing rather than structural
innovation. Subsequent AI-based methods, including reinforcement
learning [20, 28] and graph neural networks [8, 13], expanded au-
tomation to limited topology generation and device tuning. Despite
these advances, most works are limited to simplified device models,
constrained search spaces, and expert-crafted heuristics. Automat-
ing topology synthesis remains challenging [26] due to the vast
design space, nonlinear performance trade-offs, and the nuanced
judgment that human designers intuitively apply.
2.2 Context management in AI agent
Effective use of LLMs in complex design workflows requires careful
management of limited context windows through isolation, com-
pression, and reflection[15]. Isolated contexts allow agents to focus
on specific subtasks without interference, compressed contexts sum-
marize essential details to conserve space, and reflective contexts
capture insights or mistakes from prior reasoning to improve fu-
ture performance [4]. Techniques such as summarization, pruning,
and retrieval-based re-injection help maintain coherence across
multi-step reasoning while preventing information overload [18].
Through dynamic distillation and reuse of relevant information, the
agent sustains continuity, recalls prior design choices, and applies
analogical reasoning to new problems [14].
Recent works such as ReAct [25] and Reflexion [21] demonstrate
concrete implementations of reflective context mechanisms, com-
bining action traces with self-evaluation to enhance long-horizon
reasoning. Their iterative feedback processes highlight the impor-
tance of structured context updates for robust, multi-step decision
making.
2.3 Agentic frameworks in circuit design
Recent research has explored combining LLMs with circuit sim-
ulators for analog design automation. However, many existing
approaches still rely on prompt engineering [23] or rule-based
scripts [27] without true feedback loops or specification-driven rea-
soning. Their validation is often conducted using idealized device
models [5] rather than practical PDK environments, limiting real-
world applicability. Furthermore, context length and knowledge
retention are underutilized, preventing effective cross-task learning
and simulation-grounded feedback [6, 11].
In addition, most prior systems lack rigorous quantitative eval-
uation, do not test generality across multiple specification-driven
tasks, and rarely release complete code or experimental pipelines,
making independent reproduction and fair comparison difficult.
This work addresses these limitations through a self-evolving,
simulation-grounded framework that integrates multi-context fu-
sion, introspective feedback, and numerical optimization, while
providing open-source code.
3 Methodology
Our proposed framework, AnalogSAGE, translates analog circuit
specifications into executable designs through an iterative, memory-
augmented reasoning process. As illustrated in Figure 1, this process
is driven by three sequential agents: a Topology Selection Agent pro-
poses an initial architecture, a Topology Refinement Agent modifies
its structure to resolve design issues, and a Parameter Optimization
Agent determines the final numerical values for each device.
Crucially, the reasoning of the agents is supported by four Strat-
ified Memory Layers, which form the cognitive backbone of the
system. Evolution Memory transfers high-level insights across dif-
ferent design tasks, while Introspective Optimization enables self-
correction by learning from prior iterations within the same task.
Stage Context Fusion maintains a compressed summary of the cur-
rent design state, and Analog Design Experience grounds the agents
with verified domain knowledge.
This tight integration of agents and memory allows AnalogSAGE
to operate as a self-evolving system, continually improving its
design capabilities across both iterations and tasks. Here, a design
task is defined by a set of specifications that is considered a pass only
when all requirements are satisfied after simulation. Because initial
solutions may fail, multiple design iterations are often required
within a single task to refine the solution until the specifications
are met, or the design still fails after reaching the maximum number
of iterations.
3.1 Stage agents
The design process is divided into three reasoning stages, each con-
trolled by a stage agent. Agents operate with different objectives:
selecting a topology, refining its structure, and determining nu-
merical parameter values. All agents share the same memory layer
organization, but receive different tool feedback and contextual
information. Information flows forward across stages, while com-
pressed feedback is passed backward through the memory system,
forming a closed loop that can iteratively solve the design task.
3.1.1 Topology Selection Agent. The Topology Selection Agent initi-
ates the design process by translating textual performance speci-
fications into a concrete circuit architecture in netlist format. To
avoid generating invalid topologies, the agent first queries the Strat-
ified Memory Layers for contextual signals derived from past design
tasks and previous iterations. Conditioned on this context, it formu-
lates a natural language description of the target architecture. This
description is passed to the Candidates Module, which retrieves the
best-matching validated topology from the database. This retrieval-
based initialization ensures that the design process begins from a
structurally sound foundation.
3.1.2 Topology Refinement Agent. Once a promising topology is
identified, the design flow enters an iterative refinement stage.
Based on the gathered context, the agent performs localized struc-
tural modifications, such as adjusting bias networks or compensa-
tion paths. The refined topology, or the original one if no modifica-
tion is required, is then passed to the next stage. All refinements are
intentionally constrained to local edits to ensure that the resulting
topology remains structurally valid.

## Page 3

AnalogSAGE: Self-evolving Analog Design Multi-Agents with Stratified Memory and Grounded Experience
Figure 2: Interaction between the four context layers and the stage agents: Evolution Memory, Introspective Optimization,
Stage Context Fusion, and Analog Design Experience. Each design iteration updates different parts of the memory layers based
on the generated results.
3.1.3 Parameter Optimization Agent. Once the topology is fixed,
the framework switches the reasoning mode from structural de-
cisions to numerical prediction. The agent first infers feasible pa-
rameter ranges for all devices in the circuit, including transistor
dimensions, bias conditions, and passive elements. These ranges de-
fine the search space and call the Numerical Module to run Bayesian
optimization and circuit simulation to evaluate candidate parameter
sets. The resulting performance metrics are fed back into the Strat-
ified Memory Layers, allowing the agent to tighten the parameter
bounds in subsequent iterations and progressively converge toward
a specification-satisfying solution.
3.2 Stratified memory layers
The four memory layers provide the contextual backbone for the
framework. They allow the stage agents to reason with both long-
term transferable expertise and short-term task feedback. As shown
in Figure 2, each layer contributes a different form of context: the
Evolution Memory Layer stores success design insights shared across
tasks, the Introspective Optimization Layer records failure reflec-
tions within a task, the Stage Context Fusion Layer compresses
intermediate reasoning traces, and the Analog Design Experience
Layer exposes tool-level and domain-specific knowledge outside
the LLM. Working together, these layers enable iterative refinement
without exceeding model context limits, and allow the system to
improve over repeated deployments.
3.2.1 Evolution Memory Layer. It provides long-term transferable
knowledge across design tasks. After each successful task, the In-
sight Module generates a concise summary of the underlying design
rationale, including effective topology choices, compensation strate-
gies, and reasoning heuristics that led to convergence. Each sum-
mary is stored together with the associated specification, topology,
and performance results.
When a new task arrives, the framework retrieves entries and
injects them into the prompt as contextual guidance. Over time, this
iterative accumulation and reuse of insights forms a transferable
representation of analog design knowledge, enabling cross-task
generalization and reducing the number of iterations required to
meet new specifications, as it can learn from prior successful designs
to have a higher chance of picking the right solution path.
3.2.2 Introspective Optimization Layer. While the Evolution Mem-
ory Layer supports cross-task generalization, the Introspective Op-
timization Layer enables within-task self-correction. After each
unsuccessful design iteration, the Reflection Module analyzes the
reasoning trace and simulation results, and produces a concise re-
flection explaining the failure. Typical causes include inappropriate
topology selection, insufficient biasing, or conflicting specification
trade-offs. Each reflection also provides a concrete corrective action
in future iterations and is stored in the Introspective Optimization
Layer.
In subsequent iterations under the same design task, where the
specification remains unchanged. The framework retrieves these
reflections and injects them into the prompt, preventing repeated
failure patterns. This mirrors the workflow of expert analog design-
ers, who refine intuition through targeted failure analysis rather
than restarting from scratch. By continually incorporating these re-
flections, the system accumulates task-specific optimization heuris-
tics, forming a closed loop that improves iteration efficiency and
reduces redundant exploration during analog synthesis.
3.2.3 Stage Context Fusion Layer. As the number of iterations in-
creases within a single design task, the accumulated textual context,
including LLM design history, retrieved knowledge, and simulation
feedback, grows rapidly in length. Such expansion increases compu-
tational overhead and degrades reasoning quality due to the LLM’s
limited context capacity. To mitigate this problem, the Stage Context
Fusion Layer employs a Compression Module that condenses long
reasoning traces into concise summaries while preserving essen-
tial performance outcomes and decision rationales in the original
context.

## Page 4

Zining Wang, Jian Gao, Weimin Fu, Xiaolong Guo, and Xuan Zhang
The fused context is then passed backward across stages, allow-
ing later stage results to inform earlier stage decisions. This mech-
anism forms a feedback bridge that maintains continuity across
iterations, reduces redundant decision exploration, and enables
progressively more focused reasoning. By continuously distilling
and re-injecting contextual information, the system establishes a
closed-loop refinement process without exceeding model context
limits.
3.2.4 Analog Design Experience Layer. This layer grounds the LLM’s
reasoning in verifiable analog design knowledge rather than relying
solely on language priors. Instead of generating circuits purely from
the model itself, the stage agents could use APIs to communicate
with external tools whenever structural evidence, analog domain
knowledge, or numerical validation is required. These interactions
are routed through three modules: the Candidates Module, which re-
trieves structurally valid topologies; the Knowledge Module, which
provides literature-derived design guidance; and the Numerical
Module, which returns simulation-based performance feedback. By
integrating these tool results directly into the reasoning loop, the
system forms a hybrid symbolic–numeric workflow in which every
major design decision is supported by grounded evidence rather
than speculative generation.
Candidates Module. This module works with the Topology Vector
Database, which stores analog circuit topologies paired with their
netlists and concise two-sentence textual summaries. The first sen-
tence describes the global architectural style (e.g., folded-cascode,
two-stage Miller), while the second explains the internal stage struc-
ture and signal flow. These summaries are embedded into a vector
database for semantic retrieval.
As shown in Figure 3, each entry is constructed from two sources:
the netlist and its original design document. The LLM parses the
source text and generates the two-sentence description, which
is then embedded into a vector representation. The database is
organized as a key–value store, where the embedded description
serves as the key and the netlist as the value, enabling efficient
retrieval from textual queries.
When triggered, the stage agent first produces a full natural-
language description of the desired topology. The Candidates Mod-
ule then condenses this long description into the same two-sentence
format used by the database and performs a cosine similarity search
to obtain structurally relevant candidates. The retrieved results are
further re-ranked according to constraints inferred from the agent’s
original description, ensuring that the final ordering reflects the in-
tended architectural direction rather than pure semantic similarity.
The re-ranked topologies are returned to the design loop as valid
structural hypotheses for subsequent refinement.
Knowledge Module. This module operates on a Retrieval-Augmented
Generation (RAG) system that enriches the LLM’s reasoning with
domain-specific knowledge extracted from human-authored ana-
log design literature. All collected documents are segmented into
semantically coherent sections, with non-informative content such
as acknowledgements removed. The processed segments are then
embedded to construct a Design Knowledge RAG.
During the design process, whenever a stage agent requires
theoretical or empirical guidance, such as specification trade-offs,
Figure 3: Construction workflow of the Topology Vector Data-
base. Each topology is converted into a two-sentence textual
representation and stored as a key–value pair, where the em-
bedded description serves as the key and the netlist as the
value, enabling efficient retrieval.
the module is invoked. It automatically expands the original query
into multiple sub-queries, searches the RAG, and retrieves the most
relevant text segments. To prevent context overflow, the retrieved
content is summarized into concise paragraphs tailored to the query,
filtering out irrelevant information before being returned to the
agent as auxiliary knowledge. This module ensures that the LLM can
reason with up-to-date and contextually relevant analog knowledge
without exceeding its context length limitations.
Together with the Candidates Module, human-grounded analog
experiences are inserted into the design reasoning process.
Numerical Module. The Numerical Module serves as the numer-
ical optimization component of the design pipeline. It deploys
Bayesian Optimization (BO) to optimize device parameters while
accommodating diverse circuit topologies. Once triggered, the BO
script iteratively samples parameter values, calls the Circuit Simula-
tor, evaluates performance metrics, and updates a surrogate model
to identify the optimal configuration.
Unlike the LLM, whose reasoning over analog behavior may
be approximate or speculative, circuit simulation provides ground-
truth numerical feedback which therefore injects simulation-grounded
performance signals into the design loop, ensuring that parameter
decisions are guided by accurate device-level evidence rather than
language-only predictions.
4 An Example Task Walk-through
To demonstrate the workflow in AnalogSAGE, we use an opera-
tional amplifier (op-amp) design task with a set of spec require-
ments, as shown in Table 1, which is intentionally designed to be
challenging and exceeds the design space in the topology candidates
database as an example to illustrate how AnalogSAGE generates a
new circuit design.
As shown in Figure 4, the Topology Selection Agent, equipped
with memory from the Insight Module and augmented by the Knowl-
edge Module, identified an initial topology, which is a two-stage
operational amplifier. This topology is then passed to the Topology
Refinement Agent, which performed no modification and directly
forwarded the design to the Parameter Optimization Agent, where
parameter ranges were estimated and then provided to the Nu-
merical Module for numerical optimization. However, the resulting
design failed to meet the target specifications, particularly the phase
margin. As a failure case in the first iteration, the Compression Mod-
ule analyzed the cause of failure, which in this case was insufficient

## Page 5

AnalogSAGE: Self-evolving Analog Design Multi-Agents with Stratified Memory and Grounded Experience
Table 1: Spec targets and simulation results in the example.
Specifications Unit Design Target AnalogSAGE Result
Gain dB ≥ 70 72.88
GBW Hz ≥ 2 × 105 2.4 ×105
Phase Margin deg ≥ 60 63.80
CMRR dB ≥ 50 54.57
PSRR dB ≥ 40 43.88
PSRN dB ≥ 40 46.55
Power 𝜇W ≤ 35 30.35
Figure 4: The log of the walk-through example. The refined
structure is marked in red, adding Miller compensation with
a nulling resistor connected to the output (Vout).
phase margin, and the Reflection Module generated a reflection
stating that phase margin must be improved. This feedback was
returned to the Topology Refinement Agent.
In the second iteration, the Topology Refinement Agent received
the failure analysis and reflection from the previous iteration and
queried the Knowledge Module again for improvement suggestions.
The retrieved knowledge summarized that introducing Miller com-
pensation would improve phase margin. Following this suggestion,
the Topology Refinement Agent performed refinement by adding
a nulling resistor in series with the Miller capacitor. The modi-
fied topology was regenerated as a PySpice [19] subcircuit and
converted into an ngspice [22] compatible netlist, since the LLM
is more familiar with Python-based circuit descriptions than raw
SPICE syntax. Through iterative optimization in the Parameter Opti-
mization Agent, the system identified device parameters whose sim-
ulated performance satisfied all target specifications. This marked
the design as successfully completed. The final result was then
stored to support self-evolution and improve performance on fu-
ture design tasks.
5 Experiment Setup
5.1 Specification-driven design tasks
To evaluate the proposed system, we selected op-amps as the tar-
get circuit family, since they represent a fundamental building
block in analog design. As shown in Table 2, we defined ten de-
sign tasks with gradually increasing difficulty, grouped into three
levels. Each task defines a combination of performance metrics,
including power, DC gain (Gain), common-mode rejection ratio
(CMRR), power supply rejection ratio (PSRR), gain-bandwidth prod-
uct (GBW), phase margin (PM), and power supply noise rejection
Table 2: Specification requirements for Tasks 1–10.
Level Task
Power
(𝜇W)
Gain
(dB)
CMRR
(dB)
PSRR
(dB)
GBW
(Hz)
PM
(deg)
PSRN
(dB)
Easy
1 ≤ 1000 ≥ 45 ≥ 20 ≥ 20 ≥ 1.0 × 105 ≥ 60 ≥ 20
2 ≤ 1000 ≥ 45 ≥ 40 ≥ 40 ≥ 1.0 × 106 ≥ 60 ≥ 40
3 ≤ 100 ≥ 80 ≥ 50 ≥ 50 ≥ 1.0 × 106 ≥ 60 ≥ 50
4 ≤ 100 ≥ 80 ≥ 70 ≥ 70 ≥ 1.0 × 106 ≥ 70 ≥ 70
5 ≤ 100 ≥ 100 ≥ 70 ≥ 70 ≥ 1.0 × 106 ≥ 70 ≥ 70
Medium
6 ≤ 10 ≥ 60 ≥ 70 ≥ 70 ≥ 5.0 × 105 ≥ 50 ≥ 70
7 ≤ 10 ≥ 60 ≥ 70 ≥ 70 ≥ 5.0 × 105 ≥ 50 ≥ 70
8 ≤ 1 ≥ 25 ≥ 55 ≥ 55 ≥ 1.0 × 105 ≥ 45 ≥ 55
9 ≤ 2 ≥ 45 ≥ 30 ≥ 30 ≥ 1.0 × 106 ≥ 45 ≥ 30
Hard 10 ≤ 5 ≥ 80 ≥ 30 ≥ 30 ≥ 1.0 × 106 ≥ 45 ≥ 30
(PSRN). Tasks 1–5 correspond to relatively easy specifications un-
der relaxed constraints, while Tasks 6–9 impose stricter limits on
power, bandwidth, and noise rejection requirements. Task 10 is the
most challenging case.
5.2 Baselines from prior works
To ensure a fair and representative evaluation, we compared our
method against two opensource frameworks and adapted one method
to our framework.
• SPICEPilot [23] focuses on automatic SPICE netlist gener-
ation and verification through a PySpice-based benchmark.
It integrates LLM-driven code synthesis, simulation, and
rule-based correctness checks under zero-shot prompting.
• AnalogCoder-Pro [11] employs one-shot prompting with
multiple iterations using PySpice simulations to generate
sized topology results.
• Artisan-style SAGE is based on the Artisan [6], which
integrates several predefined design patterns and instructs
the LLM to generate designs based on given candidates.
Since their codebase and candidate sets are not publicly
available, we construct our own candidate library and em-
bed it into the prompts. We further apply the Numerical
Module for parameter optimization.
5.3 Evaluation metrics
We employed three complementary metrics to assess design suc-
cess, reasoning efficiency, and optimization effectiveness: Pass@k,
Average Iterations, and Normalized Parameter Search Space.
Pass@k. The Pass@k metric measures the probability that at least
one correct design appears among 𝑘 generated trials:Pass@k =
1 −
(𝑛−𝑐
𝑘 )
(𝑛
𝑘)
, where 𝑛 is the total number of trials and 𝑐 is the number
of correct ones. In our experiments, we use 𝑛 = 5 and 𝑘 = 1.
Average Iterations. For iterative frameworks, we report the aver-
age number of reasoning iterations needed to reach a valid design.
Fewer iterations indicate more efficient reasoning and stronger
convergence.
Normalized Parameter Search Space. We compute the parameter
search range selected by the agent and normalize it to the full
search space (set to 1). Smaller values indicate tighter parameter
constraints and more efficient optimization.

## Page 6

Zining Wang, Jian Gao, Weimin Fu, Xiaolong Guo, and Xuan Zhang
Table 3: Pass@1 comparison between proposed AnalogSAGE
and baselines.
Framework 1 2 3 4 5 6 7 8 9 10 Avg.
SPICEPilot [23] 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
AnalogCoder-Pro [11] 0.0 0.0 0.0 20.0 0.0 0.0 0.0 0.0 0.0 0.0 2.0
Artisan-style SAGE 100.0 80.0 80.0 60.0 100.0 20.0 0.0 0.0 20.0 0.0 46.0
AnalogSAGE 100.0 100.0 100.0 100.0 100.0 80.0 100.0 100.0 100.0 80.0 96.0
Table 4: Evaluation result under four configurations.
Configuration Metric 1 2 3 4 5 6 7 8 9 10
W/o IO and EM Pass@1 100.0 80.0 100.0 40.0 80.0 40.0 20.0 60.0 40.0 0.0
W/o IO Pass@1 100.0 100.0 100.0 80.0 100.0 20.0 40.0 40.0 60.0 0.0
W/o EM
Pass@1 100.0 100.0 100.0 100.0 100.0 40.0 60.0 80.0 100.0 20.0
Avg Iter 1.0 1.2 1.4 1.4 1.2 NA NA NA 1.8 NA
Full(AnalogSAGE)
Pass@1 100.0 100.0 100.0 100.0 100.0 80.0 100.0 100.0 100.0 80.0
Avg Iter 1.0 1.0 1.2 1.0 1.2 NA 1.2 1.4 1.0 NA
5.4 Agent framework configuration
We constructed the Topology Database by labeling 50 representative
topology candidates and built the Design Knowledge RAG using ap-
proximately 10,000 analog design papers collected from ISSCC pro-
ceedings [1] spanning 1955 to 2024. All experiments were conducted
using the gemini-2.5-flash [7] and text-embedding-3-large [17]
models via API access. Circuit simulations were performed using
ngspice under the SKY130 PDK, adapted from the open-source
AnalogGym script [12]. For fairness, we set the maximum num-
ber of reasoning iterations in AnalogSAGE to three, matching the
iteration limit used in AnalogCoder-Pro.
6 Experiment Results
6.1 Performance comparison
Table 3 compares the proposed AnalogSAGE against baselines. SPI-
CEPilot, which relies solely on zero-shot prompting without sim-
ulation feedback or iterative reasoning, fails on all tasks, yielding
a 0% pass rate and 0% Pass@1. AnalogCoder-Pro introduces lim-
ited feedback but lacks grounded numerical optimization under the
PDK. It occasionally discovers a correct design after many trials,
achieving a 10% pass rate and only a 2% Pass@1. Artisann-style
SAGE benefits from predefined circuit templates and simulation-
guided parameter tuning, achieving a 70% pass rate. However, its
first-try success remains below 50%, demonstrating weak general-
ization to strict specifications. In contrast, AnalogSAGE achieves a
100% pass rate, meaning every design task is eventually solved,
and a 96.0% Pass@1, meaning 96% of tasks succeed without re-
quiring additional trials. This reflects a 10 × pass rate and a 48×
in Pass@1 over the prior works, demonstrating stronger reliabil-
ity, faster convergence, and better generalization across difficulty
levels.
6.2 Role of self-evolving memory layers
To quantify the contribution of the proposed self-evolving memory
design, we conducted experiments under four configurations. The
lowest configuration disables both the Introspective Optimization
Layer (IO) and the Evolution Memory Layer (EM), denoted as W/o
IO and EM. The second removes only IO (W/o IO), while the third
removes only EM (W/o EM). The full system keeps both layers
enabled. All configurations were evaluated across ten design tasks,
and the results are summarized in Table 4.
Table 5: Comparison of normalized parameter search space
across different configurations.
Method 1 2 3 4 5 6 7 8 9 10 Avg.
W/o IO and Knowledge 0.11 0.36 0.67 1.00 1.00 1.00 0.62 0.31 0.37 0.29 0.57
W/o IO 0.29 0.03 0.89 0.31 0.37 0.35 0.36 0.58 0.06 0.0008 0.32
W/o Knowledge 0.03 0.37 0.64 1.00 1.00 1.00 1.00 0.30 0.37 0.33 0.60
Full (AnalogSAGE) 0.07 0.02 0.27 1.00 0.45 0.32 0.05 0.37 0.04 0.003 0.26
The system without either IO or EM can handle easy and medium
tasks, but completely fails on the most challenging ones. It also
exhibits unstable convergence, often requiring multiple trials to ob-
tain a feasible solution. Adding only EM improves performance on
medium tasks by enabling cross-task knowledge transfer, which ini-
tializes the search closer to feasible operating regions. The gradual
Pass@1 improvement (from 20% to 60% in Tasks 6–9) confirms that
accumulated design priors benefit later tasks, although the system
still fails on the hardest case due to the lack of adaptive correction.
In contrast, enabling only IO allows the system to eventually solve
Task 10, demonstrating that introspective self-correction can repair
failing design trajectories. However, its Pass@1 remains low (20%),
indicating that without insight priors the system lacks stability.
When both IO and EM are enabled, the framework achieves the
best overall performance, reaching a 100% pass rate and the highest
Pass@1 with fewer iterations. This shows that the Evolution Memory
Layer provides informed initialization for new design tasks, while
the Introspective Optimization Layer adaptively corrects reasoning
paths using grounded feedback. The two layers are complementary
and jointly necessary for robust autonomous analog design.
6.3 Parameter optimization efficiency
Beyond reasoning accuracy, we also evaluated AnalogSAGE’s effi-
ciency in parameter optimization. In this stage, the LLM predicts
parameter ranges for each transistor and passive component. These
predicted ranges directly shape the search space for the subsequent
numerical optimizer. Tighter and more accurate ranges reduce the
search space, accelerate convergence, and increase the likelihood
of success. Table 5 shows the normalized parameter search space
across different configurations where the smaller is better. Our
AnalogSAGE achieves an average space of 0.26, indicating a 4×
reduction in parameter search space, which outperforms other
three configurations. These results confirm that the Knowledge
Module provides physically consistent parameter bounds, while the
Introspective Optimization Layer further tightens feasible regions
through reflective, simulation-driven feedback.
7 Conclusion
This work presents AnalogSAGE that integrates LLM-based self-
evolving reasoning, stratified memory, and grounded experience.
By coordinating three stage agents under a four-layer memory ar-
chitecture, it tightly couples symbolic reasoning with numerical
optimization in a closed-loop workflow. The results suggest that
self-evolving, memory-structured agentic frameworks can capture
and generalize analog design knowledge in ways that parallel hu-
man learning. Across ten specification-driven op-amp design tasks,
AnalogSAGE achieves a 10× pass rate, a 48× pass@1, and a 4×
reduction in parameter search space.

## Page 7

AnalogSAGE: Self-evolving Analog Design Multi-Agents with Stratified Memory and Grounded Experience
References
[1] [n. d.]. ISSCC - International Solid-State Circuits Conference — isscc.org. https:
//www.isscc.org/. [Accessed 18-11-2025].
[2] Arman Akbari, Jian Gao, et al. 2025. CircuitSense: A Hierarchical Circuit Sys-
tem Benchmark Bridging Visual Comprehension and Symbolic Reasoning in
Engineering Design Process. arXiv preprint arXiv:2509.22339 (2025).
[3] Deepthi Amuru, Andleeb Zahra, et al. 2023. AI/ML algorithms and applications
in VLSI design and technology. Integration 93 (2023), 102048. doi:10.1016/j.vlsi.
2023.06.002
[4] Edward Y Chang and Longling Geng. 2025. SagaLLM: Context Management,
Validation, and Transaction Guarantees for Multi-Agent LLM Planning. arXiv
preprint arXiv:2503.11951 (2025).
[5] Pin-Han Chen, Yu-Sheng Lin, et al. 2025. MenTeR: A fully-automated Multi-
agenT workflow for end-to-end RF/Analog Circuits Netlist Design. arXiv preprint
arXiv:2505.22990 (2025).
[6] Zihao Chen, Jiangli Huang, et al. 2024. Artisan: Automated operational amplifier
design via domain-specific large language model. In Proceedings of the 61st
ACM/IEEE Design Automation Conference. 1–6.
[7] Gheorghe Comanici, Eric Bieber, et al. 2025. Gemini 2.5: Pushing the frontier with
advanced reasoning, multimodality, long context, and next generation agentic
capabilities. arXiv preprint arXiv:2507.06261 (2025).
[8] Zehao Dong, Weidong Cao, et al. 2023. CktGNN: Circuit graph neural network
for electronic design automation. arXiv preprint arXiv:2308.16406 (2023).
[9] Xingang Guo, Yaxin Li, et al. 2025. Toward Engineering AGI: Benchmarking
the Engineering Design Capabilities of LLMs. arXiv preprint arXiv:2509.16204
(2025).
[10] Vladyslav Hamolia and Viktor Melnyk. 2021. A Survey of Machine Learning
Methods and Applications in Electronic Design Automation. In 2021 11th In-
ternational Conference on Advanced Computer Information Technologies (ACIT).
757–760. doi:10.1109/ACIT52158.2021.9548117
[11] Yao Lai, Souradip Poddar, et al. 2025. Analogcoder-pro: Unifying analog circuit
generation and optimization via multi-modal llms. arXiv preprint arXiv:2508.02518
(2025).
[12] Jintao Li, Haochang Zhi, et al. 2025. AnalogGym: An Open and Practical Testing
Suite for Analog Circuit Synthesis. In Proceedings of the 43rd IEEE/ACM Inter-
national Conference on Computer-Aided Design (Newark Liberty International
Airport Marriott, New York, NY, USA) (ICCAD ’24). Association for Computing
Machinery, New York, NY, USA, Article 59, 9 pages. doi:10.1145/3676536.3697117
[13] Mingjie Liu, Walker J Turner, et al. 2021. Parasitic-aware analog circuit sizing with
graph neural networks and Bayesian optimization. In 2021 Design, automation &
test in Europe conference & exhibition (DATE). IEEE, 1372–1377.
[14] Miao Lu, Weiwei Sun, et al. 2025. Scaling llm multi-turn rl with end-to-end
summarization-based context management. arXiv preprint arXiv:2510.06727
(2025).
[15] Lingrui Mei, Jiayu Yao, et al. 2025. A survey of context engineering for large
language models. arXiv preprint arXiv:2507.13334 (2025).
[16] Rayan Mina, Chadi Jabbour, and George E. Sakr. 2022. A Review of Machine
Learning Techniques in Analog Integrated Circuit Design Automation. Electronics
11, 3 (2022). doi:10.3390/electronics11030435
[17] Arvind Neelakantan, Tao Xu, et al. 2022. Text and code embeddings by contrastive
pre-training. arXiv preprint arXiv:2201.10005 (2022).
[18] Charles Packer, Vivian Fang, et al. 2023. MemGPT: Towards LLMs as Operating
Systems. (2023).
[19] Fabrice SALVAIRE. [n. d.]. PySpice — pyspice.fabrice-salvaire.fr. https://pyspice.
fabrice-salvaire.fr/. [Accessed 18-11-2025].
[20] Keertana Settaluri, Zhaokai Liu, et al. 2022. Automated Design of Analog Circuits
Using Reinforcement Learning. IEEE Transactions on Computer-Aided Design of
Integrated Circuits and Systems 41, 9 (2022), 2794–2807. doi:10.1109/TCAD.2021.
3120547
[21] Noah Shinn, Federico Cassano, et al. 2023. Reflexion: Language agents with
verbal reinforcement learning. Advances in Neural Information Processing Systems
36 (2023), 8634–8652.
[22] Holger Vogt, Giles Atkinson, et al. 2021. Ngspice User’s Manual Version 34
(ngspice release version).
[23] Deepak Vungarala, Sakila Alam, et al. 2024. Spicepilot: Navigating spice code
generation and simulation with ai guidance. In 2024 IEEE International Conference
on Rebooting Computing (ICRC). IEEE, 1–6.
[24] Xiaomeng Yang, Jian Gao, et al. 2025. Zerosim: Zero-shot analog circuit eval-
uation with unified transformer embeddings. arXiv preprint arXiv:2511.07658
(2025).
[25] Shunyu Yao, Jeffrey Zhao, et al. 2022. React: Synergizing reasoning and act-
ing in language models. In The eleventh international conference on learning
representations.
[26] Danial Noori Zadeh and Mohamed B. Elamien. 2025. Generative AI for Analog
Integrated Circuit Design: Methodologies and Applications. IEEE Access (2025),
1–1. doi:10.1109/ACCESS.2025.3553743
[27] Haoyi Zhang, Shizhao Sun, et al. 2025. Analogxpert: Automating analog topology
synthesis by incorporating circuit design expertise into large language models.
In 2025 International Symposium of Electronics Design Automation (ISEDA). IEEE,
772–777.
[28] Zhenxin Zhao and Lihong Zhang. 2020. Deep reinforcement learning for analog
circuit sizing. In 2020 IEEE International Symposium on Circuits and Systems
(ISCAS). IEEE, 1–5.

