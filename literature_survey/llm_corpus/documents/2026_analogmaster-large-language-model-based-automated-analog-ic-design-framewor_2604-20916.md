---
title: "AnalogMaster: Large Language Model-based Automated Analog IC Design Framework from Image to Layout"
authors: "Qin, Xian Rong; Zhang, Yong; Hu, Ying; Su, Tao; Jia, Bo-Wen; Xu, Ning"
date: "2026/04/22"
category: "end_to_end"
source_url: "https://arxiv.org/abs/2604.20916"
pdf: "papers/2026_analogmaster-large-language-model-based-automated-analog-ic-design-framewor_2604-20916.pdf"
sha256: "7a67ef5b5469c304d4730ae97230b4a08e993605e829012166df3ff532927608"
pages_extracted: 23
---

# AnalogMaster: Large Language Model-based Automated Analog IC Design Framework from Image to Layout

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

Graphical Abstract
AnalogMaster: Large Language Model-based Automated Analog IC Design Framework from Image to Layout
Xian Rong Qin, Yong Zhang, Ying Hu, Tao Su, Bo-Wen Jia, Ning Xu
arXiv:2604.20916v1
[cs.AR]
22
Apr
2026

## Page 2

Highlights
AnalogMaster: Large Language Model-based Automated Analog IC Design Framework from Image to Layout
Xian Rong Qin, Yong Zhang, Ying Hu, Tao Su, Bo-Wen Jia, Ning Xu
• Research highlight 1: Proposed AnalogMaster, the first LLM-based framework for end-to-end analog IC design
automation.
• Research highlight 2: Introduced a joint reasoning mechanism to enhance netlist extraction from complex circuit
images.
• Resarch highlight 3: Designed a parameter search agent to auto-compress device parameter space for down-
stream physical design.
• Rearch highlight 4: Demonstrated state-of-the-art performance across multiple MLLMs on a 15-circuit bench-
mark.

## Page 3

AnalogMaster: Large Language Model-based Automated Analog IC Design
Framework from Image to Layout
Xian Rong Qina,1
, Yong Zhanga,b,1
, Ying Hua
, Tao Sua
, Bo-Wen Jiaa,b,∗
, Ning Xua,b,∗
aSchool of Information Engineering, Wuhan University of Technology, , 430070, Wuhan, China
bState Key Laboratory of Integrated Circuits and Systems, Fudan University, , 200433, Shanghai, China
Abstract
Design automation has the potential to substantially improve the efficiency of analog integrated circuit (IC) design.
However, existing algorithms and tools typically focus on individual stages, such as device sizing, placement, or
routing, and still require significant manual intervention to complete the full design flow. While large language models
(LLMs) have recently demonstrated remarkable success in automating digital IC design workflows, these advances
cannot be directly transferred to analog IC design. Key challenges include strongly coupled performance metrics, the
predominance of unstructured circuit schematic images, and the fact that most prior approaches address only isolated
stages of the analog design process, limiting their ability to capture end-to-end performance impact. To address these
challenges, we propose AnalogMaster, an extensible, LLM-based framework that enables end-to-end automation of
analog IC design through a unified pipeline spanning circuit image–to–netlist generation, parameter optimization,
placement, and routing. AnalogMaster integrates a joint reasoning mechanism that leverages in-context learning and
intent reasoning to achieve accurate and robust image-to-netlist conversion. A parameter search agent integrating
self-enhanced prompt engineering and context truncation is developed for effective device sizing and downstream
physical design. Experimental evaluations on 15 representative circuits with varying levels of complexity demonstrate
strong and consistent performance across multiple models. In particular, GPT-5 achieves success rates of 92.9% and
99.9% on Pass@1 and Pass@5, respectively. These results validate the effectiveness and robustness of the proposed
framework and establish a practical paradigm for applying LLMs to full-stack analog IC design automation.
Keywords: Electronic design automatic, LLM, Analog ICs, Layout, Netlist
1. Introduction
The rapid advancement of large language models (LLMs) has enabled significant breakthroughs in natural lan-
guage processing and general-purpose code generation(Jin et al., 2025). Inspired by these successes, recent research
has begun to explore the application of LLMs to Electronic Design Automation (EDA). Benefiting from the high-level
abstraction in hardware description languages (HDLs) and the availability of large-scale, high-quality open-source
datasets, LLM-based approaches have made notable progress in digital circuit design. Representative efforts include:
(1) HDL code generation and benchmark evaluation (Liu et al., 2023b; Thakur et al., 2023; Lu et al., 2024; Thakur
et al., 2024; Liu et al., 2024c); and (2) automated bug detection, repair, and testbench/script synthesis (Qiu et al., 2024;
Bhandari et al., 2024b; Liu et al., 2023a). In contrast, automated analog integrated circuit(IC) design presents fun-
damentally different challenges. Key performance metrics, such as gain, bandwidth, power consumption, and noise,
exhibit strong nonlinear dependencies and complex cross-coupling effects. These intricate interdependencies propa-
gate throughout the entire design flow—spanning netlist generation, device sizing, placement, and routing—rendering
∗Corresponding author
Email addresses: qinxianrong@whut.edu.cn (Xian Rong Qin), yongz@whut.edu.cn (Yong Zhang), hu_ying@whut.edu.cn (Ying
Hu), sutao0606@whut.edu.cn (Tao Su), jiabowen@whut.edu.cn (Bo-Wen Jia), xuning@whut.edu.cn (Ning Xu)
1These authors have contributed equally to this work and are co-first authors.

## Page 4

each stage highly sensitive to decisions made in others. Consequently, even a single circuit topology must be metic-
ulously adapted to meet diverse application-specific specifications, requiring deep domain knowledge and holistic
co-optimization across all stages. Isolated tuning of any individual step is insufficient; moreover, as circuit com-
plexity increases, the combinatorial explosion of cross-stage constraints renders manual, stage-by-stage optimization
extremely time-consuming and labor-intensive. This reliance on human expertise not only limits scalability but also
underscores the formidable difficulty—and pressing necessity—of achieving a fully automated, end-to-end analog IC
design methodology.
Netlists in analog IC design are typically constructed interactively by engineers using commercial platforms such
as Virtuoso. Recently, several studies have explored netlist generation using learning-based approaches. AnalogCoder
(Lai et al., 2025) proposes a training-free LLM agent that translates natural language specifications into netlists via
intermediate Python code, while SpicePilot (Vungarala et al., 2024) employs domain-informed prompting to itera-
tively guide PySpice/SPICE netlist synthesis. Compared with digital IC design, analog IC design lacks large-scale,
well-structured datasets; instead, a substantial portion of analog design knowledge is embedded in schematic dia-
grams. Consequently, directly extracting netlists from circuit schematics is essential for advancing the application
of LLMs in analog IC EDA. Moreover, representing circuit structures through general-purpose programming lan-
guages incurs substantial token overhead and higher syntactic error rates compared with native SPICE formats, which
degrades generation accuracy and complicates downstream design stages. Even with accurate netlist extraction,
achieving high-performance analog ICs requires precise device sizing—an area where automation remains largely
decoupled from topology synthesis. In the domain of device sizing, (Žiga Rojec et al., 2019) proposed an evolution-
ary algorithm-based approach that simultaneously optimizes both the topology and component parameters of analog
circuits through a novel graph-based representation and hybrid parameter optimization. MACE(Lyu et al., 2018)
proposes a batch Bayesian optimization framework that parallelizes optimization by integrating multiple acquisition
functions into a multi-objective optimization setting. (Somayaji et al., 2021)introduces a prioritized reinforcement
learning (RL) framework that combines non-uniform prioritized experience replay, a two-stage critic network em-
bedding design knowledge, and trajectory-guided local exploration. AmpAgent (Liu et al., 2024a) further adopts
a multi-agent collaborative architecture that integrates literature mining, mathematical reasoning, and hybrid opti-
mization (ABC/TuRBO) to design multi-stage amplifiers. Although these methods demonstrate impressive results
in device sizing, they rely on manually specified circuit topologies or initial netlists and do not extend to placement
and routing based on optimized parameters. As a result, their impact on overall circuit performance and physical
feasibility remains difficult to assess. For analog IC placement, (Ma et al., 2010)presents a simulated annealing (SA)
based framework that incorporates sequence pairs and center-based corner block lists to handle symmetry, common-
centroid, and general placement constraints. (Xu et al., 2017) proposes a hierarchical analytical placement technique
for high-performance analog ICs, achieving significant reductions in both critical net wirelength and layout area com-
pared with conventional approaches and manual designs. Nevertheless, these methods require a human-provided
netlist as input and do not proceed to post-placement routing, leaving routing feasibility and post-layout circuit per-
formance unverified. In the routing domain, (Zhang et al., 2025b) introduces a design-rule-compliant bidirectional A*
algorithm combined with an attention-augmented RL framework for track assignment, improving routing efficiency.
(Zhang et al., 2025c) proposes a pin-access-planning-driven matching routing methodology for analog ICs, enhancing
both routing symmetry and layout performance. (Xu et al., 2025) presents PARoute2, a performance-driven analog
routing paradigm that leverages machine learning to address the limitations of traditional methods. By integrating a
3DGNN-based performance predictor and a 3DGRU-based routing guidance generator, PARoute2 generates nonuni-
form routing guidance to optimize post-layout performance metrics. However, these techniques operate under the
assumption of a given netlist and pre-computed placement, and thus do not constitute an end-to-end solution from
specification to physical implementation.
In summary, existing studies predominantly decompose analog IC design into isolated subtasks, such as netlist
generation, sizing, placement, and routing, then optimize each component independently. Although these approaches
make meaningful technical progress, they neglect the strong interdependencies across the analog design flow, thereby
obscuring how local design decisions propagate to global performance and physical feasibility. As summarized in
Table1, few previous works have achieved end-to-end automation spanning circuit image interpretation, parameterized
netlist synthesis, performance optimization, and placement and routing. This fragmentation creates a critical gap that
limits both the practical applicability of current methods and the systematic evaluation of LLM capabilities in analog
EDA. To address this gap and fully exploit the potential of LLMs across the entire analog design lifecycle, we propose
2

## Page 5

AnalogMaster, an extensible LLM-based framework for full-stack analog IC automation.
Table 1: Comparison of Recent LLM-based Methods for Analog IC Design
Method
Image-Netlist
Generationa Sizing Placement Routing
Fully
Automatedb Benchmarksc
AnalogCoder1
× × × × ✓ ✓
SPICEPilot2
× × × × × ×
LaMAGIC3
× × × × ✓ ×
LLM-USO4
× ✓ × × ✓ ×
AnalogXpert5
× × × × ✓ ✓
MasaCHAI6
✓ × × × ✓ ✓
LADAC7
× ✓ × × × ×
AmpAgent8
× ✓ × × × ×
LayoutCopilot9
× × ✓ × × ×
AnalogMaster ✓ ✓ ✓ ✓ ✓ ✓
a Convert the circuit image to the netlist. b Without human intervention. c Provided benchmarks. 1Lai et al. (2025), 2Vungarala et al. (2024),
3Chang et al. (2024), 4Somayaji and Li (2025), 5Zhang et al. (2025a), 6Bhandari et al. (2024a), 7Liu et al. (2024b), 8Liu et al. (2024a), 9Liu et al.
(2025).
Our main contributions are:
1. We propose AnalogMaster, an extensible LLM-based framework that enables end-to-end analog IC design
automation, including circuit images-to-netlist conversion, sizing, placement, and routing. This work establishes a
new paradigm for integrating LLM into the complete IC design flow.
2. We address the high error rate in directly converting circuit images to SPICE netlists by introducing a joint
reasoning mechanism, which substantially improves the model’s netlist extraction accuracy from complex circuit
schematics by synergistically integrating multi-dimensional reasoning cues derived from circuit images.
3. We develop a parameter search agent that effectively compresses the device parameter search space for visually
extracted netlists, addressing the limitation that topology-only netlists can’t be directly used for placement and routing.
Compared with traditional methods that rely on manually defined parameter ranges, the proposed approach is more
general and adaptive.
4. Experimental results demonstrate that the AnalogMaster framework consistently achieves outstanding perfor-
mance across multiple MLLMs on a benchmark comprising 15 representative circuit diagrams with varying difficulty
levels. In particular, GPT-5 achieves success rates of 92.9% on Pass@1 and 99.9% on Pass@5, validating the effec-
tiveness and robustness of the proposed framework.
2. Background
2.1. Multimodal Large Language Models & Agent
Multimodal Large Language Models (MLLMs) are pretrained on massive generic corpora of images and text,
enabling strong cross-modal generalization through aligned token representations: given an image ximg and a text
sequence xtext, they construct a joint context Z = [Etext, Eimg] for autoregressive decoding. This architecture supports
in-context learning(Min et al., 2022), allowing the model to infer task intent from few-shot demonstrations without
parameter updates. However, when applied to analog IC design—a domain with scarce, expert-dependent annotations
and highly structured circuit images—MLLMs face severe limitations. Their pretraining data lack the domain-specific
semantics of circuit topology, making direct end-to-end netlist generation from raw diagrams unreliable. Recent
studies (Bhandari et al., 2024a)(Liu et al., 2024a) confirm that even state-of-the-art MLLMs fail to consistently recover
correct netlists.
MLLMs are fundamentally stateless, probabilistic systems: each inference step processes input and generates
output as independent high-dimensional vector transformations, without intrinsic memory or sustained reasoning
capabilities. However, many analog IC design tasks—such as circuit netlist synthesis, sizing, placement, and rout-
ing—demand extended cognitive functions like long-term memory, autonomous planning, environmental awareness,
3

## Page 6

and dynamic decision-making. Successful execution requires iterative strategy formulation, anomaly detection, result
validation, and refinement until the final objective is achieved. To bridge this gap, recent works(Liu et al., 2024a)(Wu
et al., 2024)(Somayaji and Li, 2025) have augmented agents by integrating LLMs with external memory modules, tool
invocation interfaces, state-tracking mechanisms, and specialized prompt engineering strategies. These enhancements
significantly extend the practical applicability of LLMs in EDA contexts. Additionally, multi-agent collaborative
frameworks(Liu et al., 2024a) have been proposed to enable role specialization and distributed reasoning, thus facili-
tating the automation of more sophisticated and higher-order EDA functions.
However, both MLLMs and agents remain constrained by finite context windows: excessive input truncates critical
information, while insufficient context impairs coherence. In a complex analog circuit image, these constraints amplify
the risk of hallucination—generating syntactically valid but fictional or irrelevant netlists that do not correspond to the
input image, leading to functionally incorrect or non-existent circuits.
2.2. Automated process for analog IC design
The analog IC design process can be divided into the following five stages.
1) Circuit Design. Traditional design mandates that engineers construct circuit topologies meeting functional re-
quirements based on specific application scenarios by utilizing fundamental circuit theories and specified components.
Examples include amplifier circuits, filter circuits, comparator circuits, etc.
2) Netlist Generation. In conventional circuit design tools, engineers manually input parameters and connectiv-
ity relationships of circuit elements to generate a textual file containing all circuit information through commercial
software; this is known as a netlist. The netlist describes the type, pin connections, parameter settings, and more for
each component in the circuit, serving as the foundation for subsequent design processes. However, forming a netlist
directly from circuit images poses a perception-and-reasoning challenge for automated algorithms, necessitating ac-
curate localization, classification, and topological inference. Formally, this task can be expressed as:
Pnetlist = P








N
\
i=1
{ĉi = ci} ∩
\
(i,j)∈E
{êij = eij}







 , (1)
where N denotes the total number of components in the image; ci and ĉi represent the ground-truth and predicted class
labels of the i-th component, respectively; E denotes the set of true inter-component electrical connections; eij ∈ {0, 1}
indicates the presence (1) or absence (0) of a physical connection between components i and j, with êij being its
predicted counterpart; and Pnetlist quantifies the probability of perfectly recovering the complete netlist—i.e., the
joint likelihood of correctly identifying all component types and accurately reconstructing all pairwise connectivity
relationships simultaneously. Recent studies have also explored netlist generation. For instance, Pan et al. (2026)
proposed a method to infer missing component connectivity from incomplete netlists, thereby partially alleviating
reliance on conventional circuit design tools.
3) Device Sizing. In analog IC design, optimizing device dimensions is crucial for achieving the desired circuit
performance. For a given topology, determining optimal parameters for active and passive components (such as the
width-to-length ratio (W/L) of MOSFETs, resistances, capacitances, etc.) is required to meet specifications like gain,
bandwidth, power consumption, area, common-mode rejection ratio (CMRR), among others. These objectives are
often interconnected or conflicting, and as the number of devices increases, the search space for parameters expands
exponentially, making manual tuning both labor-intensive and less likely to achieve optimality.
Let x ∈ X ⊆ Rd
denote the vector of design parameters for circuit C, and let p = (p1, . . . , pm) ∈ P represent
the performance metrics obtained through simulation, where pi(x) is a function of x. The design goals are typically
expressed through a figure-of-merit (FOM), which aggregates individual metrics into a composite function:
FOM(x) = f p1(x), . . . , pm(x)

(2)
The optimization task aims to find the global maximum point
x∗
= arg max
x∈X
FOM(x) (3)
or a feasible solution set satisfying all constraints. Given the strong nonlinearity, complex trade-offs, and coupling
effects in analog circuits, this sizing optimization problem is inherently high-dimensional, non-convex, and computa-
tionally expensive.
4

## Page 7

To address this issue, Bayesian Optimization (BO) is widely employed in automated analog IC sizing due to its
sample efficiency and ability to balance exploration and exploitation. In our study, we first utilize a parameter search
agent3.4 to significantly reduce the original parameter space by eliminating infeasible or redundant regions based
on domain knowledge and preliminary analysis. Then, we apply BO on the reduced space to identify high-quality
device parameters, ensuring stable circuit operation while guaranteeing convergence reliability and minimizing costly
simulations.
4) Placement. In analog IC design, device placement is vital for performance, matching accuracy, and robustness
against process/environmental variations. Unlike digital circuits, analog designs are highly sensitive to layout effects
such as device positioning, parasitics, thermal gradients, and wiring symmetry. Therefore, placement strategies must
comply with geometric design rules and maintain functional integrity under manufacturing and operational fluctua-
tions. After completing device sizing via BO, we adopt the SA algorithm for analog placement. SA balances global
exploration and local refinement by adjusting the acceptance of suboptimal moves through a temperature image, ef-
fectively generating high-quality, manufacturable layouts that meet performance requirements.
5) Routing. Analog routing is a crucial step that bridges the SPICE netlist to physical implementation, signif-
icantly influencing analog IC performance. Unlike digital routing, which prioritizes connectivity and timing, ana-
log routing must address non-ideal physical effects—including interconnect parasitics and device geometric mis-
matches—that degrade key metrics like DC gain, CMRR, bandwidth, and noise. In our framework, after performing
BO-based device sizing and SA-driven placement, we conduct detailed routing using an enhanced A* search algo-
rithm. This algorithm employs a customized heuristic cost function integrating obstacle avoidance, electrical sensi-
tivity, and symmetry constraints, embedding analog-specific design rules into pathfinding to generate high-quality,
manufacturing-compliant routes while preserving circuit electrical integrity.
3. method
3.1. Method OverView
Unlike the end-to-end systems adopted in most digital IC design, analog IC design features a low level of abstrac-
tion, a limited number of standardized modular units, and strong coupling correlations between various stages in the
design flow. Therefore, it is difficult to construct an end-to-end design system simply through a sequential scripting
process. We present AnalogMaster, an end-to-end, training-free framework that automates the complete analog IC de-
sign flow, from circuit image input to physical layout generation. As illustrated in Figure1, AnalogMaster integrates
multiple tightly coupled stages, including circuit image preprocessing, parameterized netlist extraction, device siz-
ing, placement, and routing. Specifically, circuit image preprocessing combines a customized YOLO-based detector,
EasyOCR, and electrical connectivity analysis to recover circuit components and their interconnections. The extrac-
tion of the netlist from the image is then performed through a joint reasoning mechanism. Device sizing is formulated
as a constrained optimization problem and solved using BO, while placement and routing are carried out using SA
and A*-based path search, respectively. To improve optimization efficiency, a parameter search agent first analyzes
the inferred circuit topology to compress the feasible design space, thereby enabling BO to more effectively explore
device parameters under target performance specifications. Through the seamless integration of vision, reasoning,
and optimization modules, AnalogMaster achieves accurate, scalable, and fully automated analog IC design without
model retraining.
3.2. Circuit Image Annotation
To support robust circuit image understanding, we have meticulously constructed a dataset of approximately
10,000 high-quality annotated circuit image, covering common component categories found in analog IC schematics.
Based on this dataset, we trained a YOLOv9 model specifically for circuit component detection. As shown in Figure2,
the trained YOLOv9 model localizes all circuit components in the input image and outputs bounding boxes defining
device positions. In parallel, EasyOCR is applied to identify and mask textual annotations, preventing interference
with electrical connectivity inference. Detected component regions are also masked, yielding a wire-only image that
retains only conductive paths. Morphological operations and connected-component analysis are then applied to ex-
tract electrically connected regions. Small components below a predefined area threshold, which typically correspond
5

## Page 8

Component Localization
Circuit Connectivity Detection
Circuit Image
Joint Reasoning
UE SPICE Netlist2
M1 is an NMOS transistor, and its
gate should be connected to...
M2 is an PMOS transistor, and its
drain should be connected to..
Complexed Context
UE SPICE Netlist1
Context Complex
Intent Inference
Netlist 1
Netlist 2
Netlist 3 Final Netlist
M1A (Vout1 Vin1 Node2 GND) nmos
M2A (Vout2 Vin2 Node2 GND) nmos
M3B (Vout1 Node2 VDD VDD) pmos
M4B (Vout2 Node2 VDD VDD) pmos
M5 (Node2 Vb GND GND) nmos
{"M1A":
{"range_w":
{"low": 2.0,…}
Parameter Search Agent
Tools
Task1: Circuit Structure Analysis
Symmetric constraint extraction
Value Estimation
Task2: NGSPICE
Simulation Verification
Observation of MOSFET
Conduction Status
Bias Voltage Estimation
Bias Voltage Scaning
Task3: …..
Simulator
Fully Automated
Routing
Fully Automated
Placement
Bayesian-based Sizing
Optimization
UE SPICE Netlist3
Circuit Images
Train
Schematic Text OCR
Tool Call
Feedback
Action 1: …
Action 2 : …
Circuit Image
Annotation
Figure 1: Workflows of AnalogMaster: The circuit image, preprocessed via YOLO-based detection and connectivity analysis, is converted into
a netlist using the proposed joint reasoning mechanism. A parameter search agent compresses the device parameter space, followed by BO for
sizing, SA for placement, and A* for routing—enabling a fully automated design flow.
to non-functional ones, are discarded. Each valid connected region is assigned a unique color for visual disambigua-
tion. To ensure consistent net representation, an overlap-aware filtering strategy based on centroid Euclidean distance
is employed to merge spurious node annotations within the same electrical region. The final output integrates three
complementary sources of information: YOLO-derived component bounding boxes, color-encoded connectivity re-
gions, and refined network identifiers. This structured, multi-dimensional representation preserves both topological
and semantic information, forming a robust foundation for subsequent netlist extraction.
(a) Circuit image (b) YOLO detection (c) Text OCR
(e) Electrical regions extraction
(f) Annotated circuit image (d) Wire-only image
Figure 2: The process flow for circuit image analysis. (a) Original circuit image. (b) YOLO-based detection of components in the circuit. (c) Text
detection using OCR to identify labeled components. (d) Wire-only image extracted from the circuit. (e) Electrical regions extraction showing
nodes and their connections. (f) Annotated circuit image with labeled nodes and components.
3.3. Joint Reasoning
To improve the accuracy of circuit image-to-netlist conversion, we introduce Joint Reasoning, a training-free
paradigm that enhances the image comprehension capability of general-purpose MLLMs. As illustrated in Figure 3,
6

## Page 9

this framework constructs three distinct parallel inputs from raw and annotated circuit images, and feeds them into
parallel MLLM branches for independent reasoning. A high-level LLM is subsequently adopted to synthesize multi-
branch outputs via intent reasoning, thereby generating a reliable and accurate final netlist. Moreover, the reasoning
process and circuit structural features are compressed into compact context information, which effectively narrows
the parameter search space for Bayesian optimization in the downstream Parameter Search Agent.
UE SPICE Netlist2
UE SPICE Netlist1
Netlist 1
Netlist 2
Netlist 3 Final Netlist
M1A (Vout1 Vin1 Node2 GND) nmos
M2A (Vout2 Vin2 Node2 GND) nmos
M3B (Vout1 Node2 VDD VDD) pmos
M4B (Vout2 Node2 VDD VDD) pmos
M5 (Node2 Vb GND GND) nmos
UE SPICE Netlist3
Raw image
Annotated circuit image
COT Prompt
Reference
Image
Combine
MICL
prompt
Synthesis
Context Complex
Intent reasoning
Final Netlist
Summary
Figure 3: Overview of the Joint Reasoning workflow. Three heterogeneous inputs are processed via parallel MLLM reasoning branches, followed
by multi-branch result synthesis to generate accurate netlists. Compact contextual information is further extracted to facilitate downstream Bayesian
optimization.
3.3.1. Chain-of-Thought Prompting
The conversion of a circuit images into a SPICE netlist requires three essential reasoning steps: (1) identifying all
circuit components and correctly classifying their types; (2) determining the electrical connectivity of each terminal
(port) associated with every component; and (3) merging equipotential nodes to form a structurally consistent and
functionally complete netlist. To guide the MLLM through this structured reasoning process, we adopt a Chain-of-
Thought (CoT) prompting strategy. As illustrated in Figure 4, the prompt first directs the model to analyze each
component independently, focusing on its semantic category and the local connectivity of its ports. After completing
these per-component analyses, the model is then directed to integrate the localized observations into a global circuit
representation by resolving node equivalences and consolidating connections into a unified netlist.
This stepwise decomposition not only mirrors the intrinsic logic of netlist generation but also provides flexibility
in ambiguous or uncertain inference scenarios. By decoupling perception from topological reasoning, the CoT-based
Joint Reasoning mechanism improves both accuracy and robustness of the final output, enabling reliable recovery of
circuit semantics even in the presence of visual noise, partial occlusion, or incomplete schematic information.
3.3.2. Multimodal In-Context Learning with Circuit Image-Driven System Prompt
While the CoT prompting effectively enhances the MLLM’s logical reasoning capability in recognizing circuit
diagrams and converting them into netlists, it fails to directly provide specific examples for the model to learn from.
Different from pure text-based system prompts that lack multimodal guidance, we introduce the circuit image–driven
multimodal in-context learning (MICL) strategy to make up for this deficiency, further improving the accuracy and
generalization of netlist generation. As illustrated in Figure 5, we first select a representative reference circuit image
and pair it with a carefully designed CoT prompt that explicitly captures the complete reasoning trajectory, spanning
component recognition, connectivity inference, and netlist synthesis. This multimodal input is then provided to the
7

## Page 10

Task Description
You are a professional and experienced analog IC engineer. Based on the provided circuit diagram image, you are required to complete the following task: Identify
all components in the circuit diagram and generate a SPICE netlist based on their interconnections.
Example
Here is an example circuit diagram:
User Query
This is a circuit schematic. Please identify all components and generate a SPICE netlist based on their connections.
Response
Step 1: Component Identification From the schematic, I observe the following components:
NMOS transistors: M1, M2, M3
Resistors: RD1, RP, RD2
Power supply: VDD
Ground: GND
Step 2: Terminal Connectivity Analysis
1. MOSFETs:
M1 (NMOS) Drain → Vout1; Gate → Vin1; Source → connected to source of M2
M2 (NMOS) Drain → Vout2; Gate → Vin2; Source → connected to source of M1
Note: This is a differential pair, so Vout1 and Vout2 are distinct output nodes.
M3 (NMOS) Drain → connected to the common source node of M1/M2;Gate → Vb;Source → GND
2. Resistors: RD1: connects VDD to Vout1; RD2: connects VDD to Vout2; RP: connects Vout1 to the common source node of M1/M2
Step 3: SPICE Netlist Construction
Based on the above, the netlist is:
.subckt differential_amplifier VDD GND Vin1 Vin2 Vb Vout1 Vout2
M1A Vout1 Vin1 Node4 GND nmos
M2A Vout2 Vin2 Node4 GND nmos
M3 Node4 Vb GND GND nmos
RD1 VDD Vout1 resistor
RD2 VDD Vout2 resistor
RP Vout1 Node4 resistor
.ends
Figure 4: A prompt guiding the MLLM to follow the CoT approach for sequentially analyzing a circuit image and extracting a netlist. The diagram
illustrates the step-by-step process the model follows to identify components, analyze terminal connectivity, and generate the netlist from the circuit
image.
Which of the following drugs is antidepressant
---Donepezil,Citalopram,or Aspirin?
Questio
n
D C
A
Prompti
ng
Single
Label
Completely
Wrong
Previous
?
Instinctive Ambiguity Aversion to avoid risk Hum
an
High
risk
Donepezil
High
risk
It must be Donepezil.
It must be Citalopram.
Low
risk
Emm...actually I’m not sure.I think it might be Donepezil,or Citalopram.
D C
A
Include
Correct
Ou
rs
Prompti
ng
Single
Label Ambiguity
Aversion
Donepezil Citalopram
?
As a professional and experienced analog IC engineer, you are required to
complete the following tasks based on the provided circuit image. First, identify
all components in the image, including their types and terminal connections.
Then…..
OK ! I have read and understood the reference images and the corresponding thinking process. Next, please give the
circuit images that need to be identified, and I will analyze them step by step to get the correct network table.
According to the given circuit image, identify the components of the circuit and
describe the connection relationship as a netlist…… You need to track the wires
actually drawn in the diagram to determine the specific connection points of each
component.
?
This is a differential amplification circuit with a common source common gate auxiliary branch.
1. Circuit structure composition
- Differential pair tube: M1 and M2 form an NMOS differential pair, which is the core amplification unit of the circuit. The
input signals V_in1 and V_in2 are added to the gates of the two tubes respectively.
- Active load and leakage resistor: Two R_D are leakage load resistors, which convert the leakage current change of M1 and
M2 into voltage changes, and finally output .....
Figure 5: Illustration of the multimodal context learning process. First, reference images and CoT prompts are input into the MLLM to establish a
recognition logic based on the circuit image and the corresponding analysis process. Subsequently, the circuit image to be analyzed is provided to
the model for netlist extraction.
8

## Page 11

MLLM, and the entire model response, including intermediate reasoning steps and the final model output, is preserved
verbatim. The recorded interaction serves as a contextual anchor and is prepended to subsequent inference queries.
By conditioning new inputs on this exemplar-driven context, the model is guided to reproduce a consistent, struc-
tured, and semantically grounded reasoning process across diverse circuit images. In effect, MICL emulates the role
of a multimodal system prompt, enabling reliable few-shot adaptation without requiring architectural modification,
additional supervision, or model fine-tuning.
3.3.3. Parallel Reasoning
Due to the inherent stochasticity of MLLM decoding, repeated inferences using identical CoT prompts and MICL
contexts may yield slight output variations. Such inconsistencies can result in partial netlist extraction errors, which
may propagate to downstream design stages and compromise the overall automation flow. To enhance robustness and
mitigate this instability, we introduce a parallel reasoning mechanism that exploits multiple complementary visual
representations of the input circuit. As shown in figure1, building upon the circuit image annotation pipeline described
in 3.2, the reasoning process is decomposed into three parallel branches: (1) the original raw circuit image; (2) a
structurally enhanced circuit image derived from electrical connectivity analysis, in which components and wiring are
disentangled and annotated; and (3) a dual-images input that jointly presents both representations, enabling cross-view
comparison and collaborative reasoning. These branches operate in a decoupled and parallel manner during inference,
without introducing additional computational passes beyond a single inference step. This multi-perspective reasoning
strategy substantially improves accuracy, consistency, and fault tolerance, effectively suppressing stochastic decoding
artifacts and enhancing the reliability of netlist extraction.
3.3.4. Intent reasoning
Given the high complexity of converting analog IC circuit diagrams to SPICE netlists, even with the integration
of CoT and MICL mechanisms, inherent limitations persist. As LLMs are essentially probabilistic models and the
task itself involves intricate circuit topology analysis and electrical connectivity verification, partial errors may still
occur during the diagram-to-netlist conversion process. Notably, minor errors in this process can lead to catastrophic
failures in the entire circuit system, which is unacceptable in an end-to-end integrated system.
```spice
subckt differential_amplifier VDD GND Vin1 Vin2
Vb Vout1 Vout2
M1A (Vout1 Vin1 Node_A GND) nmos
M2A (Vout2 Vin2 Node_A GND) nmos
M3B (Vout1 Node_B VDD VDD) pmos
M4B (Vout2 Node_B VDD VDD) pmos
M5 (Node_A Vb GND GND) nmos
R1 (Vout1 Node_B) resistor
R2 (Vout2 Node_B) resistor
ends
```
```spice
subckt differential_amplifier VDD GND Vin1
Vin2 Vb Vout1 Vout2
M1A (Vout1 Vin1 Node_A GND) nmos
M2A (Vout2 Vin2 Node_A GND) pmos
M3B (Vout1 Node_B VDD VDD) pmos
M4B (Vout2 Node_B VDD VDD) nmos
M5 (Node_A Vb Node_B GND) nmos
R1 (Vout1 Node_B) resistor
R2 (Vout2 Node_B) resistor
ends
```
```spice
subckt differential_amplifier VDD GND Vin1
Vin2 Vb Vout1 Vout2
M1A (Vout1 Vin1 Node_A GND) nmos
M2A (Vout2 Vin2 Node_A GND) nmos
M3B (VDD Node_B VDD VDD) pmos
M4B (Vout2 Node_B VDD VDD) nmos
M5 (Node_A Vb GND GND) nmos
R1 (Vout1 Node_B) resistor
R2 (Vout2 Node_B) resistor
ends
```
```spice
subckt differential_amplifier VDD GND Vin1
Vin2 Vb Vout1 Vout2
M1A (Vout1 Vin1 Node_A GND) nmos
M2A (Vout2 Vin2 Node_A GND) nmos
M3B (VDD Node_B VDD VDD) pmos
M4B (Vout2 Node_B VDD VDD) nmos
M5 (Node_A Node_A GND GND) nmos
R1 (Vout1 Node_B) resistor
R2 (Vout2 Node_B) resistor
ends
```
UE SPICE Netlist2
UE SPICE Netlist3
UE SPICE Netlist1
Spice Netlist Synthesis
Figure 6: Intent reasoning process for netlist synthesis, which demonstrates how intent reasoning extracts the correct components from three
incorrect netlist results and combines them to form a fully accurate netlist.
Therefore, as illustrated in Figure 6, we introduce a text-generation LLM as a high-level inference engine that
synchronously consumes the outputs of the three aforementioned parallel multimodal reasoning branches from the
9

## Page 12

Parallel Reasoning mechanism. It should be emphasized that these three reasoning branches are completely indepen-
dent, even though they adopt the same model in the Parallel Reasoning. This independence stems from the fact that
the model receives entirely different inputs in each branch, both in terms of input quantity and content. Specifically, as
elaborated in the Parallel Reasoning section, the three branches correspond to three distinct visual representations of
the input circuit. The output from each reasoning branch is treated as a prior hypothesis of the circuit structure. Owing
to their distinct perspectives, including component localization, electrical connectivity, and cross-view comparison,
the error locations and failure modes produced by these branches when processing complex circuits are inherently
diverse and partially uncorrelated.
Through a context-aware intent reasoning mechanism—essentially an error correction and fallback strategy—the
text-generation LLM performs semantic fusion and consistency correction across these heterogeneous, complemen-
tary, and partially redundant multi-dimensional information. The LLM takes the output of each of the three indepen-
dent reasoning branches as a key clue, comprehensively analyzing and cross-verifying the components and logical
connections contained in each branch. On this basis, it identifies and eliminates erroneous components and incorrect
logical connections that may exist in individual branches. By integrating the valid information from all branches and
combining the inherent functional logic of analog circuits, the LLM infers and restores the most reasonable circuit
configuration that meets the expected functional requirements. Finally, with this restored functional circuit as the
target benchmark, the LLM further extracts the correct components and reliable logical connection relationships from
each reasoning branch output, systematically integrating these correct parts to reconstruct the intended circuit topol-
ogy. This multi-step reasoning and integration process ultimately yields a structurally consistent and semantically
correct SPICE netlist that fully conforms to the design intent of the original circuit diagram.
3.3.5. Context Compression
Mainstream LLMs rely on attention mechanisms for contextual modeling(Vaswani et al., 2017), but their quadratic
computational complexity leads to performance degradation when processing long sequences. In the proposed frame-
work, reasoning traces generated by MLLMs and LLMs contain valuable prior knowledge for subsequent Parameter
Search Agents. However, their length and redundancy can significantly reduce decision-making efficiency if incor-
porated directly as contextual input. To address this challenge, as illustrated in Figure 7, we propose a lightweight
context compression strategy. After the text-generation LLM completes intent reasoning and produces the correct
SPICE netlist, semantic extraction is simultaneously applied to the reasoning records generated by the three multi-
modal branches.
Intent reasoning
analysis process
Key reasoning process
Final Netlist
Branch1
Compressed context
Compressor
CoT-Response1
MICL
CoT
Raw circuit image
Reference image1
Branch2
CoT-Response1
enhanced circuit image
Reference image2
Branch3
CoT-Response1
Raw circuit image
Reference image1
CoT-Response2
Enhanced circuit image
Reference image2
Raw circuit image
Reference image1
CoT-Response3
Enhanced circuit image
Reference image2
Branch3
Intent resoning prompt
Raw Context Window
Circuit function
and application
New Context Window
Intent reasoning result
Figure 7: Illustration of context compression using an LLM. The diagram shows how raw context is summarized into a compressed context,
enabling more efficient processing and reasoning.
This process preserves the core analytical logic and conclusions that are directly relevant to the verified circuit
structure, while discarding irrelevant details and redundant intermediate steps. The resulting compressed context
substantially reduces sequence length and alleviates the computational burden of the attention mechanism. At the
same time, it retains sufficient structural and semantic information to guide the Parameter Search Agent effectively,
enabling more efficient and targeted exploration of the device parameter space during subsequent optimization stages.
10

## Page 13

3.3.6. Advantages of Joint Reasoning
The proposed joint reasoning framework improves both the robustness and correctness of netlist extraction through
two main approaches: 1) multi-view hypothesis expansion and 2) in-context distributional shaping.
Let Y denote the space of syntactically and electrically valid netlists, and let y∗
∈ Y be the ground-truth netlist
corresponding to a given circuit image. In standard single-branch inference, an MLLM with parameters θ takes a
fixed visual input x and defines a conditional distribution pθ(y | x) over candidate outputs y ∈ Y. Due to stochastic
decoding, each forward pass yields a random sample y ∼ pθ(y | x). This single sample being correct, or equal to y∗
,
has a probability that is precisely the Pass@1 metric:
Pass@1single = Py∼pθ(·|x)(y = y∗
) (4)
Repeating this process k times yields the conventional Pass@ksingle = 1 − 1 − Pass@1single
k
. However, the per-
formance gain of repeated sampling saturates when errors across samples are correlated, which is common under
identical input conditions. In contrast, the proposed joint reasoning framework employs three distinct visual encod-
ings of the same circuit: 1) x(1)
: the original raw image, 2) x(2)
‘: a structurally enhanced circuit image derived from
electrical connectivity analysis, 3)x(3)
: a dual-view composition of both. Each induces a separate decoding distri-
bution pθ(y | x(b)
) for branch b ∈ {1, 2, 3}. Notably, the three branches are statistically independent in terms of error
modes—though they adopt the same MLLM, they receive entirely different inputs , as elaborated in the Intent Reason-
ing section, which ensures their errors are uncorrelated and lays the foundation for subsequent probability analysis.
To formalize the benefit of this diversity, we define the high-probability support of branch b as
C(b)
=

y ∈ Y pθ(y | x(b)
) > ϵ (5)
where ϵ > 0 is a small constant that excludes negligible-probability outliers. Intuitively, C(b)
contains all netlists that
branch b can plausibly generate in a typical inference. Crucially, due to input heterogeneity, we observe C(i)
⊈ C(j)
for i , j—meaning each branch covers parts of Y missed by others. The intent reasoning module then acts as a
consensus selector over the union
S3
b=1 C(b)
. Consequently, the success probability of joint reasoning satisfies
Pass@1joint = P

y∗
∈
3
[
b=1
C(b)

≥ 1 −
3
Y
b=1

1 − P(y∗
∈ C(b)
)

≥ Pass@3single (6)
The final inequality holds because errors across branches are rarely correlated. Empirically, when one branch misiden-
tifies a component or connection, other branches often recover the correct interpretation. This diversity ensures that
joint reasoning performs at least as well as, and usually better than, three independent runs of a single-branch system.
Furthermore, the CoT prompts and MICL examples embedded in each branch serve as distributional biases. For-
mally, token-level generation follows
pθ(yt | y<t, x(b)
, c(b)
) ∝ exp fθ(y<t, x(b)
, c(b)
)

(7)
where c(b)
denotes the CoT/MICL context specific to branch b, and fθ(·) is the model’s logit function. Each (x(b)
, c(b)
)
pair steers the autoregressive process toward reasoning paths consistent with its visual semantics.
Finally, the text-generation LLM used for intent reasoning approximates a structured consensus posterior:
pϕ(y | y(1)
, y(2)
, y(3)
) ∝ pstruct(y) ·
3
Y
b=1
I

Consistent(y, y(b)
)

(8)
where pstruct(y) encodes domain-specific constraints, and I[·] is an indicator of semantic compatibility implicitly
learned during pretraining. This fusion suppresses stochastic artifacts inconsistent across views, effectively imple-
menting constrained decoding without explicit search.
Although the framework could be extended by introducing additional semantically equivalent visual variants to
further enlarge the hypothesis space, practical limitations arise as the number of branches increases. Additional
branches expand the input context length for the intent-reasoning LLM, potentially degrading reasoning quality due
to finite context windows. Moreover, marginal gains diminish as branch representations become redundant, while
computational cost grows linearly. In practice, we find that three carefully designed branches (raw, structurally en-
hanced, and dual-view) strike a favorable balance between diversity, efficiency, and accuracy.
11

## Page 14

3.4. Parameter Search Agent
Circuit images typically lack detailed component parameter information. However, key parameters are critical for
ensuring correct circuit operation and meeting target performance specifications, such as resistor values, MOSFET
channel dimensions (W/L), and bias voltages. To address this challenge, we introduce an LLM-based parameter
search agent. Through Self-Enhanced Prompt Engineering and a multi-tool collaborative invocation mechanism, the
agent constructs a feasible and structured parameter search space, enabling stable circuit functionality and effective
performance optimization.
3.4.1. Self-Enhanced Prompt Engineering
We propose a self-enhancing prompt engineering strategy to address the limitations of conventional ReAct agents.
Standard ReAct agents suffer from local reasoning bias due to the lack of macro-level planning, causing inefficient
tool usage and redundant actions. They operate in a stepwise, feedback-driven trial-and-error mode analogous to
Bayesian optimization , which improves accuracy via accumulated prior knowledge but is ill-suited for LLMs. LLM
inference incurs high computational costs, and iterative feedback loops expand context windows, drastically reducing
efficiency. Moreover, given the excessively large parameter space in device sizing, direct initialization and execution
of BO often fail to converge to feasible solutions, necessitating prior parameter space compression. Unlike BO that
pursues precise solutions, LLMs excel at efficiently narrowing the parameter search space in a coarse-grained manner.
In contrast, our strategy enables the agent to first conduct high-level task planning based on circuit topology, device
characteristics, and design constraints, decomposing global optimization objectives into ordered, prioritized sub-tasks.
Instead of completely relying on real-time feedback, the agent follows a refined "Thought–Action–Observation" cycle
to progressively narrow and refine the parameter space.
As illustrated in Figure 1, this design enables the agent to perform macro-level reasoning while sequentially
completing the planned task list. By mitigating the weak long-horizon planning capability inherent in standard ReAct
agents, the proposed method supports adaptive and stable parameter space construction across a wide range of circuit
complexities, from simple amplifiers to multi-module analog systems.
3.4.2. Context Truncation
TODO-List
……
New message
Tool Message1
Tool Message2
Tool Message3 TODO-List
New message
Raw context
Context after truncation
Figure 8: Comparison before and after context truncation. This diagram demonstrates the process of removing redundant historical tool messages
and retaining only the most relevant information, such as the TODO list and the latest tool message.
During iterative tool invocation, the agent continuously accumulates feedback information, resulting in increas-
ingly long context sequences. Excessive context length can obscure critical information and degrade the model’s
ability to retain the high-level planning objectives specified in the to-do list. Rather than applying generic context
compression techniques, we introduce a context truncation mechanism tailored specifically for agent-based parameter
search. A critical observation is that the specific information returned by tools (e.g., MOSFET operating state pa-
rameters) is valid for the current parameter configuration. When the agent transitions to a new parameter candidate,
these tool outputs must be regenerated and therefore, do not need to be retained. Exploiting this property, the context
truncation mechanism selectively discards obsolete tool feedback while preserving the agent’s planning state and de-
cision logic. As shown in figure8, this significantly reduces token computational overhead by eliminating redundant
information, while ensuring that the agent maintains access to valid context. By balancing efficiency and accuracy,
context truncation provides robust support for continuous, iterative parameter search.
12

## Page 15

4. Experiment
4.1. Dataset
We construct the Circuit Element Detection (CED) dataset, a comprehensive collection meticulously aggregated
from multiple public resources. Specifically, the original dataset is composed of 3,334 images from (Circuit Circuit
Do Circuit, 2022), 1,613 images from (Un, 2025), and 734 images from (anotnio, 2024), summing up to 5,681 raw
images. We eliminated samples representing irrelevant contexts that would not be encountered in practical applica-
tions, thereby ensuring the dataset accurately reflects realistic, high-quality circuit image environments. To enrich the
data diversity and improve model generalization, we apply a set of effective data augmentation strategies, including
rotation, random brightness adjustment, and random cropping, which effectively expands the dataset to the final scale
of 9,753 images. The dataset is systematically partitioned into training, validation, and test sets with an 8:1:1 ratio to
ensure a balanced evaluation framework.
The CED dataset encompasses 12 fundamental circuit element classes, including AC Source, BJT, Battery, Capac-
itor, DC Source, Diode, Ground, Inductor, MOSFET, Resistor, Current Source, and Voltage Source, which collectively
constitute the building blocks of analog integrated circuits. As depicted in Figure 9, compared with the v3qwe(project
gtqqq, 2022) dataset adopted by MasaCHAI, our dataset facilitates more accurate detection of circuit elements, sig-
nificantly reducing bounding box offset errors. This improvement directly enhances wire continuity preservation in
subsequent connectivity inference.
(a) Data used in MasaCHAI (c) Our data
(b) Wire-only image (d) Wire-only image
Figure 9: Comparison of circuit element detection in different datasets. (a) Circuit image from the dataset used in MasaCHAI(Bhandari et al.,
2024a). (b) Wire-only image derived from the dataset in (a). (c) Circuit image from our dataset. (d) Wire-only image derived from our dataset.
4.2. Experimental Environment
All experiments were conducted on a unified computing platform to ensure consistency and reproducibility. The
hardware configuration consists of an Intel(R) Xeon(R) Silver 4310 processor with a base frequency of 2.10 GHz,
282 GB of memory, and an NVIDIA A100-SXM4-40GB Graphics Processing Unit (GPU). The process library for
all circuits is open-sourced SKY130PDK. The proposed framework was implemented using the LangGraph frame-
work. Supporting tools and libraries include PyTorch (2.5.1), ngspice (43) for circuit simulation, and OpenCV-Python
(4.11.0.86) for image processing. The training hyperparameters used for YOLOv9 are summarized in Table 2.
4.3. Benchmark
Following the evaluation paradigm established by AnalogCoder(Lai et al., 2025), we selected 15 representative
circuit images from the AnalogGenies(Gao et al., 2025) dataset as test benchmarks, as shown in Table 3. These circuits
were classified into three difficulty levels: simple, medium, and difficult. This classification is based on factors such
as the number of components, the degree of image interference, and the complexity of the interconnection topology.
For the evaluation process to be considered successful, the proposed framework must satisfy the following criteria
simultaneously: 1. Accurately identify the input circuit image and generate the correct netlist; 2. Successfully
optimize device parameters to meet the target circuit performance specifications; 3. Ensure the placement and routing
results comply with process design constraints.
13

## Page 16

Table 2: Parameter of traing YOLO
Parameter Value
Epochs 400
Training Batch Size 32
Testing Batch Size 32
Image Size 640 × 640
Number of Worker Threads 8
Optimizer SGD
IflrAuto True
Initial Learning Rate 0.001
Patience 100
Random Seed for Reproducibility 0
Weight Decay 0.0005
Momentum 0.937
Table 3: Benchmark Description. All circuit task descriptions with varying difficulty levels are listed herein. The difficulty is categorized into
three levels:Easy, Medium, and Difficult.
Case Circuit Description
1 NMOS-input cascode amplifier
2 CMOS differential amplifier with common-mode rejection
3 CMOS differential pair with active load
4 NMOS cross-coupled cascode amplifier
5 Differential pair with source degeneration
6 CMOS differential amplifier with source degeneration
7 CMOS differential amplifier with current-source load
8 Differential pair with active load
9 CMOS differential amplifier with resistive feedback
10 Feedback-enhanced NMOS cascode amplifier
11 Two-stage cascaded differential amplifier
12 Differential pair with active load and cascode configuration
13 Multistage CMOS operational amplifier
14 Dynamic Latch Comparator
15 Differential pair with active load and cascode structure
14

## Page 17

4.4. Evaluation Metric
We adopt Pass@k(Chen, 2021), which provides an unbiased approximation of the Pass@k defined in our analy-
sis3.3.6, as the primary evaluation criterion. Pass@k measures the probability that at least one valid result satisfying
the task requirements is produced within a maximum of k independent generation attempts. Compared with single-
generation accuracy, Pass@k captures the potential performance ceiling of the model in scenarios involving random-
ness or uncertainty (e.g., non-deterministic decoding in LLMs). By allowing multiple samples, Pass@k provides a
more comprehensive evaluation of model performance. The mathematical definition is as follows:
Pass@k = 1 −
n−c
k

n
k
 (9)
where n represents the total number of attempts, c represents the number of successful generations, and
n
k

is
the binomial coefficient. In all experiments, we set n = 15. We conducted a systematic evaluation of four MLLMs:
Qwen-VL-Max, GLM-4.5V, GPT-4o-mini, and GPT-5.
4.5. Bayesian Optimization Experimental Setup
To achieve automatic parameter tuning of the circuit under test, a Bayesian optimization framework is employed
as the core experimental precondition for circuit performance optimization. The Tree-structured Parzen Estimator
(TPE) is adopted as the surrogate model, and the Expected Improvement (EI) is utilized as the acquisition function
to balance the exploration of untested parameter regions and the exploitation of high-performance regions in the
parameter space. The parameter search space is automatically determined by the parameter search agent after a
limited number of iterations. A total of 100 SPICE simulation iterations is executed to balance optimization efficiency
and computational cost. The optimization initialization adopts random sampling, and a median pruning strategy
is integrated to early terminate unpromising trials that perform worse than the median of historical results, with
convergence defined by completing a fixed number of optimization trials. The optimization direction is dynamically
specified according to the target circuit performance metric, and the entire simulation process is conducted under a
fixed temperature of 25°C using the tt (typical-nmos/typical-pmos) process corner of the Sky130 PDK.
4.6. Experimental Results
Table 4 summarizes the task completion performance of each MLLM within AnalogMaster under identical ex-
perimental configurations. The results indicate that the overall performance of the proposed framework is strongly
influenced by the capabilities of the underlying MLLM. A complete analog IC design flow requires the model to (i)
accurately identify circuit structure and generate syntactically compliant netlists, (ii) perform higher-level analytical
reasoning (e.g., recognizing device symmetry and inferring circuit functionality), and (iii) adhere strictly to instruction
constraints (e.g., producing SPICE netlists in a prescribed format). Failure at any of these stages, such as generating
an incorrect or noncompliant netlist, can propagate to downstream modules, including intent reasoning and the pa-
rameter search agent. In such cases, missing or inconsistent information prevents the construction of a feasible device
parameter range. The key results of the AnalogMaster framework process are shown in Figure 10. Among the four
evaluated models, GPT-5 achieved the best overall performance. Moreover, the remaining three models maintain a
relatively high Pass@5 success rate at k = 5 samplings, which indicates the robustness of our framework in ensuring
consistent performance even when applied to different models.
However, we observed that the GPT-4o-mini, Qwen-VL-Max, and GLM-4.5V models exhibit unsatisfactory per-
formance in certain cases. Our research reveals that this phenomenon stems from two main reasons: on the one hand,
the models themselves have a certain degree of bias towards a small number of cases. Such bias causes the models
to violate the objective facts of circuit images in all reasoning branches, leading to confusion between NMOS and
PMOS, or arbitrary speculation about port connection relationships, which in turn results in inconsistency between
the netlists and the circuit images. On the other hand, although the parameter search agent has significantly com-
pressed the parameter search space, the BO algorithm, as a sequential optimization algorithm based on a probabilistic
model, fails to meet the preset performance indicators within the effective number of iterations specified in this study.
Notably, this phenomenon is significantly alleviated in GPT-5, indicating that the framework proposed in this study
has certain requirements for the performance of the MLLM itself.
15

## Page 18

Table 4: Main Results. We evaluated the task completion performance of the AnalogMaster framework using four models, namely Qwen-VL-Max,
GLM-4.5V, GPT-4o-mini, and GPT-5, respectively.
Case
GPT-4o-mini Qwen-VL-Max GLM-4.5V GPT-5
Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5
1 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
2 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
3 86.7 100.0 46.7 98.1 100.0 100.0 100.0 100.0
4 93.3 100.0 93.3 100.0 100.0 100.0 100.0 100.0
5 99.3 100.0 100.0 100.0 100.0 100.0 100.0 100.0
6 86.7 100.0 93.3 100.0 100.0 100.0 100.0 100.0
7 6.7 33.3 20.0 73.6 0.0 0.0 100.0 100.0
8 40.0 95.8 13.3 57.1 33.3 91.6 73.3 100.0
9 26.7 84.6 60.0 99.8 46.7 98.1 100.0 100.0
10 46.7 98.1 33.3 91.6 53.3 99.3 100.0 100.0
11 20.0 73.6 93.3 100.0 53.3 99.3 80.0 100.0
12 40.0 95.8 33.3 91.6 33.3 91.6 60.0 99.8
13 0.0 0.0 0.0 0.0 6.7 33.3 86.7 100.0
14 0.0 0.0 6.7 33.3 0.0 0.0 80.0 100.0
15 0.0 0.0 6.7 33.3 0.0 0.0 100.0 100.0
Avg 49.7 72.1 53.3 78.6 55.1 74.2 92.0 99.9
Solve 12 12 14 14 12 12 15 15
M1A NtopL Vin1 Ntail GND nmos
M2A NtopR Vin2 Ntail GND nmos
M5B Vout1 Vb2 NtopL VDD pmos
M6B Vout2 Vb2 NtopR VDD pmos
M3C Vout1 Vb1 NbotL GND nmos
M4C Vout2 Vb1 NbotR GND nmos
M7D NtopL Vb3 VDD VDD pmos
M8D NtopR Vb3 VDD VDD pmos
M9E NbotL Vb4 GND GND nmos
M10E NbotR Vb4 GND GND nmos
M11 Ntail Vb5 GND GND nmos
"circuit_type": "differential_amplifier",
"device_params": {
"M1A": {
"range_w": {"low": 3.0, "high": 10.0, "step":
0.5}, "range_l": {"low": 0.3, "high": 1.0,
"step": 0.1}
},
"M3C": {
"range_w": {"low": 2.0, "high": 8.0, "step":
0.5}, "range_l": {"low": 0.3, "high": 1.0,
"step": 0.1}
}
…
(a) Circuit image (b) Annotated circuit image
Joint reasoning
(d) Parameter space
(c) Netlist
Ini
(e) Initial placement
(f) Final placement
(g) Final layout
SA
A* BO
Figure 10: Full execution example of the proposed AnalogMaster framework. (a) Original circuit image. (b) Annotated circuit image with identified
components and nodes. (c) Generated netlist based on joint reasoning. (d) Parameter space configuration based on the agent. (e) Initial placement
of components. (f) Final placement of components after SA. (g) Final layout after routing using A*.
16

## Page 19

4.7. Comparative Experiment
To further evaluate the effectiveness of the proposed method, we used Qwen-VL-Max and GLM-4.6V as core
models to replicate MasaCHAI(Bhandari et al., 2024a) on the same set of 15 circuit images and calculated its Pass@k
metric. While MasaCHAI performs end-to-end generation from circuit images to netlists, the proposed framework
decouples netlist recognition as an independent functional module. For a fair comparison, we evaluate only the netlist
recognition sub-module within our framework. Under this evaluation protocol, a test is considered successful if the
model generates a SPICE netlist with a correct topological structure and compliant syntax grammar from the input
image.
Table 5: Performance comparison of MasaCHAI and AnalogMaster under Qwen-VL-Max and GLM-4.6V.
Case
Qwen-VL-Max GLM-4.6V
MasaCHAI AnalogMaster MasaCHAI AnalogMaster
Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5
1 6.7 33.3 100.0 100.0 0.0 0.0 100.0 100.0
2 33.3 91.6 100.0 100.0 86.7 100.0 100.0 100.0
3 6.7 33.3 53.3 99.3 0.0 0.0 100.0 100.0
4 40.0 95.8 100.0 100.0 20.0 73.6 100.0 100.0
5 0.0 0.0 100.0 100.0 0.0 0.0 100.0 100.0
6 0.0 0.0 100.0 100.0 0.0 0.0 73.3 100.0
7 26.3 84.6 33.3 91.6 6.7 33.3 6.7 33.3
8 6.7 33.3 46.7 98.1 6.7 33.3 40.0 95.8
9 0.0 0.0 66.7 99.9 0.0 0.0 46.7 98.1
10 0.0 0.0 46.7 98.1 13.3 57.1 40.0 95.8
11 0.0 0.0 93.3 100.0 0.0 0.0 33.3 91.6
12 0.0 0.0 40.0 95.8 0.0 0.0 6.7 33.3
13 0.0 0.0 6.7 33.3 0.0 0.0 0.0 0.0
14 0.0 0.0 6.7 33.3 0.0 0.0 0.0 0.0
15 0.0 0.0 6.7 33.3 0.0 0.0 6.7 33.3
Avg 8.0 24.8 60.0 85.5 8.9 19.8 50.2 72.1
Solve 6 6 15 15 5 5 13 13
As shown in Table 5, the netlist recognition module of the proposed method significantly outperforms MasaCHAI
in terms of the Pass@k metric, demonstrating superior accuracy in circuit image understanding and architectural anal-
ysis. Further analysis revealed that MasaCHAI frequently misclassifies the type of MOSFET devices and confuses
source and drain terminals. Additionally, its relatively simple prompt design often produces netlists with inconsis-
tent formatting, resulting in parsing failures or deviations from task specifications that hinder reliable information
extraction. It is worth noting that the netlist recognition module evaluated here represents only a subset of the overall
AnalogMaster framework. Despite this, it exhibits substantially lower parameter count and computational overhead
than MasalaCHAI’s end-to-end system, highlighting its lightweight design and stronger engineering practicality.
Additionally, we conducted a focused comparison of circuit component detection performance. As shown in
Table 6, the proposed detection model significantly outperforms the version used-by MasaCHAI. This improvement is
primarily attributed to the high-quality, task-specific dataset constructed in this work. Figure ?? presents the confusion
matrix for our model, which highlights its strong performance in correctly classifying diverse circuit components.
Table 6: Performance Comparison of Component Recognition Accuracy (%)
Method AC1
BJT2
Batt3
Cap4
Cur5
DC6
Diode7
GND8
Ind9
MOS10
Res11
Volt12
ALL13
CHAI 66.9 73.1 99.0 24.7 94.5 72.7 85.3 1.7 51.4 63.2 80.1 70.2 65.2
Ours 79.0 82.4 85.3 95.7 99.4 92.9 83.1 99.1 87.0 99.4 95.4 60.3 88.3
1 AC Source; 2 Bipolar Junction Transistor; 3 Battery; 4 Capacitor; 5 Current Source; 6 DC Source; 7 Diode; 8 Ground; 9 Inductor; 10 MOSFET;
11 Resistor; 12 Voltage Source; 13All Classes.
17

## Page 20

4.8. Ablation Experiment
To systematically assess the contribution of individual components within the proposed framework, we conducted
ablation experiments using Qwen-VL-Max as the core model. The corresponding results are summarized in Table 7.
In the table, "W/O CoT" refers to the removal of the CoT3.3.1 mechanism; "W/O MICL" indicates that the proposed
MICL strategy is replaced with standard single-modal In-Context Learning (ICL). This substitution is adopted instead
of completely eliminating in-context learning, as removing contextual guidance altogether leads to highly inconsistent
netlist formats that significantly complicate downstream parsing. "W/O Intent reasoning" refers to the removal of the
intent reasoning3.3.4 mechanism. The experimental results demonstrate that removing any core component results in
a significant degradation in the overall performance of the framework.
However, while each component is indispensable, the extent of performance degradation upon its removal varies
depending on circuit complexity. Removing the CoT mechanism reduces the average Pass@1 to 39.6% and Pass@5 to
64.0%. This mechanism is indispensable for complex circuits with intricate connectivity (Cases 7, 8, and 10), whereas
it has minimal impact on simple standard circuits (Cases 1 and 2). This is because simple circuits do not require such
a complex and rigorous reasoning process to successfully complete the image-to-netlist conversion. Replacing MICL
with ICL reduces the average Pass@1 to 32.0% and Pass@5 to 55.3%. MICL is particularly effective in handling
cases where image wiring is messy and complex, even if the underlying circuit logic is not. The advantage of this
component is most evident in Case 7. Removing the Intent reasoning component leads to the most severe performance
decline. This is because, without the error-correction mechanism, the framework’s fault tolerance drops significantly.
It becomes heavily reliant on the model generating a completely correct netlist in a single attempt and performing a
correct circuit analysis to enable the Parameter Search Agent to accurately locate the appropriate parameter space,
regardless of whether the cases are simple (Case 1,3) or complex (Case 9, 10, 11), this component greatly ensures the
overall stability of the framework. Furthermore, for difficult cases (Cases 13–15), non-zero performance is observed
only under the complete framework. This confirms that each design element plays a critical and complementary role
in ensuring the effectiveness and robustness of the proposed framework.
Table 7: Experimental results of Pass@K metrics under different settings (simplified percentage)
Case
W/O CoT W/O MICL1
W/O Intent reasoning Full Framework
Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5
1 86.7 100.0 80.0 100.0 33.3 91.6 100.0 100.0
2 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
3 40.0 95.80 46.7 98.1 6.7 33.3 46.7 98.1
4 80.0 100.0 93.3 100.0 86.7 100.0 93.3 100.0
5 80.0 100.0 33.3 91.6 46.7 98.1 100.0 100.0
6 86.7 100.0 60.0 99.8 80.0 100.0 93.3 100.0
7 6.7 33.3 0.0 0.0 6.7 33.3 20.0 73.6
8 6.7 33.3 13.3 57.1 0.0 0.0 13.3 57.1
9 33.3 91.6 13.3 57.1 6.7 33.3 60.0 99.8
10 6.7 33.3 0.0 0.0 26.7 84.6 33.3 91.6
11 46.7 98.1 33.3 91.6 33.3 91.6 93.3 100.0
12 20.0 73.6 6.7 33.3 13.3 57.1 33.3 91.6
13 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
14 0.0 0.0 0.0 0.0 0.0 0.0 6.7 33.3
15 0.0 0.0 0.0 0.0 0.0 0.0 6.7 33.3
Avg 39.6 64.0 32.0 55.3 29.3 54.9 53.3 78.6
Solve 12 12 10 10 11 11 14 14
1 Replace it with ICL;
5. Conclusion
This paper presents AnalogMaster, a novel LLM-driven framework for end-to-end automation of analog IC de-
sign. The proposed framework first achieves accurate extraction of structurally complete netlists from circuit images
18

## Page 21

through a joint reasoning mechanism. To enable downstream physical design, a parameter search agent is then intro-
duced to transform isolated netlists into parameterized representations suitable for analog design workflows, overcom-
ing the limitations of conventional sizing methods that rely on manually predefined parameter ranges. Finally, Analog-
Master integrates robust classical algorithms to sequentially perform device sizing optimization, automatic placement,
and analog routing, forming a complete and coherent design pipeline. Experimental results demonstrate that Analog-
Master consistently delivers strong performance across multiple models. In particular, GPT-5 achieves success rates
of 92.9% and 99.9% on the Pass@1 and Pass@5 metrics, respectively, validating the robustness and effectiveness of
the proposed framework for full-flow analog IC design. This work establishes a comprehensive paradigm for deeply
integrating LLMs with traditional analog IC design algorithms and provides a practical foundation for highly reliable
and fully automated analog IC design.
Funding
This work was supported in part by the National Natural Science Foundation of China (92473207,92373102,62104174),
State Key Laboratory of Integrated Chips and Systems (No.SKLICS-K202506), and National Science and Technology
Major Project(No.2021ZD0114600).
CRediT authorship contribution statement
Xianrong Qin: (Co-First Author), Investigation, Conceptualization, Methodology, Writing – original draft. Yong
Zhang: (Co-First Author), Validation, Visualization. Ying Hu: Data collection, figure preparation. Tao Su: Visual-
ization. Bo-Wen Jia: (Corresponding Author), Supervision, Project administration, Writing – review & editing. Ning
Xu:(Corresponding Author), Supervision, Project administration, Resources.
Declaration of competing interest
The authors declare that they have no known competing financial interests or personal relationships that could
have appeared to influence the work reported in this paper.
Data availability
Data will be made available on request.
Acknowledgments
The authors would like to thank Prof. Jun Hu, Prof. Keren Zhu for their valuable discussion.
References
anotnio, 2024. prok computer vision dataset. https://universe.roboflow.com/anotnio/prok. [Avail-
able:2025].
Bhandari, J., Bhat, V., He, Y., Rahmani, H., Garg, S., Karri, R., 2024a. Masala-chai: A large-scale spice netlist dataset
for analog circuits by harnessing ai. arXiv preprint arXiv:2411.14299 .
Bhandari, J., Knechtel, J., Narayanaswamy, R., Garg, S., Karri, R., 2024b. Llm-aided testbench generation and bug
detection for finite-state machines. arXiv preprint arXiv:2406.17132 .
Chang, C.C., Shen, Y., Fan, S., Li, J., Zhang, S., Cao, N., Chen, Y., Zhang, X., 2024. Lamagic: Language-model-based
topology generation for analog integrated circuits. arXiv preprint arXiv:2407.18269 .
Chen, M., 2021. Evaluating large language models trained on code. arXiv preprint arXiv:2107.03374 .
19

## Page 22

Circuit Circuit Do Circuit, 2022. Cnn circuit computer vision model. https://universe.roboflow.com/
circuit-circuit-do-circuit/cnn_circuit. [Available:2025].
Gao, J., Cao, W., Yang, J., Zhang, X., 2025. Analoggenie: A generative engine for automatic discovery of analog
circuit topologies. arXiv preprint arXiv:2503.00205 .
project gtqqq, 2022. v3qwe computer vision dataset. https://universe.roboflow.com/project-gtqqq/
v3qwe. [Available:2025].
Jin, W., Wang, N., Zhang, L., Tian, X., Shi, B., Zhao, B., 2025. A review of ai-driven automation technologies: Latest
taxonomies, existing challenges, and future prospects. Computers, Materials and Continua 84, 3961–4018. URL:
https://www.sciencedirect.com/science/article/pii/S1546221825007416, doi:https://doi.org/
10.32604/cmc.2025.067857.
Lai, Y., Lee, S., Chen, G., Poddar, S., Hu, M., Pan, D.Z., Luo, P., 2025. Analogcoder: Analog circuit design via
training-free code generation, in: Proceedings of the AAAI Conference on Artificial Intelligence, pp. 379–387.
Liu, B., Zhang, H., Gao, X., Kong, Z., Tang, X., Lin, Y., Wang, R., Huang, R., 2025. Layoutcopilot: An llm-powered
multi-agent collaborative framework for interactive analog layout design. IEEE Transactions on Computer-Aided
Design of Integrated Circuits and Systems .
Liu, C., Chen, W., Peng, A., Du, Y., Du, L., Yang, J., 2024a. Ampagent: An llm-based multi-agent system
for multi-stage amplifier schematic design from literature for process and performance porting. arXiv preprint
arXiv:2409.14739 .
Liu, C., Liu, Y., Du, Y., Du, L., 2024b. Ladac: Large language model-driven auto-designer for analog circuits.
Authorea Preprints .
Liu, M., Ene, T.D., Kirby, R., Cheng, C., Pinckney, N., Liang, R., Alben, J., Anand, H., Banerjee, S., Bayraktaroglu,
I., et al., 2023a. Chipnemo: Domain-adapted llms for chip design. arXiv preprint arXiv:2311.00176 .
Liu, M., Pinckney, N., Khailany, B., Ren, H., 2023b. Verilogeval: Evaluating large language models for verilog code
generation, in: 2023 IEEE/ACM International Conference on Computer Aided Design (ICCAD), IEEE. pp. 1–8.
Liu, S., Fang, W., Lu, Y., Wang, J., Zhang, Q., Zhang, H., Xie, Z., 2024c. Rtlcoder: Fully open-source and efficient
llm-assisted rtl code generation technique. IEEE Transactions on Computer-Aided Design of Integrated Circuits
and Systems .
Lu, Y., Liu, S., Zhang, Q., Xie, Z., 2024. Rtllm: An open-source benchmark for design rtl generation with large
language model, in: 2024 29th Asia and South Pacific Design Automation Conference (ASP-DAC), IEEE. pp.
722–727.
Lyu, W., Yang, F., Yan, C., Zhou, D., Zeng, X., 2018. Batch bayesian optimization via multi-objective acquisition
ensemble for automated analog circuit design, in: International conference on machine learning, PMLR. pp. 3306–
3314.
Ma, Q., Xiao, L., Tam, Y.C., Young, E.F., 2010. Simultaneous handling of symmetry, common centroid, and general
placement constraints. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems 30, 85–
95.
Min, S., Lewis, M., Zettlemoyer, L., Hajishirzi, H., 2022. Metaicl: Learning to learn in context, in: Proceedings
of the 2022 conference of the North American chapter of the Association for Computational Linguistics: Human
Language Technologies, pp. 2791–2809.
Pan, G., Zhou, T., Zhao, J., Li, Z., Lin, Y., Ma, B., Wang, Y., Liò, P., Wang, S., 2026. Graph neural networks based
analog circuit link prediction. Engineering Applications of Artificial Intelligence 163, 113035. URL: https://
www.sciencedirect.com/science/article/pii/S0952197625030660, doi:https://doi.org/10.1016/
j.engappai.2025.113035.
20

## Page 23

Qiu, R., Zhang, G.L., Drechsler, R., Schlichtmann, U., Li, B., 2024. Autobench: Automatic testbench generation and
evaluation using llms for hdl design, in: Proceedings of the 2024 ACM/IEEE International Symposium on Machine
Learning for CAD, pp. 1–10.
Žiga Rojec, Árpád Bűrmen, Fajfar, I., 2019. Analog circuit topology synthesis by means of evolu-
tionary computation. Engineering Applications of Artificial Intelligence 80, 48–65. URL: https://
www.sciencedirect.com/science/article/pii/S0952197619300119, doi:https://doi.org/10.1016/
j.engappai.2019.01.012.
Somayaji, N.K., Hu, H., Li, P., 2021. Prioritized reinforcement learning for analog circuit optimization with design
knowledge, in: 2021 58th ACM/IEEE Design Automation Conference (DAC), IEEE. pp. 1231–1236.
Somayaji, N.K., Li, P., 2025. Llm-uso: Large language model-based universal sizing optimizer. IEEE Transactions
on Computer-Aided Design of Integrated Circuits and Systems .
Thakur, S., Ahmad, B., Pearce, H., Tan, B., Dolan-Gavitt, B., Karri, R., Garg, S., 2024. Verigen: A large language
model for verilog code generation. ACM Transactions on Design Automation of Electronic Systems 29, 1–31.
Thakur, S., Blocklove, J., Pearce, H., Tan, B., Garg, S., Karri, R., 2023. Autochip: Automating hdl generation using
llm feedback. arXiv preprint arXiv:2311.04887 .
Un, 2025. Mosfet detection computer vision dataset. https://universe.roboflow.com/un-eiyzu/
mosfet-detection-nrqmg. [Available:2025].
Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A.N., Kaiser, Ł., Polosukhin, I., 2017. Attention
is all you need. Advances in neural information processing systems 30.
Vungarala, D., Alam, S., Ghosh, A., Angizi, S., 2024. Spicepilot: Navigating spice code generation and simulation
with ai guidance, in: 2024 IEEE International Conference on Rebooting Computing (ICRC), IEEE. pp. 1–6.
Wu, H., He, Z., Zhang, X., Yao, X., Zheng, S., Zheng, H., Yu, B., 2024. Chateda: A large language model powered
autonomous agent for eda. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems 43,
3184–3197.
Xu, B., Li, S., Xu, X., Sun, N., Pan, D.Z., 2017. Hierarchical and analytical placement techniques for high-
performance analog circuits, in: Proceedings of the 2017 ACM on International Symposium on Physical Design,
pp. 55–62.
Xu, P., Tu, J., Chen, G., Zhu, K., Chen, T., Ho, T.Y., Yu, B., 2025. Paroute2: Enhanced analog routing via
performance-drive guidance generation. IEEE Transactions on Computer-Aided Design of Integrated Circuits and
Systems .
Zhang, H., Sun, S., Lin, Y., Wang, R., Bian, J., 2025a. Analogxpert: Automating analog topology synthesis by
incorporating circuit design expertise into large language models, in: 2025 International Symposium of Electronics
Design Automation (ISEDA), IEEE. pp. 772–777.
Zhang, Y., Li, W.J., Ge, G.J., Wang, J.Q., Jia, B.W., Xu, N., 2025b. Reinforcement learning-driven net order selection
for efficient analog ic routing. Integration , 102623.
Zhang, Y., Liu, A., Yin, Y., Xu, N., Jia, B., 2025c. Pin access planning-driven matching-based analog integrated
circuits routing. Microelectronics Journal , 106960.
21

