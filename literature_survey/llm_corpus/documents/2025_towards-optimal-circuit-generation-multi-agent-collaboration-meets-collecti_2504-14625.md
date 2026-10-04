---
title: "Towards Optimal Circuit Generation: Multi-Agent Collaboration Meets Collective Intelligence"
authors: "Qin, Haiyan; Feng, Jiahao; Feng, Xiaotong; Xing, Wei W.; Kang, Wang"
date: "2025/04/20"
category: "adjacent_applications"
source_url: "https://arxiv.org/abs/2504.14625"
pdf: "papers/2025_towards-optimal-circuit-generation-multi-agent-collaboration-meets-collecti_2504-14625.pdf"
sha256: "77df5c27217ca37b7e91fa35e3dd5a0ee5667cd8df40ce727c682154a9920ed2"
pages_extracted: 9
---

# Towards Optimal Circuit Generation: Multi-Agent Collaboration Meets Collective Intelligence

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

Towards Optimal Circuit Generation: Multi-Agent
Collaboration Meets Collective Intelligence
*Note: This work has been submitted to the IEEE for possible publication. Copyright may be transferred without notice.
Haiyan Qin∗, Jiahao Feng‡, Xiaotong Feng∗, Wei W. Xing†§, Wang Kang∗§
∗ National Key Laboratory of Spintronics, Hangzhou International Innovation Institute;
School of Integrated Circuit Science and Engineering, Beihang University, China
‡ School of Mechanical and Electronic Engineering,
Wuhan University of Technology, Wuhan, China
† School of Mathematical and Physical Sciences, University of Sheffield, Sheffield, United Kingdom
∗ {haiyanq, betty513, wang.kang}@buaa.edu.cn, ‡ 325167@whut.edu.cn, † w.xing@sheffield.ac.uk
Abstract—Large language models (LLMs) have transformed
code generation, yet their application in hardware design pro-
duces gate counts 38%–1075% higher than human designs. We
present CircuitMind, a multi-agent framework that achieves
human-competitive efficiency through three key innovations:
syntax locking (constraining generation to basic logic gates),
retrieval-augmented generation (enabling knowledge-driven de-
sign), and dual-reward optimization (balancing correctness with
efficiency). To evaluate our approach, we introduce TC-Bench,
the first gate-level benchmark harnessing collective intelligence
from the TuringComplete ecosystem—a competitive circuit de-
sign platform with hundreds of thousands of players. Experi-
ments show CircuitMind enables 55.6% of model implementa-
tions to match or exceed top-tier human experts in composite
efficiency metrics. Notably, our framework elevates the 14B-
parameter Phi-4 model (ranked 6th among 1,000 human experts
on TuringComplete) to outperform both GPT-4o mini and Gemini
2.0 Flash without requiring specialized training. These innova-
tions establish a new paradigm for hardware optimization where
collaborative AI systems leverage collective human expertise to
achieve optimal circuit designs. Our model, data, and code are
open-source at https://github.com/BUAA-CLab/CircuitMind.
Index Terms—circuit design, large language models, multi-
agent systems, Boolean optimization, gate-level netlist
I. INTRODUCTION
Large language models (LLMs) have demonstrated rev-
olutionary capabilities in code generation across multiple
domains, yet their application to hardware design reveals a
significant efficiency gap. When tasked with generating digital
circuits, even state-of-the-art LLMs produce implementations
requiring 38%–1075% more gates than those created by skilled
human designers. As shown in Table I, this inefficiency is par-
ticularly pronounced in complex circuits involving conditional
logic and arithmetic operations.
TABLE I: Comparison of LLM-Generated and Human-
Optimized Circuits in TuringComplete [1]
Circuit LLM (Behavioral) Human Expert Overhead
8-bit Adder 142 57-75 89%-149%
Logic Engine 125 52-70 79%-140%
Arithmetic Engine 348 102-252 38%-241%
Conditional Check 94 8-18 422%-1075%
Synthesis via Yosys [2] (LLM: Gemini 2.0 Flash)
This significant performance disparity stems from what
we term the Boolean Optimization Barrier—a fundamental
limitation in how individual LLMs approach gate-level digital
design. Unlike human experts who apply systematic minimiza-
tion techniques and Boolean algebra manipulations, LLMs
lack the structured search processes and symbolic reasoning
capabilities essential for finding globally optimal solutions at
the netlist level. They excel at generating functionally correct
code but struggle with the complex trade-offs required to
achieve physical efficiency. This barrier becomes particularly
evident when analyzing how LLMs handle conditional state-
ments, combinational loops, and arithmetic operations, which
they often implement using redundant gate structures that
human experts would readily simplify.
The field’s response to this challenge has evolved through
three distinct phases (Fig. 1). Initial efforts focused on behav-
ioral code generation, achieving modest functional accuracy
while neglecting physical efficiency. The second phase en-
hanced verification through multi-agent approaches, improving
correctness but still targeting high-level RTL rather than gate-
level optimization. Recent frameworks have begun incorporat-
ing physical optimization, yet continue generating behavioral
code that yields suboptimal gate-level implementations.
This evolution mirrors compiler development, progressing
from simple translation to sophisticated multi-pass optimiza-
tion. However, a critical insight remains overlooked: hardware
design requires specialized reasoning processes that a single-
agent LLM fundamentally cannot replicate. The solution to the
Boolean Optimization Barrier lies not in larger models or more
domain-specific datasets, but in rethinking the fundamental
architecture of AI-based hardware design systems.
We introduce CircuitMind, a novel framework that dis-
tributes complex gate-level reasoning across specialized agents
that mirror expert design teams, and TC-Bench, a benchmark
that leverages collective intelligence from competitive circuit
designers to provide human-aligned evaluation. Together, these
innovations establish a new paradigm where collaborative AI
systems overcome individual reasoning limitations through
architectural innovation rather than through greater scale or
additional training data.
arXiv:2504.14625v3
[cs.AR]
1
May
2025

## Page 2

Our key contributions include:
• Architectural Innovation: A hierarchical six-agent
framework that overcomes the Boolean Optimization
Barrier by distributing complex reasoning tasks across
specialized roles, enabling sophisticated gate-level opti-
mization without specialized netlist training.
• Syntax Locking (SL): A constraint mechanism restrict-
ing generation to basic logic gates, eliminating behavioral
abstraction and forcing genuine Boolean reasoning at the
netlist level.
• Dual-Reward Optimization: A framework balancing
functional correctness with physical efficiency, yielding
up to 950% improvement in Solution Efficiency Index
(SEI).
• Human-Competitive Results: CircuitMind enables
55.6% of implementations to match or exceed top-tier
human experts, with our Phi-4 [3] implementation out-
performing both GPT-4o mini [4] and Gemini 2.0 Flash.
• TC-Bench: The first gate-level benchmark with human-
aligned metrics across 28 diverse circuits, establishing
realistic efficiency targets based on competitive human
design expertise.
The integration of CircuitMind with TC-Bench demon-
strates that the efficiency gap between AI-generated and
human-optimized circuits can be significantly narrowed
through intelligent collaboration that overcomes inherent rea-
soning limitations, rather than through larger models or more
training data.
II. BACKGROUND AND PROBLEM DEFINITION
Digital circuit design at the gate level requires balancing
functional correctness with physical efficiency. We define
the problem domain, establish key metrics, analyze AI-based
design approaches, and identify the limitations preventing
human-competitive efficiency.
A. Gate-Level Circuit Optimization
A digital circuit can be represented as a directed acyclic
graph G = (V, E), where vertices V represent logic gates
and edges E represent signal connections. The optimization
objective is to minimize both the total gate count |V | and
critical path delay D(G), while preserving functionality F(G):
min
G
α|V | + βD(G), s.t. F(G) = Fspec, (1)
where α, β are weighting parameters. This NP-hard problem
requires non-local reasoning. Traditional logic synthesis tools
like Yosys [2] and ABC [5] employ heuristic minimization
algorithms and technology mapping to tackle this complexity,
while human experts apply Boolean algebra manipulation,
Karnaugh maps, shared logic identification, and technology
mapping across entire circuit structures.
B. Measuring Design Efficiency
We define the Solution Efficiency Index (SEI) for function-
ally correct implementations:
SEItask =
1
α · G + β · D
(2)
Fig. 1: LLM-based EDA Evolution: From Behavioral to Phys-
ical Design
where G represents gate count and D represents delay. We
use α = β = 1 for balanced evaluation, reflecting common
optimization goals in TuringComplete [1]. However, real-
world circuit optimization often involves complex trade-offs
between area, delay, and power, and the TuringComplete [1]
scoring mechanism might incorporate additional factors like
simulation ticks. For benchmark-level aggregation, we use
geometric mean:
SEIbenchmark = exp
1
n
n
X
i=1
ln(max(SEItask,i, ϵ))
!
(3)
where n is the total number of tasks, SEItask,i is the SEI for
task i (0 if failed), and ϵ = 10−5
ensures that the logarithm
is defined for failed tasks.
C. Evolution of AI in Hardware Design
LLMs in electronic design automation have evolved through
three distinct phases (Fig. 1):
1) Phase 1: Behavioral Code Generation (2022-2023):
Early frameworks like VeriGen [6] and RTLCoder [7] achieved
30-40% functional accuracy on VerilogEval [8] but disregarded
physical efficiency. Analysis of PyraNet [9] shows gate-level
netlists constitute less than 1% of available HDL code, limiting
models’ exposure to optimized patterns.
2) Phase 2: Verification-Enhanced Generation (2023-
2024): Multi-agent frameworks such as MAGE [10] and
AIVRIL [11] increased accuracy to 65-95% through verifica-
tion feedback but operated exclusively at the behavioral RTL
level without addressing physical optimization challenges.
3) Phase 3: Early Physical Optimization (2024-present):
Recent approaches like BetterV [12] and CodeV [13] incorpo-
rate physical considerations but remain limited by generating
behavioral code that synthesis tools must convert to gates. As
Table II shows, while frameworks like RTLSquad [14] use
multi-agent systems for RTL optimization and GenEDA [15]
focuses on reasoning about netlists, no existing framework en-
forces direct gate-level reasoning or aligns with human expert
optimization practices in the same manner as CircuitMind.
The qualitative assessments in Table 2 reflect the primary
focus of each framework; for instance, BetterV [12] uses a
discriminator for optimization guidance, while HiVeGen [16]
targets PPA but still operates at the RTL level.

## Page 3

TABLE II: Comparative Analysis of Verilog Generation
Frameworks
Framework Methodology Gate-Level PPA Human
Enforcement Optimization Alignment
MAGE [10] Multi-Agent No Low No
BetterV [12] Discriminator No Medium No
CodeV [13] Multi-level No Low No
AIVRIL [11] Verification No Low No
PromptV [17] Teacher-Learner No Low No
HiVeGen [16] Hierarchical No High No
RTLSquad [14] Multi-Agent (RTL Opt.) No High No
GenEDA [15] Encoder-Decoder (Netlist Reasoning) No Low No
CircuitMind Multi-Agent Yes (SL) High (DR) Yes (TC-Bench)
D. The Boolean Optimization Barrier
LLM-generated circuits exhibit gate counts 38%–1075%
higher than human-optimized designs due to four fundamental
limitations:
1) Associative vs. Structured Reasoning: LLMs generate
sequences based on statistical patterns rather than perform-
ing the structured symbolic reasoning required for Boolean
optimization. Human experts apply systematic minimization
techniques like Quine-McCluskey algorithms [18] that LLMs
cannot effectively replicate.
2) Local vs. Global Optimization: The autoregressive na-
ture of LLMs forces local, token-by-token decision making
without the ability to revise earlier decisions when later
improvements become apparent. Circuit optimization requires
global restructuring and non-local transformations that are
challenging for current generative models.
3) Abstraction Leakage: LLMs naturally gravitate toward
behavioral abstractions like if-else statements that intro-
duce significant inefficiencies when synthesized to gate-level
netlists, bypassing essential Boolean optimization.
4) Limited Training on Optimized Netlists: The extreme
scarcity of gate-level examples in training data severely limits
LLMs’ exposure to optimized circuit patterns and structures.
E. Benchmark Misalignment
A key challenge in advancing LLM-based hardware de-
sign is the disconnect between evaluation metrics and phys-
ical implementation quality. As shown in Table III, existing
benchmarks have evolved from functional correctness (Verilo-
gEval [8]) to limited PPA metrics (RTLLM [19]) and more
comprehensive quality assessments (ResBench [20], GenBen
[21]), but most lack alignment with human expert perfor-
mance. TC-Bench addresses this gap by leveraging collective
intelligence from TuringComplete’s [1] competitive environ-
ment to establish meaningful gate-level optimization targets
based on actual human performance tiers.
Despite these advancements, TC-Bench offers a unique con-
tribution by harnessing collective intelligence from a compet-
itive gaming environment to establish gate-level optimization
challenges and human performance tiers based on the SEI
metric. This approach provides a distinct perspective focused
on direct gate-level optimization efficiency derived from a
large community actively competing on such tasks. Table III
provides an updated comparison.
F. The Fundamental Reasoning Gap
Existing approaches attempt to bridge the efficiency gap
through three primary strategies, all facing significant lim-
itations. Scale-based approaches fail because the barrier is
fundamentally algorithmic rather than capacity-related; simply
increasing model size does not address the underlying reason-
ing limitations in Boolean optimization. Fine-tuning strategies
are hampered by extreme scarcity of optimized gate-level
examples, creating an insurmountable data bottleneck. Single-
agent verification loops, while improving functional correct-
ness, lack the specialized reasoning capabilities necessary
for effective Boolean optimization and cannot replicate the
structured symbolic manipulation that human experts employ.
Addressing this gap requires a paradigm shift from scaling
individual models to collaborative systems that distribute spe-
cialized reasoning tasks across multiple agents—an approach
that addresses the fundamental limitations in how LLMs
process logical optimization problems.
III. TC-BENCH: HARNESSING COLLECTIVE
INTELLIGENCE
To address the benchmark misalignment problem identi-
fied earlier, we introduce TC-Bench—a gate-level evaluation
framework that leverages collective intelligence from compet-
itive circuit designers on the TuringComplete platform. This
framework establishes meaningful efficiency targets for AI-
generated circuits based on human performance benchmarks.
A. Collective Intelligence Approach
TC-Bench derives its data from TuringComplete, where a
large community competes to optimize digital circuits using
minimal logic gates while meeting functional requirements.
Solutions are ranked based on gate count and performance (de-
lay), generating thousands of expert-refined implementations
that often significantly exceed the efficiency of conventional
synthesis tool outputs. TC-Bench harvests these optimized
designs across player skill levels (Fig. 2(a)), creating a multi-
tiered reference framework that provides diverse, human-
optimized implementations rather than relying on standard
synthesis results.
B. Benchmark Structure
TC-Bench comprises 28 challenges selected based on circuit
complexity, functional diversity, and optimization potential,
stratified into three difficulty levels (Fig. 2(b)):
• Easy (10 challenges): Basic combinational logic imple-
menting fundamental gates and simple selectors, typically
requiring 2-4 gates in expert solutions.
• Medium (11 challenges): More complex components
including adders, multiplexers, and basic ALUs, requiring
5-36 gates in expert solutions.
• Hard (7 challenges): Advanced designs encompassing
multi-bit arithmetic, logic engines, and sequence detec-
tors, requiring 40-250+ gates in expert solutions.
Each challenge includes a functional specification, bench-
marking data from top human designs, and a comprehensive

## Page 4

TABLE III: Comparison of Benchmarks for LLMs in Hardware Design
Benchmark Design Level Key Metrics Data Source Alignment Key Features
TC-Bench(ours) Gate Func. Corr., SEI Game (TuringComplete) Yes (Tiers) Intelligence source, SEI metric
VerilogEval [8] RTL Func. Corr. Academic/Web Limited Focus on functional correctness
RTLLM [19] RTL Func. Corr., PPA Academic/Web Limited PPA metrics, reference designs
ResBench [20] RTL Func. Corr., FPGA Synthetic/Academic No FPGA resource efficiency
GenBen [21] Mixed Func. Corr., QoR, Debug Diverse (Silicon+) Limited QoR focus, contamination handling
RTL-Repo [22] Multi-file RTL Func. Corr. GitHub Limited Focus on multi-file projects
Abbr.: Func. Corr. = Functional Correctness, FPGA = Field-Programmable Gate Array, QoR = Quality of Results,
Fig. 2: TC-Bench’s collective intelligence harvesting process
testbench. Crucially, the benchmark requires direct netlist
construction using only predefined basic logic gates, enforcing
genuine Boolean optimization rather than relying on higher-
level HDL abstractions.
C. Human-Aligned Evaluation Metrics
Building on the evaluation metrics introduced in Section
2.2, TC-Bench employs the Solution Efficiency Index (SEI)
to compare AI-generated circuits against human expertise.
1) Human Reference Tiers: To enable nuanced comparison
against human expertise, we analyzed the aggregated SEI
scores from TuringComplete leaderboard ranks, identifying
three representative performance tiers:
• Top tier (Rank 1-10): Expert optimizers. SEI range:
[0.0951, 0.1252].
• Mid tier (Rank 11-300): Skilled practitioners. SEI range:
[0.0905, 0.0924].
• Low tier (Rank 301-1000): Competent designers. SEI
range: [0.0851, 0.0905].
These tiers establish concrete targets for LLM-based circuit
generation frameworks and provide a meaningful scale for
measuring progress toward human-competitive efficiency. By
positioning AI systems on this human-aligned scale, TC-Bench
enables direct comparison between machine-generated and
expert-optimized implementations in terms that matter to real-
world hardware design.
Fig. 3: CircuitMind System Architecture
IV. CIRCUITMIND: MULTI-AGENT COLLABORATION
To overcome the Boolean Optimization Barrier in circuit
design, we introduce CircuitMind—a hierarchical multi-agent
framework that distributes complex reasoning tasks across
specialized agents, enabling capabilities that individual LLMs
cannot achieve alone.
A. Architectural Overview
CircuitMind adopts a three-tier architecture inspired by
engineering team hierarchies, with six specialized agents dis-
tributed across strategic, coordination, and execution layers:
• Strategic Layer: UserProxy (translates requirements into
formal specifications) and Mediator (orchestrates agent
interactions and resource allocation).
• Coordination Layer: Reviewer (provides PPA-focused
feedback) and Summarizer (extracts and archives opti-
mization patterns).
• Execution Layer: CoderAgent (generates gate-level
netlists) and Executor (performs verification through
compilation and simulation).
This structured decomposition addresses the Boolean opti-
mization challenge by distributing specialized reasoning tasks
across multiple agents, enabling capabilities that a single LLM
cannot achieve. Each agent focuses on a specific aspect of the
design process—specification analysis, code generation, ver-
ification, or optimization—allowing the system to overcome
individual limitations through collaborative problem-solving.
B. Core Technical Innovations
CircuitMind introduces three key innovations that directly
address the fundamental limitations of LLMs in circuit design:

## Page 5

1) Syntax Locking: Syntax Locking (SL) constrains genera-
tion to five basic logic gates (AND, OR, NOT, XOR, NAND),
enforcing genuine Boolean optimization and preventing behav-
ioral abstraction leakage. By implementing a strict context-free
grammar limited to fundamental combinational elements, SL
forces models to perform actual Boolean expression simplifi-
cation rather than relying on high-level abstractions.
This constraint parallels the environment faced by hu-
man designers in TuringComplete, where optimization occurs
through strategic arrangement of basic components. SL is
enforced through multiple mechanisms: explicit netlist format
requirements in prompts, retrieval of optimized netlist-level
examples, and Reviewer verification of gate-level compliance.
2) Retrieval-Augmented Generation: Retrieval-Augmented
Generation (RAG) [23] enables knowledge-driven circuit de-
sign by identifying and reusing optimized subcircuits, allowing
efficient learning from limited examples. The implementation
uses a dynamic knowledge database where successful circuit
patterns are stored, indexed by functionality, and retrieved
during the design process.
When receiving a design task, CoderAgent first classifies
the circuit type (combinational or sequential) and identi-
fies required building blocks. It then queries the knowledge
database for optimized implementations of these components,
incorporating them directly into generation prompts as con-
crete examples of efficient design patterns. This approach
enables the model to leverage past successes while adapting
to new requirements, creating a bootstrapping mechanism that
progressively enhances design capabilities.
The knowledge database evolves as CircuitMind success-
fully completes design tasks, with Summarizer extracting
optimized subcircuits and adding them to the knowledge base.
Initially populated with only basic logic gates, the database
gradually accumulates more sophisticated modules, enabling
the system to tackle increasingly complex designs without
requiring extensive pre-training on gate-level netlists.
3) Dual-Reward Optimization: Dual-Reward Optimization
explicitly balances functional correctness with physical effi-
ciency metrics through dynamic feedback loops. This frame-
work combines two complementary evaluation dimensions:
Functional Correctness Score measures how well the gener-
ated netlist meets specified requirements through comprehen-
sive testing, quantified as the percentage of test cases passed.
Physical Efficiency Score evaluates gate count, critical path
delay, and other physical design metrics against reference
implementations, calculated as a weighted sum adjusted ac-
cording to design priorities.
These dual rewards guide the generation process through
Reviewer feedback targeting both functional issues and in-
efficient structures, CoderAgent prioritization of components
with demonstrated good PPA metrics, and knowledge database
rankings based on efficiency metrics. This approach creates
explicit optimization incentives that mimic the competitive
pressures in the TuringComplete ecosystem, guiding models
toward designs that not only work correctly but approach
human expert efficiency.
Fig. 4: CoderAgent and Reviewer dynamic prompting process
C. Collaborative Workflow
CircuitMind’s collaborative workflow (Fig. 3) begins when
UserProxy receives a circuit design task and converts it into a
structured specification. Mediator routes this specification to
CoderAgent, which initiates a detailed analysis process:
First, CoderAgent determines whether the circuit requires
combinational or sequential logic, guiding which netlist tem-
plates will be retrieved from the knowledge database.
Next, it analyzes overall functionality to identify required
building blocks, querying the knowledge database for netlist-
level representations of these components.
CoderAgent then generates a complete circuit implementa-
tion incorporating these optimized components while adhering
to syntax locking constraints.
Mediator routes the generated netlist to Reviewer, which
performs two-step verification: confirming adherence to
netlist-level requirements and evaluating physical efficiency
metrics.
Executor performs functional verification through compi-
lation and simulation, with results fed back to Reviewer
for potential optimization. This dynamic prompting process
between CoderAgent and Reviewer is illustrated in Fig. 4.
If verification succeeds, Summarizer updates the knowledge
database with newly validated optimized patterns; if errors
occur, Reviewer queries the knowledge database for similar
error patterns and potential solutions, initiating an iterative
debugging process.
This structured collaboration addresses the Boolean opti-
mization challenge by decomposing complex reasoning tasks
across specialized agents. While individual LLMs struggle
with maintaining complex state across optimization steps and
evaluating PPA tradeoffs, CircuitMind’s distributed architec-
ture enables sophisticated reasoning through agent specializa-
tion and coordinated interaction.
D. Knowledge Evolution Mechanism
CircuitMind achieves continuous improvement through a
dual-layer learning architecture:

## Page 6

Short-Cycle Design Refinement involves Reviewer sug-
gesting modifications based on knowledge database insights,
CoderAgent adopting optimized solutions upon successful
verification, and Summarizer logging effective strategies. This
immediate feedback loop enables rapid adaptation within indi-
vidual design tasks, with agents learning from both successes
and failures in real-time.
Long-Term Knowledge Evolution involves systematically
capturing optimization patterns in a structured knowledge
base, allowing identification of common design patterns and
proactive optimization in subsequent projects. The knowledge
acquisition process encompasses experience capture (extract-
ing optimization patterns from successful designs), knowledge
organization (indexing information by functional requirements
and error types), and adaptive reuse (applying historical solu-
tions to new challenges).
This evolutionary approach allows CircuitMind to over-
come data scarcity limitations by creating its own internal
optimization targets based on accumulated design experience.
Rather than requiring extensive gate-level training data, the
system bootstraps its capabilities through iterative refinement
and knowledge accumulation, becoming increasingly capable
with each successfully completed design task.
V. EXPERIMENTAL RESULTS
To evaluate CircuitMind’s effectiveness in generating gate-
level digital circuits, we conducted comprehensive experiments
using the TC-Bench benchmark, comparing various model
configurations against human expert performance tiers.
A. Experimental Setup
We evaluated CircuitMind using a diverse set of LLMs
as backend engines: open-source models (Qwen series [24],
Phi-4 [3], DeepSeek series [25] [26]) and commercial models
(GPT-4o mini [4], Gemini 2.0 Flash [27]). These models pow-
ered the core CoderAgent and Reviewer components, while
Qwen2.5:14B was consistently used for RAG components. For
baseline comparison, we used direct circuit generation by the
same LLMs without the CircuitMind framework. To ensure
fair comparison, we applied consistent conditions across all
experiments: 4-bit quantization for most open-source models
(except DeepSeek), API access for commercial and DeepSeek
models, and a maximum of 2 revision attempts per task.
We evaluated CircuitMind using the Pass@k metric from
[28], defined as:
Pass@k = EProblems
"
1 −
n−c
k

n
k

#
, (4)
where k is samples per task, n is total samples, and c is
correct solutions. With n = 20, we estimated correctness from
correct sample fractions. Pass@k reflects the probability of
solving a task with the best of k samples.
All implementations were synthesized to gate-level netlists
using Yosys 0.34 [2] with standard optimization passes dis-
abled, ensuring results reflect the actual logical structures
TABLE IV: TC-Bench Performance: Base LLMs vs. Circuit-
Mind
Model
Pass@1 SEI
Easy Medium Hard Overall Easy Medium Hard Overall
Gemini 2.0 Flash 0.60 0.53 0.11 0.434 0.21 0.06 0.02 0.063
CM(Gemini-2.0) 0.97 0.76 0.94 0.888 0.17 0.09 0.07 0.102
GPT-4o mini 0.86 0.64 0.13 0.570 0.30 0.07 0.001 0.028
CM(GPT-4o mini) 0.89 0.64 0.82 0.779 0.14 0.09 0.09 0.104
DS-R1 1.00 0.96 0.26 0.775 0.16 0.06 0.01 0.046
CM(DS-R1) 1.00 0.84 0.99 0.936 0.13 0.09 0.03 0.071
DS-V3 0.98 0.82 0.11 0.671 0.23 0.09 0.002 0.035
CM(DS-V3) 1.00 0.82 0.99 0.929 0.11 0.04 0.03 0.051
Phi-4 0.67 0.38 0.02 0.380 0.25 0.07 0.001 0.026
CM(Phi-4) 0.95 0.61 0.26 0.629 0.17 0.10 0.089 0.115
QW:7B 0.45 0.29 0.00 0.263 0.21 0.04 0.00 0.010
CM(QW:7B) 0.49 0.22 0.13 0.288 0.15 0.13 0.06 0.105
QW:14B 0.64 0.36 0.00 0.355 0.28 0.04 0.00 0.010
CM(QW:14B) 0.93 0.36 0.14 0.500 0.14 0.06 0.08 0.088
QWC:14B 0.53 0.38 0.06 0.341 0.18 0.06 0.001 0.022
CM(QWC:14B) 0.80 0.38 0.28 0.500 0.19 0.08 0.06 0.097
QWC:32B 0.67 0.36 0.04 0.377 0.27 0.07 0.001 0.027
CM(QWC:32B) 0.99 0.62 0.32 0.664 0.16 0.09 0.05 0.090
CM: CircuitMind; DS: DeepSeek; QW: Qwen2.5; QWC: Qwen2.5-Coder.
SEI values are averaged across tasks in each category.
Fig. 5: Overall Pass@1 and SEI Comparison: Base LLMs vs.
CircuitMind.
produced by the models rather than the synthesis tool’s ca-
pabilities.
B. Comparative Performance Analysis
Table IV presents the comprehensive performance com-
parison between the baseline LLMs and their CircuitMind-
enhanced counterparts (CM) across the easy, medium, and
hard tasks defined in TC-Bench, evaluated using Pass@1 and
the SEI.
The results, detailed in Table IV and summarized in Fig. 5,
reveal key patterns analyzed below.
1) Functional Correctness and Physical Efficiency En-
hancements: CircuitMind delivers substantial improvements in
both functional correctness and physical efficiency across all
evaluated models. For functional correctness (Fig. 5, top), the
most striking gains appear on hard tasks, where baseline LLMs
typically struggled. Notable improvements include DeepSeek-

## Page 7

Fig. 6: Solved Problem Count by Difficulty.
R1 (0.26 to 0.99, 3.8× increase) and Phi-4 (0.02 to 0.26, 13×
increase) on hard tasks.
For physical efficiency (Fig. 5, bottom), CircuitMind con-
sistently improves overall SEI for all tested models. The
efficiency gains are particularly significant on hard tasks,
with some models showing order-of-magnitude improvements:
CM(GPT-4o mini) achieved 90× higher SEI and CM(Phi-
4) 89× higher SEI compared to their respective baselines.
While some models showed slight SEI decreases on easy
tasks, the substantial improvements on medium and hard tasks
resulted in significant overall efficiency gains, demonstrating
CircuitMind’s effectiveness in complex optimization scenarios.
2) Model-Specific Performance Patterns: Performance
gains varied notably across different models, revealing inter-
esting patterns in framework-model compatibility. CM(Phi-4)
achieved the highest overall SEI (0.115), representing a 342%
increase over its baseline—the largest relative improvement
among all tested models. Commercial models also showed
substantial gains, with CM(GPT-4o mini) and CM(Gemini-
2.0) increasing overall SEI by 271% and 61.9
Interestingly, performance improvements were not uni-
form across difficulty levels. For instance, CM(GPT-4o mini)
showed a 53.3% SEI decrease on easy tasks but dramatic
improvements on harder tasks, resulting in an overall 271%
SEI gain. This suggests that CircuitMind’s effectiveness varies
based on how well each model’s inherent strengths align with
the framework’s collaborative optimization approach.
Fig. 6 confirms these observations, showing that high-
performing implementations like CM(Gemini-2.0) and
CM(DeepSeek-R1/V3) successfully solve problems across all
difficulty categories, with particularly strong performance on
hard tasks that baseline models struggled to address.
C. Human-Competitive Performance Evaluation
To assess CircuitMind’s real-world significance, we com-
pared its performance against the human expert tiers estab-
lished in TC-Bench (Fig. 7).
The results reveal a breakthrough achievement: 55.6% of
CircuitMind implementations (5 out of 9) reach or exceed
top-tier human expert performance. Specifically:
Fig. 7: SEI vs. Model Size Compared to Human Expert Tiers.
• CM(Phi-4) achieved the highest overall SEI (0.115),
firmly placing it in the top-tier human expert range
• Four additional implementations reached top-tier
performance: CM(Qwen2.5:7B) (0.105), CM(GPT-4o
mini) (0.104), CM(Gemini 2.0 Flash) (0.102), and
CM(Qwen2.5-Coder:14B) (0.097, near the boundary)
• The remaining implementations performed comparably to
competent human designers in mid and low tiers
Perhaps most significant is that CM(Phi-4), based on a rela-
tively modest 14B-parameter model, outperformed implemen-
tations using much larger models. This suggests CircuitMind
effectively leverages inherent Boolean reasoning capabilities
rather than relying on model scale. This finding is further
supported by the DeepSeek models, which achieved high
functional correctness but lower efficiency scores (DS-V3:
0.051, DS-R1: 0.071) than several smaller models.
These results indicate that aptitude for Boolean reasoning
and compatibility with the framework’s collaborative opti-
mization process may be more important than model size
or general coding ability for achieving human-competitive
efficiency in gate-level circuit design—a direct validation of
our architectural approach over scale-based solutions.
D. Component Contribution Analysis
To isolate the impact of our key technical innovations, we
conducted an ablation study focused on the RAG components.
We compared three configurations, all maintaining Syntax
Locking and Dual-Reward mechanisms:
• V0: Baseline CircuitMind with RAG disabled
• V1: CircuitMind with only ReviewRAG enabled
• V2: Full CircuitMind with both DesignRAG and Re-
viewRAG enabled
The results in Table V demonstrate RAG’s significant
contribution to CircuitMind’s performance. Four of seven
tested model configurations (DS-R1, 4o-mini, Gemini, Phi-4)
showed progressive SEI improvements from V0 to V1 to V2,
confirming cumulative benefits from each RAG component.
Most dramatically, the full RAG system (V2) improved SEI
by 137% for Gemini (0.043 to 0.102) and 38.6% for Phi-4
(0.083 to 0.115) compared to the baseline without RAG (V0).

## Page 8

TABLE V: Ablation Study: Impact of RAG Components.
Model Pass@1 Pass@5 SEI
V0 V1 V2 V0 V1 V2 V0 V1 V2
Commercial Models
DS-V3 .936 .929 .964 .969 .954 .990 .068 .062 .051
DS-R1 .966 .955 .968 .993 .964 .997 .036 .054 .071
4o-mini .750 .900 .779 .909 .929 .909 .052 .095 .104
Gemini .904 .911 .888 .938 .929 .944 .043 .073 .102
Open-Source Models (14B parameters)
Phi-4 .643 .670 .629 .836 .852 .823 .083 .096 .115
QwenC:14B .421 .484 .500 .761 .822 .805 .061 .103 .097
Qwen:14B .452 .468 .500 .621 .655 .752 .075 .062 .088
While not all models showed strictly monotonic improve-
ment patterns, the overall trend confirms that retrieval mech-
anisms significantly enhance optimization capabilities, partic-
ularly for complex circuits where efficient pattern recognition
is crucial.
This study supports the complementary nature of our tech-
nical innovations. Syntax Locking establishes the gate-level
focus, Dual-Reward directs optimization, and RAG enhances
performance by leveraging successful design patterns from the
knowledge base. The synergy appears particularly effective
for models like Phi-4, where the full V2 system achieves
significantly higher efficiency than configurations with partial
or no RAG.
E. Case Study: Boolean Optimization
TABLE VI: Case Study: Logical Device Implementation Com-
parison
Base Model CircuitMind
Model Gates Delay SEI Gates Delay SEI
GPT-4o mini 96 12 0.0093 13 8 0.0476
DeepSeek-V3 96 12 0.0093 8 2 0.1000
Phi-4 101 16 0.0085 34 8 0.0238
A medium-difficulty logical device implementation case
study provides concrete evidence of how CircuitMind over-
comes the Boolean Optimization Barrier (Table VI). Baseline
LLMs consistently produced highly redundant circuits with
poor efficiency (e.g., 96 gates with SEI 0.009), exemplifying
the limitations described in Section 4.4.
CircuitMind dramatically transformed these implementa-
tions:
• CM(DeepSeek-V3) reduced the circuit from 96 gates to
just 8 gates, achieving a 975% SEI improvement (0.0093
to 0.1000)
• CM(GPT-4o mini) improved efficiency by 412%, cutting
gates from 96 to 13
• CM(Phi-4) achieved a 180% SEI improvement, reducing
complexity from 101 gates to 34
These results directly demonstrate how CircuitMind guides
sophisticated Boolean reasoning, transforming inefficient
structures into optimized designs through shared logic iden-
tification and Boolean simplification. Most importantly, these
improvements were achieved without explicit gate-level train-
ing, validating our central thesis that architectural innovation
in reasoning distribution can overcome fundamental LLM
limitations that scale alone cannot address.
VI. CONCLUSION
This research addressed the critical efficiency gap in LLM-
based hardware design, identifying the Boolean Optimiza-
tion Barrier and benchmark misalignment as key obstacles.
Our proposed multi-agent framework, CircuitMind, integrated
with the human-aligned evaluation capabilities of TC-Bench,
demonstrated a significant advancement in tackling these chal-
lenges.
Experimental results clearly show that our approach sub-
stantially improves both functional correctness and, crucially,
the physical efficiency (SEI) of AI-generated circuits across
diverse LLMs. Achieving human-competitive performance in
55.6% of implementations, notably with a smaller model like
Phi-4 outperforming larger ones, underscores a core finding:
intelligent collaborative architecture and targeted optimization
strategies, as embodied in CircuitMind, are more pivotal than
raw model scale for overcoming gate-level reasoning limita-
tions. TC-Bench proved essential in quantifying this progress
against realistic human expert benchmarks.
While current work focuses on gate count and delay, lim-
itations remain regarding power optimization, industrial con-
straints, and knowledge base scalability. Future efforts should
aim to incorporate these broader physical design considera-
tions and enhance the collaborative mechanisms for complex,
real-world scenarios.
In conclusion, this work validates that structuring AI sys-
tems for collaborative, specialized reasoning, guided by bench-
marks reflecting human expertise, offers a powerful pathway
to achieving human-level performance in complex engineering
optimization tasks like digital circuit design. This paradigm
shift emphasizes architectural innovation and holds promise
for advancing AI-driven automation in hardware and poten-
tially other physically constrained domains.
REFERENCES
[1] LevelHead, “Turingcomplete game,” 2021, https://turingcomplete.game/
Accessed: 2025-04-21.
[2] C. Wolf, “Yosys open synthesis suite,” 2016.
[3] M. Abdin, J. Aneja, H. Behl, S. Bubeck, R. Eldan, S. Gunasekar,
M. Harrison, R. J. Hewett, M. Javaheripi, P. Kauffmann et al., “Phi-
4 technical report,” arXiv preprint arXiv:2412.08905, 2024.
[4] J. Achiam, S. Adler, S. Agarwal, L. Ahmad, I. Akkaya, F. L. Aleman,
D. Almeida, J. Altenschmidt, S. Altman, S. Anadkat et al., “Gpt-4
technical report,” arXiv preprint arXiv:2303.08774, 2023.
[5] R. Brayton and A. Mishchenko, “Abc: An academic industrial-strength
verification tool,” in Computer Aided Verification: 22nd International
Conference, CAV 2010, Edinburgh, UK, July 15-19, 2010. Proceedings
22. Springer, 2010, pp. 24–40.
[6] S. Thakur, B. Ahmad, H. Pearce, B. Tan, B. Dolan-Gavitt, R. Karri, and
S. Garg, “Verigen: A large language model for verilog code generation,”
ACM Transactions on Design Automation of Electronic Systems, vol. 29,
no. 3, pp. 1–31, 2024.
[7] S. Liu, W. Fang, Y. Lu, J. Wang, Q. Zhang, H. Zhang, and Z. Xie, “Rtl-
coder: Fully open-source and efficient llm-assisted rtl code generation
technique,” IEEE Transactions on Computer-Aided Design of Integrated
Circuits and Systems, 2024.

## Page 9

[8] M. Liu, N. Pinckney, B. Khailany, and H. Ren, “Verilogeval: Evaluating
large language models for verilog code generation,” in 2023 IEEE/ACM
International Conference on Computer Aided Design (ICCAD). IEEE,
2023, pp. 1–8.
[9] B. Nadimi, G. O. Boutaib, and H. Zheng, “Pyranet: A multi-layered
hierarchical dataset for verilog,” arXiv preprint arXiv:2412.06947, 2024.
[10] Y. Zhao, H. Zhang, H. Huang, Z. Yu, and J. Zhao, “Mage: A
multi-agent engine for automated rtl code generation,” arXiv preprint
arXiv:2412.07822, 2024.
[11] H. Sami, P.-E. Gaillardon, V. Tenace et al., “Aivril: Ai-driven rtl gen-
eration with verification in-the-loop,” arXiv preprint arXiv:2409.11411,
2024.
[12] Z. Pei, H.-L. Zhen, M. Yuan, Y. Huang, and B. Yu, “Betterv: Con-
trolled verilog generation with discriminative guidance,” arXiv preprint
arXiv:2402.03375, 2024.
[13] Y. Zhao, D. Huang, C. Li, P. Jin, Z. Nan, T. Ma, L. Qi, Y. Pan, Z. Zhang,
R. Zhang et al., “Codev: Empowering llms for verilog generation
through multi-level summarization,” arXiv preprint arXiv:2407.10424,
2024.
[14] B. Wang, Q. Xiong, Z. Xiang, L. Wang, and R. Chen, “Rtlsquad: Multi-
agent based interpretable rtl design,” arXiv preprint arXiv:2501.05470,
2025.
[15] W. Fang, J. Wang, Y. Lu, S. Liu, and Z. Xie, “Geneda: Unleashing
generative reasoning on netlist via multimodal encoder-decoder aligned
foundation model,” arXiv preprint arXiv:2504.09485, 2025.
[16] J. Tang, J. Qin, K. Thorat, C. Zhu-Tian, Y. Cao, C. Ding et al., “Hivegen–
hierarchical llm-based verilog generation for scalable chip design,” arXiv
preprint arXiv:2412.05393, 2024.
[17] Z. Mi, R. Zheng, H. Zhong, Y. Sun, and S. Huang, “Promptv: Leveraging
llm-powered multi-agent prompting for high-quality verilog generation,”
arXiv preprint arXiv:2412.11014, 2024.
[18] W. V. Quine, “The problem of simplifying truth functions,” The Amer-
ican mathematical monthly, vol. 59, no. 8, pp. 521–531, 1952.
[19] Y. Lu, S. Liu, Q. Zhang, and Z. Xie, “Rtllm: An open-source benchmark
for design rtl generation with large language model,” in 2024 29th Asia
and South Pacific Design Automation Conference (ASP-DAC). IEEE,
2024, pp. 722–727.
[20] C. Guo and T. Zhao, “Resbench: Benchmarking llm-generated fpga
designs with resource awareness,” arXiv preprint arXiv:2503.08823,
2025.
[21] G.-W. Wan, S. Wong, M. Xing, N. Guan, N. Xu, Q. Xu, X. Wang et al.,
“Genben: A genarative benchmark for llm-aided design.”
[22] A. Allam and M. Shalan, “Rtl-repo: A benchmark for evaluating llms
on large-scale rtl design projects,” in 2024 IEEE LLM Aided Design
Workshop (LAD). IEEE, 2024, pp. 1–5.
[23] P. Lewis, E. Perez, A. Piktus, F. Petroni, V. Karpukhin, N. Goyal,
H. Küttler, M. Lewis, W.-t. Yih, T. Rocktäschel, S. Riedel,
and D. Kiela, “Retrieval-augmented generation for knowledge-
intensive nlp tasks,” in Advances in Neural Information Processing
Systems, H. Larochelle, M. Ranzato, R. Hadsell, M. Balcan, and
H. Lin, Eds., vol. 33. Curran Associates, Inc., 2020, pp. 9459–
9474. [Online]. Available: https://proceedings.neurips.cc/paper files/
paper/2020/file/6b493230205f780e1bc26945df7481e5-Paper.pdf
[24] A. Yang, B. Yang, B. Zhang, B. Hui, B. Zheng, B. Yu, C. Li, D. Liu,
F. Huang, H. Wei et al., “Qwen2. 5 technical report,” arXiv preprint
arXiv:2412.15115, 2024.
[25] A. Liu, B. Feng, B. Xue, B. Wang, B. Wu, C. Lu, C. Zhao, C. Deng,
C. Zhang, C. Ruan et al., “Deepseek-v3 technical report,” arXiv preprint
arXiv:2412.19437, 2024.
[26] D. Guo, D. Yang, H. Zhang, J. Song, R. Zhang, R. Xu, Q. Zhu, S. Ma,
P. Wang, X. Bi et al., “Deepseek-r1: Incentivizing reasoning capability
in llms via reinforcement learning,” arXiv preprint arXiv:2501.12948,
2025.
[27] DeepMind, “Gemini 2.0 flash experimental.” 2024, https://deepmind.
google/technologies/gemini/flash Accessed: 2024-12-25.
[28] M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. D. O. Pinto, J. Kaplan,
H. Edwards, Y. Burda, N. Joseph, G. Brockman et al., “Evaluating large
language models trained on code,” arXiv preprint arXiv:2107.03374,
2021.

