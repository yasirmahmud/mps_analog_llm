---
title: "VLM-CAD: VLM-Optimized Collaborative Agent Design Workflow for Analog Circuit Sizing"
authors: "Pan, Guanyuan; Wang, Shuai; Lin, Yugui; Zhou, Tiansheng; Liò, Pietro; Zhao, Zhenxin; Wang, Yaqi"
date: "2026/01/12"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2601.07315"
pdf: "papers/2026_vlm-cad-vlm-optimized-collaborative-agent-design-workflow-for-analog-circui_2601-07315.pdf"
sha256: "8af7d563a4ef3694b8f5625db7485b25bc4322cd0c8a6333a37522a11169e054"
pages_extracted: 13
---

# VLM-CAD: VLM-Optimized Collaborative Agent Design Workflow for Analog Circuit Sizing

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

VLM-CAD: VLM-Optimized Collaborative Agent
Design Workflow for Analog Circuit Sizing
Guanyuan Pan
Hangzhou Dianzi University, China
Shuai Wang∗
Hangzhou Dianzi University, China
Yugui Lin
SCBC, Guangdong University
of Foreign Studies, China
Tiansheng Zhou
Hangzhou Dianzi University, China
Pietro Liò
University of Cambridge,
United Kingdom
Zhenxin Zhao
Hangzhou Dianzi University, China
Yaqi Wang∗
Hangzhou Dianzi University, China
Abstract
Vision Language Models (VLMs) have demonstrated remarkable potential in
multimodal reasoning. However, they can have spatial blindness and logical hallu-
cinations when interpreting densely structured engineering content, such as analog
circuit schematics. To address these challenges, we propose a Vision Language
Model-Optimized Collaborative Agent Design Workflow for Analog Circuit Sizing
(VLM-CAD) designed to support step-by-step reasoning over multimodal evidence.
VLM-CAD bridges the modality gap by integrating a neuro-symbolic structural
parsing module, Image2Net, which transforms raw pixels into explicit topologi-
cal graphs and structured JSON representations to anchor VLM interpretation in
deterministic facts. To ensure the reliability required for engineering decisions,
we further propose ExTuRBO, an Explainable Trust Region Bayesian Optimiza-
tion method. ExTuRBO employs agent-generated semantic seeds to warm-start
local searches and uses Automatic Relevance Determination to provide sensitivity
evidence for the final design report. Experimental results on 12 sizing tasks cov-
ering six circuits and four technology platforms show that VLM-CAD achieves a
pooled Strict Pass@1 of 23.3% and a Relaxed Pass@1 of 91.7%, while providing
sensitivity evidence for final design reports.
Keywords: Agentic AI, Vision Language Model, Explainability, Analog Circuit Sizing, Electronic
Design Automation
1 Introduction
Analog and mixed-signal (AMS) integrated circuits are essential components in modern electronic
systems. However, the design process, particularly circuit sizing, remains a significant challenge.
Analog circuit sizing requires navigating high-dimensional spaces to trade off Power, Performance,
and Area to achieve optimal circuit performance. Expert designers use their intuitive understanding
∗Corresponding Authors: Shuai Wang (shuaiwang.tai@gmail.com) and Yaqi Wang (wangyaqi@hdu.edu.cn).
40th Conference on Neural Information Processing Systems (NeurIPS 2026). Workshop: AI for Chip Design.
arXiv:2601.07315v5
[cs.MA]
17
Sep
2026

## Page 2

Advisor Reviewer
Equipped Sizer
DC-OP SPICE
Simulation
Full SPICE Simulation
Universal Cost
Function
ExTuRBO
Circuit Explainer
Matching Finder
DC Goal Setter
Initial Designer
A. Neuro-Symbolic Schematic Parsing via Image2Net B. Cross-Modal Circuit Netlist
and Specifications Analysis
C. DC-OP Sizing
DC Reviewer
DC Sizer
DC-OP SPICE
Simulation
D. Inference-Only
Sizing
Specs Reviewer
Language Model
Inferencing Sizer
DC-OP SPICE
Simulation
Full SPICE Simulation
Universal
Cost Function
E. ExTuRBO Sizing
and Explaining
Annotated schematic
Schematic JSON Description
Original schematic
Specs requirements
Sizing parameter ranges
Netlist
Specs requirements
Netlist
Original schematic
.include '180nm_bulk.txt’
.param cl_value = 10p
.param rl_value = 1k
.param vdd = 1.8
M3 d3 in2 midp vdd pmos W={w_m3} L={l_m3}
M4 d4 in1 midp vdd pmos W={w_m4} L={l_m4}
M6 midp bias1 vdd vdd pmos W={w_m6} L={l_m6}
……
| Specs | Target | Priority |
| :------------------- | :------------------------ | :------------ |
| DC Gain | > 65 dB | High |
| Phase Margin | > 50° | High |
| UGBW (GBW) | > 10 MHz | Medium |
| THD | < -26 dB | Medium |
| Offset | < 1 mV | Medium |
| Power | 0 mW < P < 10 mW | Low |
|Parameter | Type | Range (Min, Max) |
| :-------------| :-------- | :----------------------- |
| W_M3 | Width | [0.18u, 400.0u] |
| L_M3 | Length| [0.18u, 18.0u] |
| Cc | Cap | [1.0p, 10.0p] |
| Vbias1 | Voltage| [0.0V, 1.8V] |
......
[{"component_type": "PMOS",
"port_connection": {
"Drain": "n0",
"Source": "n1",
"Gate": "n0"
}},{"component_type": "NMOS",
"port_connection": {
"Drain": "n2",
"Source": "n3",
"Gate": "n4"}},……]
Sizing parameter ranges
Netlist
Circuit Explanation
This is a technical analysis of the provided NGSPICE
netlist and schematic, which describes a .......
### 1. Input Stage: .......
### 2. Intermediate Stage: ......
### 3. ......
### Summary of Topology ......
Below are the identified matching groups:
### 1. Group: NMOS Input Differential Pair ......
### 2. Group: PMOS Input Differential Pair ......
---
### Summary Table for Layout
Engineering ......
Final Note: .......
Circuit Matching Info
DC Goals
### 1. Global DC Strategy
Supply Voltage: .......
Current Distribution: ......
Inversion Level: ......
### 2. Transistor-Level DC Specifications
#### A. Complementary Input Stage......
……
Initial Parameters
"cl_value": “…",
"rl_value": “…",
"vdd": “…",
"cc": “…",
"l_m1": “…",
"l_m10": “…",
"l_m11": “…",
"l_m12": “…",
Spec requirements
Netlist
DC Design Iterations
DC feedback
vdd
……
@M15[vds]
……
@M17[id]
……
@M14[vdsat]
……
v(vdd)
……
@M10[gm]
……
@M6[vth]
……
@M9[vds]
……
@M13[vgs]
……
@M6[vdsat]
……
@M8[vdsat]
……
@M11[vth]
……
Iter 1:
"cc": “…",
"l_m1": “…",
"l_m10": “…",
"l_m11": “…",……
…...
Iter X:
"cc": “…",
"l_m1": “…",
"l_m10": “…",
"l_m11": “…",……
……
Sizing parameter ranges
Netlist
Full Specs Design Iterations
Full Specs Design Feedback
--- Iteration 1 ---
Running Simulation on node '...' Done.
metrics: G=..., BW=..., PM=..., DC=... | Cost=...
Refining...
Params updated. Changed …… parameters.
……
--- Iteration X ---
Running Simulation on node '...' Done.
metrics: G=..., BW=..., PM=..., DC=... | Cost=...
CONVERGED: Universal Cost ... < ...
Iter 1:
"cc": “…",
"l_m1": “…",
"l_m10": “…",
"l_m11": “…",……
…...
Iter X:
"cc": “…",
"l_m1": “…",
"l_m10": “…",
"l_m11": “…",……
……
Graph Construction
Component Detection
Corner Detection
Sizing parameter ranges
Figure 1: VLM-CAD overview. We include five stages for VLM-CAD: A. Neuro-Symbolic Schematic
Parsing via Image2Net (see Sec. 3.1), B. Cross-Modal Circuit Netlist and Specifications Analysis (see
Sec. 3.2), C. DC-OP Sizing (see Sec. 3.3), D. Inference-Only Sizing (see Sec. 3.4) and E. ExTuRBO
Sizing and Explaining (see Sec. 3.5).
of circuit schematics to make decisions. However, existing automatic analog circuit sizing approaches
primarily depend on textual netlists, creating a cognitive gap as netlists provide only textual descrip-
tions of components and their connections, lacking the global topological context and spatial cues
that schematics offer for high-level functional reasoning [1].
Vision Language Models (VLMs) have recently demonstrated exceptional capabilities for multimodal
reasoning, with the potential to bridge between visual schematics and logical design decisions.
However, applying VLMs to automatic analog circuit sizing exposes two critical challenges:
• Spatial Blindness in Dense Visual Topologies: Circuit schematics are intersection-heavy and
structurally dense. State-of-the-art (SOTA) VLMs exhibit a critical spatial blindness in such
contexts, struggling with low-level tasks like identifying line junctions or counting parallel com-
ponents [2; 3]. This failure to accurately map visual geometric relationships to logical functional
blocks leads to topological hallucinations, making pure neural-based language model reasoning
unreliable for complex circuit analysis [4].
• Lack of Quantified Explainability: Industrial-grade design requires rigorous justification for
every transistor size. While using language models for automatic analog circuit sizing offers some
level of explainability [5], this is questionable due to their tendency to hallucinate, often lacking
a ground-truth link to mathematical or physical reality [6; 7; 8]. Without a quantified sensitivity
analysis that clarifies the relationship between spatial visual features and performance metrics,
automatic analog circuit sizing methods remain experimental curiosities rather than practical tools
for production.
To overcome these challenges:
1. We propose VLM-CAD, a VLM-optimized collaborative agent workflow for robust multi-
modal reasoning in analog circuit sizing. VLM-CAD follows the step-by-step multimodal
reasoning process used by expert designers. By integrating visual topology with textual
2

## Page 3

Table 1: Comparison of VLM-CAD features with SOTA methods. VLM-CAD incorporates schematic
and netlist multimodality, LLM with ARD for quantified explainability, and LLM with ExTuRBO for
Collaborative Semantic Warm-Start initialization strategy.
Framework
Reasoning
Modality
Topology
Awareness
Explainability
Initialization
Strategy
LLMACD [9] Netlist (Text) Netlist Inference LLM CoT
(Qualitative)
LLM
AnaFlow [5] Netlist (Text) Netlist Inference LLM (Qualitative) LLM + Black-box Optimizer
EESizer [10] Netlist (Text) Netlist Inference LLM CoT
(Qualitative)
LLM
VLM-CAD (Ours) Schematic +
Netlist
Schematic +
Netlist Inference
LLM + ARD
(Quantified)
LLM + ExTuRBO (Collaborative
Semantic Warm-Start)
specifications, VLM-CAD deduces latent constraints and performs hierarchical reasoning
from DC biasing to full-spec optimization.
2. We integrate Image2Net, a neuro-symbolic structural parsing framework. Image2Net ad-
dresses VLM spatial blindness by extracting structured representations from schematics and
providing a robust visual memory that anchors the model’s reasoning in precise topological
facts.
3. We propose ExTuRBO, an Explainable Trust Region Bayesian Optimization method. Ex-
TuRBO collaboratively warm-starts local search to accelerate convergence and utilizes
Automatic Relevance Determination (ARD) lengthscales to provide simulation-grounded,
dual-granularity sensitivity evidence for the final design report.
We evaluate VLM-CAD on 12 sizing tasks across six circuits and four technology platforms, improv-
ing pooled Strict Pass@1 from 3.3% to 23.3% relative to the baseline, with 91.7% Relaxed Pass@1.
We demonstrate VLM-CAD in Figure 1 and compare it with SOTA methods in Table 1.
2 Related Work
2.1 Automatic Circuit Sizing: From ML to Agents
Automatic analog circuit sizing has evolved from traditional optimization to modern agentic work-
flows. Early efforts primarily used Machine Learning (ML) methods, such as Bayesian Optimization
and Reinforcement Learning, to explore high-dimensional design spaces [11]. However, these black-
box approaches relied exclusively on textual netlists, lacking the high-level functional hierarchies,
such as symmetric differential pairs or feedback loops, that are immediately apparent in a visual
schematic. Therefore, they unthinkingly explore the design space without understanding the structural
design intent, often leading to simulation-intensive, unintuitive search processes [12; 13].
Recently, Large Language Models (LLMs) have demonstrated their capabilities to optimize analog
circuits. LEDRO [14] utilized LLMs to prune high-dimensional search spaces through design
space reduction, while LLMACD [9] incorporated circuit behavior knowledge into prompt-based
sizing. To enhance autonomous decision-making, multi-agent frameworks like AmpAgent [15]
and AnaFlow [5] replicated expert design hierarchies and decomposed complex design tasks into
subtasks, such as topology selection and parameter refinement. However, textual LLMs struggle to
accurately reason about AMS circuit topologies [8], leading to sub-optimal trade-offs. Furthermore,
the inherent lack of mathematical precision in LLMs [16] and their susceptibility to hallucination [6]
underscore the critical need for multimodal alignment: integrating visual topological context with
textual specifications and numerical simulation feedback to ensure logical consistency.
2.2 Schematic Understanding and Spatial Reasoning in VLMs
Circuit schematics visually depict the structure and interconnections of a circuit, providing design-
ers with an intuitive reference for understanding how it operates [1]. However, automatic circuit
schematics interpretation represents a challenging domain for dense diagrammatic reasoning. While
3

## Page 4

VLMs have shown promise in general image captioning, their application to structured engineering
documents is still in its infancy. MAPS [17] finetunes a VLM to convert circuit schematics into SPICE
netlists, while CURVLM [18] finetunes a VLM for analytical reasoning on digital circuit schematics.
Despite these advancements, purely neural-based VLM architectures face fundamental bottlenecks in
high-precision spatial tasks. Even SOTA VLMs exhibited persistent spatial blindness, struggling with
reliable distance estimation and direction comprehension in dense layouts [19], leading to topological
hallucinations, where the model misinterpreted line intersections or miscounts recurring geometric
shapes [2; 20; 3]. In analog design, even a single pixel of connectivity error can invalidate the entire
physical logic. Therefore, purely neural-based reasoning remains unreliable [4; 7].
3 VLM-CAD Agentic Workflow
3.1 Neuro-Symbolic Schematic Parsing via Image2Net
Despite excellent performance on object detection and general multimodal reasoning tasks, VLMs
inherently struggle with spatial blindness in densely structured diagrams, often failing to correctly
count intersections or map pixel-level geometric relationships to logical functional blocks, leading to
topological hallucinations. To address this, we utilize a neuro-symbolic schematic parsing approach.
Specifically, we employ Image2Net [21] to transform raw pixel data into an explicit symbolic
representation, thereby anchoring the VLM’s high-level reasoning in deterministic topological facts.
• Component Detection: We finetune a YOLOv8-Pose model [22] on the train dataset from [21],
which contains 2268 noise-free circuit schematics, and employ it as the neural front-end to identify
component bounding boxes and predict terminal pose coordinates. This neural layer handles the
visual variety of schematic symbols, translating visual primitives into identifiable objects.
• Corner Detection: To capture the wiring topology, we apply deterministic feature detection
algorithms followed by Density-Based Spatial Clustering of Applications with Noise clustering [23].
By merging duplicate detections into precise geometric nodes, we mitigate the noise inherent in
neural-only detection.
• Graph Construction: We transform pixel-level visual continuity into an explicit connectivity
graph through a hybrid algorithm: vectorized array slicing validates orthogonal connections, while
Bresenham’s algorithm traces diagonal paths. By clustering physically connected nodes into
distinct electrical nets, we provide a symbolic abstraction of the visual topology.
• Annotated Schematic and JSON Description Generation: Finally, we map the parsed graph to a
structured symbolic JSON description. This description serves as an explicit, topology-grounded
visual memory for the VLM, helping the subsequent agentic design workflow maintain consistent
access to the parsed schematic structure. Additionally, we create an annotated schematic that
color-codes the distinct electrical connections for verification.
We present the details of this phase in Figure 1.A.
3.2 Cross-Modal Circuit Netlist and Specifications Analysis
After annotating the schematic, we analyze the sizing problem using four specialist agents:
• Circuit Explainer: ingests the schematic, netlist, and specifications to identify analog sub-blocks,
deduce signal flow and feedback, and produce a functional circuit description.
• Matching Finder: identifies transistor clusters requiring symmetry and emits parameter-level
equality constraints. It also infers layout implications such as interdigitation needs.
• DC Goal Setter: assigns each transistor quantitative operating targets: overdrive voltage Vov,
drain-source voltage VDS, and current density. It also defines a global biasing plan that reserves
sufficient headroom for signal swing and includes the start-up conditions required for convergence.
• Initial Designer: parses the netlist to extract all mandatory design variables and populates each key
with a conservative executable value aligned with the DC goals, mitigating parameter hallucination
and omission.
We present the details of this phase in Figure 1.B and 2.
4

## Page 5

3.3 DC-OP Sizing
VLM-CAD first establishes a DC-biased solution before costly performance simulations:
• DC Reviewer: runs a fast DC operating-point simulation at each iteration, compares node volt-
ages and device operating points against the DC goals from previous analysis, and computes a
discrepancy count as the loop stopping criterion.
• DC Sizer: updates parameters from the discrepancy report using analog design heuristics while
preserving static testbench values, matching constraints, and physical bounds across iterations.
We present the details of this phase in Figure 1.C.
3.4 Inference-Only Sizing
After DC-OP sizing, VLM-CAD optimizes all specifications through full simulations, relying solely
on the inherent knowledge of analog circuit theory provided by language models without any external
numerical optimizers.
We convert the multi-objective sizing task into a scalar Universal Cost Function J(x) shared by the
language model agents and the numerical optimizer. It has two modes: Feasibility, which prioritizes
satisfying all specifications except power, and Optimization, which minimizes power. For a design
x,
J(x) =
Pmeas(x)
Pmax
+
X
i∈S
wi · Vi(yi(x), Ti) + Psanity. (1)
Here, Pmeas is the simulated power and Pmax its limit; S is the set of performance metrics, split into
lower-bound metrics SLB such as gain and phase margin, and upper-bound metrics SUB such as THD
and offset; wi is the penalty weight of metric i; and Psanity is a large constant penalty applied only
when basic functionality is lost, such as non-convergence or near-zero gain. The violation term Vi
enforces the direction of each constraint:
Vi(yi(x), Ti) =

max(0, Ti − yi(x)), i ∈ SLB
max(0, yi(x) − Ti), i ∈ SUB.
(2)
J(x) separates two operating regimes rather than encoding feasibility by construction: above the
regime boundary the sizer prioritizes driving every specification except power toward its target, and
within the low-cost regime it shifts effort to minimizing power under periodic re-verification of each
specification against its target. In our runs, the boundary was set at J(x) ≤ 1, and a sizing loop
terminated when either J(x) < 0.5 or every hard specification of the task was met by the simulated
response.
We present the details of this phase in Figure 1.D:
• Specs Reviewer: applies a Skip-on-Fail evaluation: it verifies the DC operating point first and halts
early if biasing fails, then runs full DC/AC simulations and evaluates J(x) via (1).
• Language Model Inferencing Sizer: updates parameters from a history-aware context to avoid
oscillation, alternating between satisfying constraints and minimizing power. If a proposal repeats
the previous one, a dead-loop detector applies a small random perturbation to transistor widths to
break the stagnation.
3.5 ExTuRBO Sizing and Explaining
When inference-only sizing reaches its iteration limit or convergence plateau, VLM-CAD escalates to
ExTuRBO, an Explainable Trust Region Bayesian Optimization method that provides explainability
for the final design report in the form of sensitivity evidence, through two features:
• Collaborative Semantic Warm-Start: Standard TuRBO initializes with Latin Hypercube Sam-
pling (LHS) [24] over the full design space, wasting simulation budget on non-functional regions.
ExTuRBO instead seeds its search from the inference-only phase: the Advisor selects up to three
distinct Phase-D candidates from the low-cost regime (J(x) < 1), and each parallel worker starts
from one candidate with locally contracted bounds around it. Relative to a global search over
5

## Page 6

Circuit Explainer
Matching Finder
DC Goal Setter
Initial Designer
B. Cross-Modal Circuit Netlist
and Specifications Analysis
Annotated schematic
Schematic JSON Description
Original schematic
Specs requirements
Sizing parameter ranges
Netlist
[{"component_type": "PMOS",
"port_connection": {
"Drain": "n0",
"Source": "n1",
"Gate": "n0"
}},{"component_type": "NMOS",
"port_connection": {
"Drain": "n2",
"Source": "n3",
"Gate": "n4"}},……]
This circuit is a classic Two-Stage Miller-Compensated
Operational Amplifier (Op-Amp). It is designed to provide
high DC gain and stable frequency response for driving
capacitive loads. ……
2. Feedback and Compensation
• Nulling Resistor (Schematic $R_c$): While the
netlist shows a direct capacitor `Cc`, the
schematic includes a resistor $R_c$ in series with
$C_c$. This is used to eliminate the Right-Half Plane
(RHP) zero, further improving the Phase Margin.
Based on the provided netlist and schematic for the
Two-Stage Miller-Compensated Op-Amp, here are the
required transistor matching groups.
1. Group A: Input Differential Pair
• Transistors: `mn1` and `mn2` (Schematic: M2)
• Required Parameter Matching: ......
• Layout Considerations: ......
• Performance Impact: ......
2. Group B: First-Stage Active Load Mirror
• Transistors: `mp1` and `mp2` (Schematic: M1) ......
3. Group C: NMOS Biasing Network (Current Mirror Tree)
• Transistors: `mn4` (Reference), `mn3` (Tail), and `mn5`
(Output Load) (Schematic: M3, M4, M5) ......
Figure 2: Circuit netlist and specifications analysis in VLM-
CAD. The VLM-based circuit explainer and matching finder
utilize both the circuit schematic and netlist for multimodal
reasoning.
Table 2: Comparative analysis of
multimodal reasoning performance
between VLMs and Image2Net-
grounded parsing.
Method
Non-
Empty
Output
Functional
Accuracy
F (%)
Connectivity
Score Sk
Gemini 3
Flash Preview 122 95.90 0.6600 ± 0.1652
GPT-5.2 119 49.18 0.6096 ± 0.1278
Gemma-4
31B-it 122 52.46 0.5918 ± 0.1202
Qwen3
VL-235B
A22B-Instruct
119 58.20 0.5751 ± 0.1091
Image2Net 122 68.85 0.8121 ± 0.2116
Xglobal ⊂ RD
, this prunes the search to a local subspace Xlocal with span ratio r < 1, so the search
volume shrinks exponentially in the dimension:
Vlocal
Vglobal
≈
D
Y
d=1
rd ≈ rD
. (3)
For example, in a 48-dimensional problem with r = 0.4, we reduce the volume by a factor of
∼ 1019
, substantially reducing global exploration and focusing optimization on high-quality regions
identified by the agents.
• Grounding via Dual-Granularity Sensitivity: After optimization, ExTuRBO fits two ARD
Gaussian processes to produce simulation-grounded sensitivity evidence for the final design report:
Global Sensitivity, fit on all evaluated designs, identifies survival parameters most associated
with basic circuit feasibility, and Elite Sensitivity, fit on the top 15% of designs, identifies tuning
parameters most associated with high performance. We derive feature importance from the ARD
lengthscales as Sd ∝ 1/ℓd: the smaller ℓd, the more strongly the modeled cost depends on
parameter d.
We present the details of this phase in Figure 1.E:
• Advisor Reviewer: analyzes the inference-only history, deduplicates and selects the best distinct
low-cost candidates as seeds, and configures ExTuRBO’s local search bounds.
• Equipped Sizer: runs the parallel ExTuRBO workers on these seeds and converts the Global and
Elite sensitivity results into a design sign-off report that separates stability-critical parameters from
performance-tuning parameters.
4 Experiments
4.1 Multimodal Structural Reasoning Performance
To assess Image2Net-grounded structural parsing, we evaluate schematic parsing on 122 schematics
that contain significant visual noise from [21], which are more visually degraded than those used in
VLM-CAD (see Figure 3). We compare Image2Net with four general-purpose VLMs: Gemini 3
Flash Preview [25], GPT-5.2 [26], Gemma-4-31B-it [27], and Qwen3-VL-235B-A22B-Instruct [28].
Our comparison focuses on functional accuracy F, the percentage of schematics whose predicted
circuit-type label matches the reference, and the connectivity score Sk = 1/ log10(10 + K), where
K is the graph edit cost between the predicted and reference connectivity graphs. The complete
scoring protocol is provided in Appendix A.4.
Results show that the VLMs exhibit clear spatial blindness: functional accuracy varies widely, and
connectivity scores are consistently weaker than Image2Net’s. Common errors include misreading
line crossings as junctions and miscounting parallel transistors, and both GPT-5.2 and Qwen3-VL-
235B-A22B-Instruct return blank or non-terminating outputs on some schematics.
6

## Page 7

However, Image2Net is a specialized neuro-symbolic parser that we train on 2268 schematics and
remains imperfect under novel noise. In contrast, VLMs offer zero-shot semantic reasoning: Gemini 3
Flash Preview attains 95.90% functional accuracy, above Image2Net’s 68.85%. VLM-CAD therefore
offloads deterministic netlist recognition to Image2Net while the VLM performs high-level circuit
reasoning, grounding the agent workflow in explicit connectivity.
4.2 Sizing Results
We evaluate VLM-CAD on six circuit topologies, shown in Figure 3, under two specification sets
per circuit. The study spans discrete device models, PTM 180nm and 45nm models [29], and the
SKY130 technology platform [30].
Table 3: Benchmark circuits, technology platforms, and target specifications. BSIM PTM stands
for Berkeley Short-channel IGFET Model Predictive Technology Models [29]. SKY130 stands for
SkyWater SKY130 PDK [30]. “—” indicates that the metric is not measurable for the given circuit.
For readability, THD is reported as a percentage using THD(%) = 100 × 10THD(dB)/20
.
Circuit Technology Specs
Gain
(dB)
PM
(◦)
UGBW
(MHz)
THD
(%)
Offset
(mV)
Power
(mW)
NMCNR SKY130
SpecsA > 90 > 55 > 2.00 < 0.316% < 1.0 < 1.00
SpecsB > 80 > 60 > 1.00 < 0.562% < 1.0 < 0.15
DFCFC SKY130
SpecsA > 90 > 50 > 3.00 < 0.100% < 0.5 < 2.00
SpecsB > 80 > 45 > 5.00 < 0.316% < 1.0 < 4.00
PFC SKY130
SpecsA > 80 > 50 > 5.00 < 0.316% < 1.0 < 1.50
SpecsB > 70 > 45 > 20.00 < 1.00% < 1.0 < 1.50
Miller Op-Amp
PTM
45 nm
SpecsA > 54 > 60 > 1.00 < 0.100% < 5.0 < 0.15
SpecsB > 48 > 50 > 18.00 < 1.26% < 8.0 < 0.40
Class-AB Op-Amp
PTM
180 nm
SpecsA > 65 > 65 > 1.00 < 5.01% < 0.5 < 0.60
SpecsB > 65 > 50 > 10.00 < 5.01% < 1.0 < 10.00
Discrete Op-Amp
Discrete
MOSFET
SpecsA > 90 > 60 > 0.35 < 0.178% < 0.1 —
SpecsB > 85 > 50 > 0.80 < 0.178% < 0.1 —
Our baseline is EESizer [10], an agent that performs analog sizing through CoT inference over the
netlist. For comparability under each method’s default configuration, all methods utilize Gemma-4-
31B-it [27] as their backbone VLMs, with five independent runs per task. Following [31], for a task
with c successful runs out of n = 5, we compute
Pass@k = 1 −
n−c
k

n
k
 . (4)
We count a run as Strict when it satisfies all performance and power constraints, and as Relaxed when
the power constraint is disregarded and at most one other specification is violated. We report 95%
confidence intervals for pooled Pass@1 and Pass@3, obtained by a task-level cluster bootstrap over
the twelve tasks.
We report the per-task Pass@1/Pass@3 and pooled statistics in Table 4. VLM-CAD improves pooled
Strict Pass@1 over EESizer from 3.3% [0.0, 8.3] to 23.3% [10.0, 36.7] and Relaxed Pass@1 from
15.0% [0.0, 36.7] to 91.7% [80.0, 100.0]. Pass@3 moves in step, rising from 10.0% and 16.7%
to 46.7% and 99.2% under the Strict and Relaxed modes, respectively, and VLM-CAD matches
or exceeds EESizer on every task. For the VLM-CAD trials selected by UC, the corresponding
total runtimes were 2432.52 seconds for the Miller Op-Amp and 389.80 seconds for the Class-AB
Op-Amp, with Phase E accounting for 87.19% and 40.81% of these totals, respectively.
4.3 Ablation Study
To assess the significance of circuit schematics and the assistance of Image2Net, we conduct two
ablations using Gemma-4-31B-it: 1) input only the original schematic, without Image2Net’s annota-
7

## Page 8

(a) NMCNR. (b) DFCFC.
(c) PFC. (d) Miller Op-Amp. (e) Class-AB Op-Amp. (f) Discrete Op-Amp.
Figure 3: Schematic topologies of the six analog circuits we use: (a) a nested Miller-compensated
amplifier with a nulling resistor [32; 33], (b) a damping-factor-control frequency-compensation am-
plifier [32; 33], (c) a two-stage Miller operational amplifier [34], (d) a positive-feedback-compensated
amplifier [32; 35], (e) a complementary-input class-AB operational amplifier [10], and (f) a two-stage
discrete-MOSFET operational amplifier.
Table 4: Pass@1 and Pass@3 (%) of EESizer and VLM-CAD across six circuits and two specifica-
tions. Strict: all metrics meet their targets; Relaxed: at most one metric misses its target with the
power requirement disregarded. Bold indicates the best results. The last row reports 95% confidence
intervals of the pooled Pass@1 and Pass@3, obtained by the same task-level bootstrap.
Circuit Tech Spec
EESizer VLM-CAD
Strict Relaxed Strict Relaxed
P@1 P@3 P@1 P@3 P@1 P@3 P@1 P@3
NMCNR SKY130
SpecsA 0.0 0.0 0.0 0.0 0.0 0.0 100.0 100.0
SpecsB 0.0 0.0 0.0 0.0 0.0 0.0 100.0 100.0
DFCFC SKY130
SpecsA 0.0 0.0 0.0 0.0 0.0 0.0 100.0 100.0
SpecsB 0.0 0.0 0.0 0.0 60.0 100.0 100.0 100.0
PFC SKY130
SpecsA 0.0 0.0 0.0 0.0 40.0 90.0 80.0 100.0
SpecsB 0.0 0.0 0.0 0.0 0.0 0.0 80.0 100.0
Miller
Op-Amp
PTM
45nm
SpecsA 0.0 0.0 0.0 0.0 0.0 0.0 100.0 100.0
SpecsB 0.0 0.0 0.0 0.0 0.0 0.0 40.0 90.0
Class-AB
Op-Amp
PTM
180nm
SpecsA 20.0 60.0 100.0 100.0 60.0 100.0 100.0 100.0
SpecsB 20.0 60.0 80.0 100.0 40.0 90.0 100.0 100.0
Discrete
Op-Amp
Discrete
MOSFET
SpecsA 0.0 0.0 0.0 0.0 40.0 90.0 100.0 100.0
SpecsB 0.0 0.0 0.0 0.0 40.0 90.0 100.0 100.0
Average 3.3 10.0 15.0 16.7 23.3 46.7 91.7 99.2
95% CI
[0.0,
8.3]
[0.0,
25.0]
[0.0,
36.7]
[0.0,
41.7]
[10.0,
36.7]
[22.5,
71.7]
[80.0,
100.0]
[97.5,
100.0]
tion and JSON description; 2) input no schematic information at all. For each paradigm, we run five
trials. Table 5 reports the results: on the Miller Op-Amp, Ablation 1 performs worse and takes longer
than Ablation 2, and both ablations yield higher and more unstable universal costs than VLM-CAD;
on the Class-AB Op-Amp, both ablations meet the specifications, and VLM-CAD generally needs
less total optimization time than the schematic-only ablation, while the no-schematic ablation can be
competitive on easier settings.
Ablation 1 being worse than Ablation 2 is consistent with a negative-transfer effect: the dense,
repetitive schematic may act as a naturally adversarial visual input, consistent with VLM vulnerability
to image-space perturbations [36] (see Figure 5).
8

## Page 9

Table 5: Results and runtime from the trial with the lowest UC among five trials for each method in
the Miller and Class-AB Op-Amp settings shown. Red and blue fonts denote the best and second-best
values for UC and runtime, respectively.
(a) Results for the Miller Op-Amp.
PTM Run
Gain
(≥54dB)
PM
(≥60◦)
UGBW
(≥1MHz)
THD
(≤0.10%)
Offset
(≤5mV)
Power
(≤0.15mW)
UC
Runtime
(s)
45nm
VLM-CAD 53.180 48.703 1.523 0.01269 3.838 68.321 2.405 2432.52
Ablation 1 45.491 179.214 5.960 0.01460 3.529 111.095 9.250 2826.00
Ablation 2 52.783 17.477 10.538 0.01425 2.524 115.607 6.240 2500.28
(b) Results for the Class-AB Op-Amp.
PTM Run
Gain
(≥65dB)
PM
(≥50◦)
UGBW
(≥10MHz)
THD
(≤5.01%)
Offset
(≤1mV)
Power
(≤10mW)
UC
Runtime
(s)
180nm
VLM-CAD 85.505 51.897 35.111 0.001911 0.551 0.810 0.081 389.80
Ablation 1 67.447 111.363 73.455 0.009684 0.138 3.120 0.312 1437.40
Ablation 2 90.765 52.568 13.447 0.00001075 0.018 3.176 0.318 820.34
Sensitivity Analysis Report
Design Sign-off Report: Phase E Optimization
1. Executive Summary
Phase E of the optimization process has successfully concluded. The design has transitioned from a high-
performance baseline in Phase D to a more robust, power-efficient architecture in Phase E. While we observed ......,
the trade-off resulted in ...... The design is now considered "Elite" as it ......
2. Metric Evolution: Phase D vs. Phase E
The transition from Phase D to Phase E represents a strategic shift from ... to ...:
Power Efficiency: ...... Stability (Phase Margin): ...... Gain & Bandwidth: ...... Linearity & Offset: ......
3. Survival Factor Analysis (Global Sensitivity)
The Global Sensitivity data confirms that the design’s "Survival" is heavily dependent on ... These parameters act
as the structural anchors of the circuit—likely governing ...
Because the optimization status is SUCCESS, we can conclude that ......
4. Performance Tuners (Elite Sensitivity Analysis)
With the survival factors stabilized, the Phase E success was driven by the Elite Sensitivity parameters. Unlike the
Global factors, the Elite sensitivities are distributed across a wide range of components, indicating a highly
sophisticated, multi-variable refinement:
Impedance & Current Control:... Symmetry & Matching:... "Squeeze" Effect:...
5. Final Recommendation
The design has met all primary constraints while significantly exceeding power efficiency expectations.
Sign-off Status: APPROVED.
w_m4
w_m10
l_m15
l_m4
Figure 4: Final design report including Global
Sensitivity, Elite Sensitivity, and explanations.
Iteration
0
1000
2000
3000
4000
5000
6000
7000
Universal
Cost
Stability Report (Universal Cost):
-----------------------------------
VLM-CAD : 148.48 ± 184.10
Ablation 2 : 298.30 ± 315.31
Ablation 1 : 855.66 ± 1668.26
VLM-CAD
Ablation 1
Ablation 2
Figure 5: Universal cost distribution of Phase
D for the Miller Op-Amp. Both ablations show
higher and more unstable costs than VLM-CAD,
and ablation 1 performs worse than ablation 2.
4.4 Failure Analysis and Limitations
Strict Pass@1 is zero on six of the twelve tasks. The per-run metrics reveal distinct failure signatures
rather than a single mode: bandwidth limits the Discrete Op-Amp and the Class-AB Op-Amp,
whose UGBW falls just below target while every other metric passes; gain limits DFCFC by a wide
margin; and instability in the form of negative phase margin dominates NMCNR and PFC. The Miller
Op-Amp tasks on 45nm BSIM PTM are the only ones that combine a marginal gain miss with a
phase-margin collapse under the aggressive 18 MHz UGBW target, and Miller Op-Amp SpecsB is
the sole task whose Relaxed Pass@1 falls to 40%. Plausible contributors are the narrow feasible
region imposed by coupled gain, bandwidth, stability, and power constraints, short-channel effects
that weaken long-channel sizing heuristics, and warm-started trust regions entering locally attractive
but infeasible basins. We treat these as hypotheses rather than causal findings.
Image2Net also remains imperfect: with a connectivity score of 0.8121 ± 0.2116, parsing errors can
propagate into Phase B reasoning and later initialization. Finally, although the benchmark spans four
9

## Page 10

technology platforms, it concentrates on analog amplifier topologies, so generalization to ADCs,
PLLs, and power-management blocks remains open.
5 Conclusion
We propose VLM-CAD, a VLM-optimized collaborative agent workflow for analog circuit sizing
that combines deterministic schematic grounding, agentic reasoning, and explainable trust-region
optimization. Across 12 sizing tasks covering six circuits and four technology platforms, VLM-
CAD improves pooled Strict Pass@1 from 3.3% to 23.3% over EESizer, while achieving 91.7%
Relaxed Pass@1. The structural evaluation shows that Image2Net provides stronger connectivity
reconstruction than the evaluated general-purpose VLMs, and end-to-end runtime remains dominated
by the simulation-driven ExTuRBO stage.
Acknowledgments and Disclosure of Funding
This research was supported by National Natural Science Foundation of China under Grant
No. 62571173, Zhejiang Provincial Natural Science Foundation of China under Grant
No. LD25F020005 and No. LQN25F030009.
References
[1] Hung-Yun Hsu and Mark Po-Hung Lin. Automatic analog schematic diagram generation based
on building block classification and reinforcement learning. In 2022 ACM/IEEE 4th Workshop
on Machine Learning for CAD (MLCAD), pages 43–48, 2022. doi: 10.1109/MLCAD55463.
2022.9900093.
[2] Pooyan Rahmanzadehgervi, Logan Bolton, Mohammad Reza Taesiri, and Anh Totti Nguyen.
Vision language models are blind. In Proceedings of the Asian Conference on Computer Vision
(ACCV), pages 18–34, December 2024.
[3] An Vo, Khai-Nguyen Nguyen, Mohammad Reza Taesiri, Vy Tuong Dang, Anh Totti Nguyen,
and Daeyoung Kim. Vision language models are biased. In The Fourteenth International
Conference on Learning Representations, 2026. URL https://openreview.net/forum?
id=DG4S2OlGQA.
[4] Behzad Razavi. Analog design experiments with ai—part 1 [the analog mind]. IEEE Solid-State
Circuits Magazine, 17(4):11–15, 2025. doi: 10.1109/MSSC.2025.3611213.
[5] Mohsen Ahmadzadeh, Kaichang Chen, and Georges Gielen. (invited paper) anaflow: Agentic
llm-based workflow for reasoning-driven explainable and sample-efficient analog circuit sizing.
In 2025 IEEE/ACM International Conference On Computer Aided Design (ICCAD), pages 1–7,
2025. doi: 10.1109/ICCAD66269.2025.11240818.
[6] Yejin Bang, Ziwei Ji, Alan Schelten, Anthony Hartshorn, Tara Fowler, Cheng Zhang, Nicola
Cancedda, and Pascale Fung. HalluLens: LLM hallucination benchmark. In Wanxiang Che,
Joyce Nabende, Ekaterina Shutova, and Mohammad Taher Pilehvar, editors, Proceedings of
the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long
Papers), pages 24128–24156, Vienna, Austria, July 2025. Association for Computational
Linguistics. ISBN 979-8-89176-251-0. doi: 10.18653/v1/2025.acl-long.1176. URL https:
//aclanthology.org/2025.acl-long.1176/.
[7] Zhuofu Tao, Yichen Shi, Yiru Huo, Rui Ye, Zonghang Li, Li Huang, Chen Wu, Na Bai, Zhiping
Yu, Ting-Jung Lin, and Lei He. Amsnet: Netlist dataset for ams circuits. In 2024 IEEE LLM
Aided Design Workshop (LAD), pages 1–5, 2024. doi: 10.1109/LAD62341.2024.10691781.
[8] Lejla Skelic, Yan Xu, Matthew Cox, Wenjie Lu, Tao Yu, and Ruonan Han. Circuit: A benchmark
for circuit interpretation and reasoning capabilities of llms, 2025. URL https://arxiv.org/
abs/2502.07980.
10

## Page 11

[9] Shaojie Xu, Haochang Zhi, Jintao Li, and Weiwei Shan. Llmacd: An llm-based analog circuit
designer driven by behavior parameters. In 2025 International Symposium of Electronics Design
Automation (ISEDA), pages 157–162, 2025. doi: 10.1109/ISEDA65950.2025.11101244.
[10] Chang Liu and Danial Chitnis. Eesizer: Llm-based ai agent for sizing of analog and mixed
signal circuit. IEEE Transactions on Circuits and Systems I: Regular Papers, 73(6):3825–3834,
2026. doi: 10.1109/TCSI.2025.3646359.
[11] Konstantinos G. Liakos and Fotis Plessas. Analog design and machine learning: A review.
Electronics, 14(17), 2025. ISSN 2079-9292. doi: 10.3390/electronics14173541. URL https:
//www.mdpi.com/2079-9292/14/17/3541.
[12] Zhengfeng Wu, Ziyi Chen, Nnaemeka Achebe, Vaibhav V. Rao, Pratik Shrestha, and Ioannis
Savidis. Emerging ml-ai techniques for analog and rf eda, 2025. URL https://arxiv.org/
abs/2506.00007.
[13] M.T.M. Emmerich, K.C. Giannakoglou, and B. Naujoks. Single- and multiobjective evolutionary
optimization assisted by gaussian random field metamodels. IEEE Transactions on Evolutionary
Computation, 10(4):421–439, 2006. doi: 10.1109/TEVC.2005.859463.
[14] Dimple Vijay Kochar, Hanrui Wang, Anantha P. Chandrakasan, and Xin Zhang. Ledro:
Llm-enhanced design space reduction and optimization for analog circuits. In 2025 IEEE
International Conference on LLM-Aided Design (ICLAD), pages 141–148, 2025. doi:
10.1109/ICLAD65226.2025.00011.
[15] Chengjie Liu, Weiyu Chen, Anlan Peng, Yuan Du, Li Du, and Jun Yang. Ampagent: An
llm-based multi-agent system for multi-stage amplifier schematic design from literature for
process and performance porting, 2024. URL https://arxiv.org/abs/2409.14739.
[16] Irene Terpstra. Empowering Analog Integrated Circuit Design through Large Language Models
and Reinforcement Learning. PhD thesis, Massachusetts Institute of Technology, 2024.
[17] Erle Zhu, Yadi Liu, Zhe Zhang, Xujun Li, JinZhou, Xinjie Yu, Minlie Huang, and Hongning
Wang. MAPS: Advancing multi-modal reasoning in expert-level physical science. In The
Thirteenth International Conference on Learning Representations, 2025. URL https://
openreview.net/forum?id=GR0y0F3Ipd.
[18] Zixuan Zhao, Adhi Daiv, and Diego Porras. Curvlm: Circuit understanding via group relative
policy optimization (grpo) on vision language models. CS231n: Deep Learning for Computer
Vision, Stanford, Final Project Reports, Highlight, Spring, 2025.
[19] Yihong Tang, Ao Qu, Zhaokai Wang, Dingyi Zhuang, Zhaofeng Wu, Wei Ma, Shenhao Wang,
Yunhan Zheng, Zhan Zhao, and Jinhua Zhao. Sparkle: Mastering basic spatial capabilities in vi-
sion language models elicits generalization to spatial reasoning. In Christos Christodoulopoulos,
Tanmoy Chakraborty, Carolyn Rose, and Violet Peng, editors, Findings of the Association for
Computational Linguistics: EMNLP 2025, pages 4083–4103, Suzhou, China, November 2025.
Association for Computational Linguistics. ISBN 979-8-89176-335-7. doi: 10.18653/v1/2025.
findings-emnlp.217. URL https://aclanthology.org/2025.findings-emnlp.217/.
[20] Xingqi Wang, Yiming Cui, Xin Yao, Shijin Wang, Guoping Hu, and Xiaoyu Qin. Charthal:
A fine-grained framework evaluating hallucination of large vision language models in chart
understanding, 2025. URL https://arxiv.org/abs/2509.17481.
[21] China postgraduate ic innovation competition · eda elite challenge contest, 2024. URL http:
//edachallenge.cn.
[22] Rejin Varghese and Sambath M. Yolov8: A novel object detection algorithm with en-
hanced performance and robustness. In 2024 International Conference on Advances in
Data Engineering and Intelligent Computing Systems (ADICS), pages 1–6, 2024. doi:
10.1109/ADICS58448.2024.10533619.
11

## Page 12

[23] Martin Ester, Hans-Peter Kriegel, Jörg Sander, and Xiaowei Xu. A density-based algorithm
for discovering clusters in large spatial databases with noise. In Proceedings of the Second
International Conference on Knowledge Discovery and Data Mining, KDD’96, page 226–231.
AAAI Press, 1996.
[24] David Eriksson, Michael Pearce, Jacob Gardner, Ryan D Turner, and Matthias
Poloczek. Scalable global optimization via local bayesian optimization. In H. Wal-
lach, H. Larochelle, A. Beygelzimer, F. d'Alché-Buc, E. Fox, and R. Garnett, edi-
tors, Advances in Neural Information Processing Systems, volume 32. Curran Associates,
Inc., 2019. URL https://proceedings.neurips.cc/paper_files/paper/2019/file/
6c990b7aca7bc7058f5e98ea909e924b-Paper.pdf.
[25] Google DeepMind. Gemini 3 flash model card. Technical report, Google, Decem-
ber 2025. URL https://storage.googleapis.com/deepmind-media/Model-Cards/
Gemini-3-Flash-Model-Card.pdf.
[26] OpenAI. Introducing gpt-5.2. https://openai.com/index/introducing-gpt-5-2/, De-
cember 2025. Accessed: 2026-03-16.
[27] Gemma Team. Gemma 4 technical report, 2026. URL https://arxiv.org/abs/2607.
02770.
[28] Shuai Bai et al. Qwen3-vl technical report, 2025. URL https://arxiv.org/abs/2511.
21631.
[29] Yu Cao. Predictive technology model for robust nanoelectronic design. Springer Science &
Business Media, 2011.
[30] R Timothy Edwards. Google/skywater and the promise of the open pdk. In Workshop on
Open-Source EDA Technology, volume 124, page 132, 2020.
[31] Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto,
Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul
Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke
Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad
Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias
Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex
Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain,
William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant
Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie
Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and
Wojciech Zaremba. Evaluating large language models trained on code, 2021. URL https:
//arxiv.org/abs/2107.03374.
[32] Jintao Li, Haochang Zhi, Ruiyu Lyu, Wangzhen Li, Zhaori Bi, Keren Zhu, Yanhan Zeng, Weiwei
Shan, Changhao Yan, Fan Yang, Yun Li, and Xuan Zeng. Analoggym: An open and practical
testing suite for analog circuit synthesis (invited). In 2024 ACM/IEEE International Conference
On Computer Aided Design (ICCAD), pages 1–9, 2024. doi: 10.1145/3676536.3697117.
[33] Ka Nang Leung and P.K.T. Mok. Analysis of multistage amplifier-frequency compensation.
IEEE Transactions on Circuits and Systems I: Fundamental Theory and Applications, 48(9):
1041–1056, 2001. doi: 10.1109/81.948432.
[34] Mohsen Ahmadzadeh, Jan Lappas, Norbert Wehn, and Georges Gielen. Anacraft: Duel-play
probabilistic-model-based reinforcement learning for sample-efficient pvt-robust analog circuit
sizing optimization. IEEE Transactions on Computer-Aided Design of Integrated Circuits and
Systems, 45(2):901–914, 2026. doi: 10.1109/TCAD.2025.3582175.
[35] J. Ramos and M. Steyaert. Three stage amplifier with positive feedback compensation scheme.
In Proceedings of the IEEE 2002 Custom Integrated Circuits Conference (Cat. No.02CH37285),
pages 333–336, 2002. doi: 10.1109/CICC.2002.1012833.
12

## Page 13

[36] Xuanming Cui, Alejandro Aparcedo, Young Kyun Jang, and Ser-Nam Lim. On the robustness
of large multimodal models against image adversarial attacks. In Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition (CVPR), pages 24625–24634, June
2024.
A Appendix: Experimental and Implementation Details
A.1 Experimental Environment
Simulations use Ngspice-46 with DC operating-point, AC, and transient analyses. Experiments run
on an AMD EPYC 7513 server with 64 cores, 256 GB RAM, and four NVIDIA RTX 4090 GPUs.
We serve the Gemma-4-31B-it backbone locally with vLLM, using tensor parallelism 4, a maximum
model length of 126,000, the Gemma 4 tool-call and reasoning parsers, and GPU memory utilization
of 0.96.
A.2 Baseline and Execution Protocol
VLM-CAD and EESizer operate on the same netlists, technology models, and specifications. EESizer
runs up to 25 optimization iterations per run; the VLM-CAD DC-OP and inference-only phases run
up to 10 and 40 iterations, respectively.
A.3 VLM-CAD ExTuRBO Configuration
ExTuRBO runs up to three parallel workers, each seeded from a distinct Phase-D candidate, with a
20-point initial cloud, a default local span of 40% around the seed, and a 400-simulation budget per
worker. For length parameters, the local bounds span 0.8 to 3.0 times the seed value. The Universal
Cost uses penalty weights 1.0 for gain, 0.1 for phase margin, 0.2 for UGBW, 0.5 for THD, and 10.0
for offset, plus a sanity penalty of 100.0. Penalty terms act on raw engineering units, so the weights
express relative emphasis rather than normalized margins.
A.4 Statistical Detail
For structural evaluation, functional accuracy is computed as F = 100Ncorrect/122, where a predic-
tion is correct when its circuit-type label matches the reference annotation. The connectivity score
is computed per schematic as Sk = 1/ log10(10 + K), where K is the graph edit cost between the
predicted and reference connectivity graphs; the table reports the mean and population standard devi-
ation of Sk across the 122 schematics. A missing report entry is assigned F = 0 and K = |V | + |E|
for the reference graph. An empty or unparsable predicted netlist receives the same K penalty, while
its F remains determined by circuit-type label matching when available. The 2,268 schematics used
to train Image2Net and the 122 noisy evaluation schematics contain disjoint circuit families.
Confidence intervals for pooled Pass@1 and Pass@3 are obtained by a task-level cluster bootstrap:
we resample the twelve tasks with replacement 100,000 times with a fixed random seed, recompute
the pooled mean per replicate, and take percentile intervals. Resampling at the task level preserves
within-task dependence among the five runs.
13

