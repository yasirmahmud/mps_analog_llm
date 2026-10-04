---
title: "AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design Using Knowledge Graph RAG"
authors: "Shi, Yichen; Tao, Zhuofu; Gao, Yuhao; Zhou, Tianjia; Chang, Cheng; Wang, Yaxing; Chen, Bingyu; Zhang, Genhao; Liu, Alvin; Yu, Zhiping; Lin, Ting-Jung; He, Lei"
date: "2024/11/07"
category: "data_knowledge"
source_url: "https://arxiv.org/abs/2411.13560"
pdf: "papers/2024_amsnet-kg-a-netlist-dataset-for-llm-based-ams-circuit-auto-design-using-kno_2411-13560.pdf"
sha256: "190602742026455ef3efae68c396210104520e30fb0e0cc1fce9e5d42df0e876"
pages_extracted: 34
---

# AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design Using Knowledge Graph RAG

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG
YICHEN SHI∗, Eastern Institute of Technology, China
ZHUOFU TAO∗, University of California, Los Angeles, USA
YUHAO GAO, BTD Inc., China
TIANJIA ZHOU, CHENG CHANG, YAXING WANG, BINGYU CHEN, and GENHAO ZHANG, Uni-
versity of California, Los Angeles, USA
ALVIN LIU, BTD Inc., China
ZHIPING YU, Tsinghua University, China
TING-JUNG LIN, Institute of Digital Twin, Eastern Institute of Technology, China
LEI HE, University of California, Los Angeles, USA
High-performance analog and mixed-signal (AMS) circuits are mainly full-custom designed, which is time-consuming and labor-
intensive. A significant portion of the effort is experience-driven, which makes the automation of AMS circuit design a formidable
challenge. Large language models (LLMs) have emerged as powerful tools for Electronic Design Automation (EDA) applications,
fostering advancements in the automatic design process for large-scale AMS circuits. However, the absence of high-quality datasets
has led to issues such as model hallucination, which undermines the robustness of automatically generated circuit designs. To address
this issue, this paper introduces AMSnet-KG, a dataset encompassing various AMS circuit schematics and netlists. We construct a
knowledge graph with annotations on detailed functional and performance characteristics. Facilitated by AMSnet-KG, we propose an
automated AMS circuit generation framework that utilizes the comprehensive knowledge embedded in LLMs. We first formulate a
design strategy (e.g., circuit architecture using a number of circuit components) based on required specifications. Next, matched circuit
components are retrieved and assembled into a complete topology, and transistor sizing is obtained through Bayesian optimization.
Simulation results of the netlist are fed back to the LLM for further topology refinement, ensuring the circuit design specifications are
met. We perform case studies of operational amplifier and comparator design to verify the automatic design flow from specifications
to netlists with minimal human effort. The dataset used in this paper will be open-sourced upon publishing of this paper.
CCS Concepts: • Hardware → Design databases for EDA.
Additional Key Words and Phrases: EDA, LLM, AMSnet, Knowledge Graph, RAG, Topology Design
∗Both authors contributed equally to this research. Address comments to Lei He at Lei.Hexun@gmail.com.
Authors’ Contact Information: Yichen Shi, Eastern Institute of Technology, Ningbo, China; Zhuofu Tao, University of California, Los Angeles, Los
Angeles, USA; Yuhao Gao, BTD Inc., Ningbo, China; Tianjia Zhou; Cheng Chang; Yaxing Wang; Bingyu Chen; Genhao Zhang, University of California,
Los Angeles, Los Angeles, USA; Alvin Liu, BTD Inc., Shenzhen, China; Zhiping Yu, Tsinghua University, Beijing, China; Ting-Jung Lin, Institute of Digital
Twin, Eastern Institute of Technology, Ningbo, China; Lei He, University of California, Los Angeles, Los Angeles, USA, lhe@ee.ucla.edu.
Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not
made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components
of this work owned by others than the author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on
servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from permissions@acm.org.
© 2018 Copyright held by the owner/author(s). Publication rights licensed to ACM.
Manuscript submitted to ACM
Manuscript submitted to ACM 1
arXiv:2411.13560v1
[cs.AI]
7
Nov
2024

## Page 2

2
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
ACM Reference Format:
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin Liu, Zhiping Yu,
Ting-Jung Lin, and Lei He. 2018. AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design Using Knowledge Graph
RAG. In Proceedings of Make sure to enter the correct conference title from your rights confirmation emai (Conference acronym ’XX). ACM,
New York, NY, USA, 34 pages. https://doi.org/XXXXXXX.XXXXXXX
1 Introduction
Digital circuit synthesis has been extensively utilized in electronic design automation (EDA) [18], enabling contemporary
large-scale digital integrated circuits (ICs) in accordance with Moore’s Law. However, the degree of automation in
analog and mixed-signal (AMS) circuit design significantly lags behind that of digital design [22]. Today’s AMS circuits
are primarily full-custom designed and still heavily depend on human experts to determine circuit topologies and
component sizing. The time-consuming and labor-intensive design process significantly hinders the scalability of AMS
circuits.
Large language models (LLMs) have recently demonstrated the potential to address various EDA challenges[37],
offering new hopes for automatic AMS circuit design. However, the availability of high-quality digital circuit data
on the Internet, such as Verilog code, far exceeds that of AMS circuits mainly represented by SPICE netlists [10].
Consequently, LLMs have been extensively applied in the digital circuit domain, with significant work done in RTL code
generation[15, 27], yielding promising results. However, their application in AMS circuits is still in an exploratory phase.
Researchers have utilized LLMs to assist analog circuit topology design and transistor sizing[4, 10, 13, 35]. Compared to
traditional AI methods such as reinforcement learning and Bayesian optimization (BO), LLMs demonstrate much better
interactivity, knowledge transfer and expansion capabilities, and interpretability of design solutions.
However, due to the scarcity of public AMS circuit netlist datasets, LLMs are not sufficiently trained to produce
accurate netlists with correct topology and reasonable sizing[4, 10]. Researchers have improved the performance of
LLMs in designing AMS circuits by training them with datasets collected for specific circuits [4], or by employing prompt
engineering techniques including chain of thought (CoT) [32], ReAct [34], and in-context learning (ICL) [8, 10, 14].
Similarly, retrieval-augmented generation (RAG) is also used to supplement LLMs with additional knowledge without
requiring the expensive retraining [13].
AMSnet
dataset
specifications
design strategy
circuit component
retrieval and
assembly
sizing
(b)
(a)
KG-RAG
generate
triplet
netlist
netlist
NM1 (net3 Vin net1
net1) n09_ckt...
NM3 (net8 Vb net3
net3) n09_ckt...
PM5 (net8 net8
net12 net12)
p09_ckt...
(c) knowledge graph
(a) netlist
tag
tag
{"circuits": [ {
"id":"circuit1",
"input": "differential",
"output": "single",
"load": current
mirror...},
"id": "circuit2",
"input": "differential",
"output": single...}]}
(b) annotation
Fig. 1. AMSnet-KG dataset (left & middle) and the workflow diagram of automatic AMS circuit topology design (right). AMSnet-KG
dataset includes (a) circuit netlists, (b) corresponding annotations, and (c) an overall knowledge graph encompassing all circuits.
Given the specifications, leveraging the extensive knowledge of LLM, the flow extracts relevant relation query triplets from the
responses and retrieves the corresponding circuit topology from the knowledge graph. Automatic testbench pairing and transistor
sizing complete the circuit design flow.
Manuscript submitted to ACM

## Page 3

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 3
AMS circuit design with vanilla LLM
Task: Describe the structure of a 5 transistor opamp.
Answer:
A 5-transistor operational amplifier generally consists of the
following structural components:
• Differential Input Stage: This stage typically consists
of ...
• Active Load: Employed to increase the gain ...
• Output Stage: This stage typically consists of ...
Task: Please give me a spice netlist of 5 transistor opamp.
Answer:
* 5-Transistor Opamp Netlist
.include 'cmos_models.txt' * Include the technology file
M1 out in+ 0 0 NMOS L=1u W=10u
M2 out in- 0 0 NMOS L=1u W=10u
M3 out vdd out <missing sub> NMOS L=1u W=20u
M4 out vdd out <missing sub> PMOS L=1u W=30u
M5 vdd in+ vdd vdd PMOS L=1u W=30u
...
Fig. 2. LLM generating the full netlist directly versus describing its building blocks
Although the state-of-the-art LLMs make mistakes in directly generating netlists, they have been trained with
extensive AMS circuit knowledge in human languages. As shown in Fig. 2, they can provide promising design strategies
at a circuit component level. Transforming these strategies to full netlists then becomes a much more systematic
task. This paper addresses the critical limitations of LLMs in AMS netlist generation with a data-driven approach.
Fig. 1 presents the proposed AMSnet-KG, a comprehensive knowledge graph dataset containing netlists, architectural
description (e.g. uses cascode current mirror, provides high DM gain), pin functions (e.g. 𝑉𝑖𝑛+, 𝐼𝑟𝑒𝑓 ), and expert insights
(e.g. parameter sharing between devices, operating point requirements). The architectural descriptions are summarized
into global annotations, and are used to index circuits and match LLM-generated design strategies. The netlists are used
to circumvent LLMs’ inaccuracy in generating topology. The pin functions and expert insights are summarized into local
annotations, and are used to guide assembly and automatic sizing. Building upon AMSnet-KG, we propose AMSgen,
an automatic AMS circuit generation framework. AMSgen aims at generating fully sized netlists from performance
specifications. Given design specifications, AMSgen starts by generating high-level design strategies, describing the
circuit architecture using a number of circuit components. Matching circuit components are efficiently retrieved from
AMSnet-KG and assembled into complete a topology for simulation. Afterward, guided by the local annotations to
reduce the search space, AMSgen automatically optimizes transistor sizing through Bayesian optimization (BO). Finally,
if the generated design does not meet performance specifications, AMSgen formats the current design and achieved
performance into additional fewshot examples, and returns to the LLM-based strategy generation step. The flow
iteratively generates new designs until the performance goals are achieved.
The key contributions of this paper are as follows:
• We construct AMSnet-KG, a comprehensive dataset thoroughly annotating AMS netlists with architectural
description, pin functions, and expert insights. This labeled dataset is then arranged into a knowledge graph
facilitating effective retrieval. The raw data for AMSnet-KG sources from existing literature, and are transformed
into the proposed format.
• We propose AMSgen, an automatic AMS circuit generation framework that produces fully sized circuits from per-
formance specifications. Steps include high-level design strategy generation (e.g., generating circuit architecture
Manuscript submitted to ACM

## Page 4

4
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
with circuit components), matched components and testbench retrieval from AMSnet-KG for automated assembly
to obtain completed circuit topology, then device sizing guided by circuit annotations. Such architecture/topology
can be regenerated if performance specifications with area constraints are not met.
• We apply AMSnet-KG through AMSgen and design circuits under a 28nm technology, achieving various desired
performance specifications with minimal human effort.
The rest of the paper is organized as follows. Section 2 summarizes related work of LLMs for EDA, analog circuit
datasets, and AI based AMS topology design. Section 3 provides the details of the AMSnet-KG dataset construction.
Section 4 describes the AMSgen, the automatic AMS circuit design flow based on AMSnet-KG and KG-RAG techniques.
Finally, Section 5 presents two case studies, from required performance specifications to fully sized transistor-level
netlists. Section 6 concludes this paper and discusses future research directions.
2 Related Work
2.1 Analog Circuit Datset
Due to the scarcity of open-source AMS circuit data, LLMs cannot achieve satisfactory AMS design results. AMSnet
collects many circuit schematics from textbooks and academic papers and generates corresponding netlists[26]. AICircuit
provides nine different types of AMS circuits with circuit schematics and a large number of simulation results for
sizing[17]. [9] contains five types of AMS circuits, including schematic images, netlists, and testbenches. ALIGN provides
a comprehensive collection of AMS circuits along with some well-sized parameters[21]. AncstrGNN Benchmark collects
many analog circuit netlists without schematic images[3]. Previous open-source datasets mostly contained only circuit
schematic and netlists, which were not directly usable for LLMs. In this work, we expand and annotate the AMSnet
dataset, enabling smoother retrieval by LLM during design.
2.2 Machine Learning for AMS Circuit Design
The automatic design of AMS circuits primarily includes topology synthesis and transistor sizing. In the era before LLMs,
traditional machine learning (ML) methods for automatically designing AMS circuits were primarily divided into two
types: 1) evolutionary algorithms (EA) based methods and 2) graph-based methods. The EA-based generation of circuit
topologies, such as genetic algorithms and evolutionary algorithms, typically involves encoding circuit topologies into
forms such as trees [16] and graphs [1]. New circuit topology can be generated through genetic operations, allowing
for the evolutionary breeding of circuit designs. In graph-based methods, a comprehensive building block library is
established, and circuit topology generation is conducted through the construction of graphs. The predefined building
block library includes components such as resistors, capacitors, or any subcircuit (like current mirrors, etc.), and these
elements are combined using methods based on reinforcement learning or similar techniques. This approach allows for
efficient and intelligent assembly of complex circuit designs.
LLMs have demonstrated remarkable capabilities in numerous EDA tasks, such as Verilog and RTL code generation[28].
Recent efforts have also explored using LLMs to directly generate circuit topology [4, 10]. However, due to the scarcity
of AMS circuit topology data in pretrain datasets, these efforts have not been very effective. Artisian[4] trained an
LLM for the automatic behavior-level design of OPAMPs. However, the OPAMP-specific knowledge used to train their
LLM cannot be smoothly generalized to other types of circuits, such as comparators and LDOs. Furthermore, it models
circuits at a behavior level, resulting in higher transistor counts than a standard design. AnalogCoder [10] did not
retrain the model but used carefully designed prompts to leverage the LLM’s capability to generate netlists. Due to the
Manuscript submitted to ACM

## Page 5

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 5
precise requirements of AMS circuits, multiple attempts and iterations are needed to achieve accurate results if possible.
In this work, by using LLM for design strategy (e.g., topology category) selection rather than direct netlist generation,
our design process provides more robustness.
2.3 Sizing
The goal of analog integrated circuit sizing is to determine the parameters of circuit components, such as transistors,
within a given topology to meet design objectives. However, the substantial intra-class and inter-class variations,
coupled with varying design goals across different circuit types, render the sizing process for analog integrated circuits
particularly challenging. Researchers have extensively studied the automation of this process, primarily categorizing the
methods into two types: 1) knowledge-based and 2) optimization-based methods. For knowledge-based methods such
as [7], circuit designers create predefined schemes and equations to calculate transistor sizes. However, deriving design
schemes for every existing circuit topology is very time-consuming, and also requires continuous maintenance to keep
up with the latest process technologies. Notably, LADAC [13] designed the first LLM agent for analog circuit sizing. By
constructing an expert knowledge base, integrating ICL and CoT for decision-making, LADAC successfully derived
transistor parameters for multiple circuit topologies, and satisfied their respective design specifications. Optimization-
based methods treat the performance of circuits as a black-box function and use heuristic algorithms [11, 12, 20, 23–25, 30]
or surrogate model approaches for optimization [5, 6, 19, 33, 36]. The use of Gaussian processes as surrogate models in
optimization methods has been widely studied. ADO-LLM[35] combines an LLM with Bayesian Optimization (BO),
using both the LLM and the acquisition function to determine the next sampling point. Yet, they did not discuss how
to incorporate PDK technology nodes into their workflow, despite that PDKs play an important role in the sizing
process. Previous optimization-based methods included expert insights to reduce search space[29], but did not provide
a systematic method to store and use these insights. Therefore, we propose an universal format in this work to store
expert insights for transistor sizing.
3 AMSnet-KG Construction
This section discusses the methods used to construct the AMSnet-KG dataset. We collect raw schematics and descriptions
from literature sources, and label schematics in pages as well as components in schematics. We then propose a
connectivity detection algorithm to create the netlists. Finally, we build the knowledge graph (KG) AMSnet-KG with
collected schematics, netlists, and detailed annotations, to be defined below.
3.1 Data Collection
As shown in Fig. 3, we first collect large quantities of raw data from textbooks and academic papers. These materials
are rich in circuit schematics and verbal description, and provide sufficient foundation to form our <schematic, netlist,
annotation> dataset. To reduce the cost of manually extracting circuit schematics, we employ a semi-supervised learning
approach. We annotate bounding boxes over schematics on a subset of page images, and train an object detection model
with the labeled data. This model is then used to identify and extract all circuit schematics from the remaining pages.
Since most literature print their schematics, individual components are generally printed in a very uniform fashion,
which allows us to perform template matching. Instead of manually annotating bounding boxes on components within
schematics to form a training set as we did with literature pages, we only need to annotate a single copy of each
component type. For example, after annotating a PMOS transistor, the region of interest (RoI) within the bounding box
can be template-matched against the other schematics to quickly identify all other PMOS transistors. This drastically
Manuscript submitted to ACM

## Page 6

6
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
MP MP
MN
MN
I
VDD
VSS
VSS
C
(a)
*SPICE Netlist
M1 7 2 6 6 NMOS
M2 8 1 6 6 NMOS
M4 7 7 VDD VDD
PMOS
M3 8 7 VDD VDD
PMOS
I1 6 0
C1 0 8 1nF
(b)
(d)
web crawler
manual collection
raw data
(c)
Fig. 3. AMSnet construction pipeline. We collect a number of textbooks and papers, from which extract circuit schematic images are
extracted. Then the flow detects circuit components, symbols, and nets from images, and generates full netlist.
(a) schematic extraction and filtering, (b) component identification, (c) net identification, (d) netlist generation
Prompt: Please provide a description of each serial number on this symbol,identify symbol pin function such as
Vin, Vip, Vo, Von, Vop, and only return a JSON format response.
Input symbol：
2
1
5
3
4
5
2
6
Input symbol：
Answer：
{
"1": "Vin",
"2": "Vip",
"3": "VDD",
"4": "VSS",
"5": "Vo"
}
{
"2": "Vin",
"5": "Vip",
"6": "Vo"
}
Answer：
Fig. 4. An example of symbol pin identification by GPT4
reduces the amount of effort required for manual labeling. It is important to note that the orientation information will
be used in later steps to determine component connectivity. Therefore, we require a label for each orientation of each
component class.
Despite being more cost-effective than manual labeling, template matching remains time-consuming compared to
deep-learning models. We therefore continue with the semi-supervised approach, where we train a model using data
from template matching, and apply the model to label the remaining schematics. Unlike components, subcircuit symbols
are not always printed uniformly, such as the OPAMPs in Fig. 4. Hence, they require large numbers of manual labeling
to train a similar detection model. We manually evaluated the recall metric of the component and symbol detection
results, and the overall recall was 97%. At this point, we have the location labels and class labels for all components in
our schematics.
Manuscript submitted to ACM

## Page 7

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 7
3.2 Netlist Generation from Schematic Image
After identifying the components, we can label the net connections. The current version relies on two assumptions: 1)
all the wires are represented by solid lines on the schematic diagram, and 2) without a junction, two intersecting wires
are not considered connected. These assumptions enable us to implement the net detection algorithm as follows.
starting
pixels
starting
element
N1
Groups: [
{N1, M1},
{M1},
{M1, M2}
]
M1
M2
(a) (b) (c)
Fig. 5. Grouping neighboring components for
net detection
2D convolution create
intersection
redo
grouping
Groups: [
{N1, N5},
{N2, N5},
{N3, N5},
{N4, N5}
]
rerouting
Groups: [
{N1, N3},
{N2, N4}
]
N1
N2
N3
N4 N2
N1
N5 N4
N3
N1
N4
N5
N2
N3
Fig. 6. Resolving intersection cases in net de-
tection
source
drain
gate
Fig. 7. An example of component
pin ordering
The first step is to group all neighboring components. Starting from each pixel on each bounding box, the algorithm
expands into neighboring wire pixels in all directions, until it encounters other components. This step groups directly
components into clusters, each representing a net. Fig. 5 shows an example.
Each group has four possible cases: 1) The group contains only the starting component, as shown in the green
group in Fig. 5. In this case, no connectivity has been detected. 2) The group contains exactly two components, as
depicted by the blue and brown groups in Fig. 5; here, the two components are connected. 3) The group has an odd
number of components (more than two). This scenario likely indicates that a junction has been omitted. However, the
algorithm cannot determine which subset of the group is connected and which is intersecting. Therefore, we flag the
entire schematic as an exception; manual attention is required to correct it before it can be analyzed again.
For the last case, there are an even number (more than 2) of components in the group, as illustrated in Fig. 6. We
assume the schematic has not omitted any junctions; thus, the intersecting wires are not connected. To address this, we
locate the intersections by applying a 2D convolution to the searched wiring. Given that the area around the intersection
typically contains a higher density of wire pixels, we identify the indices with the maximum values as the intersection
point and add it to our labeled components. Later, the algorithm eliminates the four-component cluster by repeating the
grouping process, connecting each of the four components to the intersection. In the case of more than four components
in the group, each iteration reduces the group size by two until each group is left with two components eventually. It is
important to note that line weights and layout may influence this step, making the dimension of the convolution kernel
a tunable parameter. We reroute opposite connections to each other, and then delete the intersection to finalize the
process.
After all the nets are identified, the SPICE netlist format for some components requires the correct order of connections.
For example, the connections to four-terminal MOSFETs must follow the order of drain, gate, source, and body / substrate.
For components, the bounding boxes are labeled with orientation, and the algorithm can determine a range of angles
for each connection, as shown in Fig. 7. For symbols, as shown in Fig. 4, we mark a number on each pin, and use MLLM
to determine their function. The quality of the net labeling process was manually verified, which arrived at an accuracy
of 96%. Erroneous results are manually corrected to ensure data quality.
Manuscript submitted to ACM

## Page 8

8
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
At this point, the algorithm identifies circuit components, symbols, and nets in the given schematic diagrams. Using
this information, we are able to generate the unsized netlist.
（a） （b）
{ "building_blocks": [
{ "name": "cascode current mirror",
"cpnts": ["M5", "M6", "M7", "M8"] },
... ],
"pins": {"vin+": ["net8"],
"vin-": ["net9"],
"vout": ["net4"],
... },
"sizing_constraints": [
{ "cpnts": ["M1", "M2"],
"rule": "share_parameters" },
{ "cpnts": ["M3", "M4"],
"rule": "share_parameters" },
... ]
... }
{ "Circuit": {
"Name": "Telescopic OPAMP",
"Input": "Differential pair",
"Output": "Single ended",
"Load": "Cascode current mirror",
"Advantages": {
"Gain": "High",
"Output resistance": "High",
"CMRR": "High",
"PSRR": "High"},
"Drawbacks": {
"Output swing": "Low",
"Number of poles": "Increase"
...}}}
Local Annotation Global Annotation
Vin Vip
Vb
Vout
M3 M4
M1 M2
M9
VDD
M
M8
8
M
M6
6
M
M7
7
M
M5
5
Vb0
Fig. 8. (a) An example of component labeling, (b) corresponding annotations.
3.3 Circuit Annotation
In addition to schematics and netlists, we also include annotations to enrich circuit data. We define two types of
annotations: local and global, where local annotations describe one or more component(s) of the circuit, while global
annotations describe the entire architecture.
The purpose of local annotations is to guide usage of the current circuit, and is hand-labeled in this work. Some
examples include identifying nets as inputs, biases, outputs, or identifying groups of components as building blocks
such as current mirror or differential pair. An important type of local annotation is to label sizing constraints, marking
certain sets of components to follow constraints such as symmetry in lengths and widths. These constraints are usually
expert insights gained from their design experience, and play an important role in the sizing process to reduce search
space and computation costs. Net labels, on the other hand, are used to connect pins of circuits to testbenches.
The purpose of global annotations is to guide topology selection, to use a circuit as opposed to other circuits. It is
common that each individual circuit does not have a generally agreed name to identify them. Instead, they are described
by circuit components or remembered by their specialties. An engineer may describe an OPAMP with its stages, or
decide to use it for its exceptional performance in a specific area. To enable this information retrieval process, we label
each circuit with its architectural descriptions such as performance specialties and circuit subcomponents. Specifically,
we exploit the fact that the source literature where we first obtained each schematic, would most likely include some
verbal description in the context. We use this information in addition to the schematic image and invoke MLLM to
summarize qualitative description into key-value pairs, such as {gain: high} or {load: current mirror}.
Manuscript submitted to ACM

## Page 9

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 9
(e)
(a) (b)
(c) (d)
Circuit
(a)
Circuit
(b)
5-Transistor OPAMP
Name
Cascode current mirror
Load
OPAMP
Class Class
Input
Output Output
Input
Differential
Single
High
DM Gain
CMRR
PSRR
Current mirror
Load
Layout area
Low
DM gain
High High
Phase margin
Evaluates
3dB bandwidth
Evaluates
GBW
Calculates
Calculates
CM gain
Evaluates
CMRR
Calculates Calculates
Load capacitance
Sets
Testbench
(c)
Testbench
(d)
M
M3
3 M
M4
4
M
M1
1 M
M2
2
M
M5
5
V
Vin
in V
Vip
ip
V
Vb
b
V
VDD
DD
V
Vo
ou
ut
t
M7 M8
M5 M6
Vin Vip
Vb
Vout
M3 M4
M1 M2
M9
VDD
V
VDD
DD
C
CL
L
AC
AC
DC
DC
V
VSS
SS
CL
DC
AC
V
VDD
DD
V
VSS
SS
Sets
Vb0
Evaluates
Fig. 9. (a) 5-Transistor OPAMP, (b) Telescpoic cascode OPAMP, (c) Testbench for DM gain, (d) Testbench for common mode gain, (e)
Corresponding knowledge graph diagram.
We take one step further and produce JSON files with LLM to introduce more structure into our annotation data.
Fig. 8 (a) illustrates annotation on a circuit, while Fig. 8 (b) presents the local and global annotation files for the given
circuit. Since individual JSON files may share the same annotation, we simplify the data retrieval process and reduce
data volume by merging equivalent annotations to form a knowledge graph in the following section.
3.4 Knowledge Graph Construction
A knowledge graph (KG) is a structured form of knowledge representation that expresses the relationships between
entities. In a knowledge graph, nodes typically represent entities, and edges signify various semantic relationships
between these entities. The basic unit of composition is the “entity-relation-entity” relation query triplet, where entities
are interconnected through relations, forming a graph-like data structure. In this work, we collect all schematics, netlists,
and annotation data to create the AMSnet-KG dataset.
AMSnet-KG defines an entity as either a string or a circuit. The annotation keys and values are simply represented
as strings, while the circuit object contains a schematic, a netlist, a set of image attributes such as bounding boxes and
net marks, and all local annotations. Relations, on the other hand, only serve to connect entities and are therefore also
simply strings. Fig. 9 illustrates a small scale knowledge graph created from two circuits and two testbenches.
3.5 Dataset Summary
To summarize, AMSnet currently contains 894 circuits with schematic diagrams, component bounding boxes, and
netlists. AMSnet-KG selects application-specific circuits and further include local and global annotations. It currently
contains OPAMPs, comparators, bandgaps, LDOs, and ADCs, as well as related testbenches.
Manuscript submitted to ACM

## Page 10

10
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
Yes
No
Topology regeneration
KG-RAG
Spec Design strategy Extract triplet
Assemble circuit
and testbench
Sizing Sized topology
Meet
specification?
Fig. 10. Pipeline of AMS circuit design from performance specifications to fully sized transistor-level netlist
Prompt
Task : Generate a [CIRCUIT TYPE]. Analyze each constraint and provide a topology design following the given example.
Requirements ： Using [TECH NODE] process with the following requirements: [CONSTRAINT 1, CONSTRAINT 2,…,
CONSTRAINT n]
Here is an example：
Requirements：Designing [CIRCUIT TYPE] using [TECH NODE] process with the following requirements:
[CONSTRAINT 1,...,CONSTRAINT n].
Analyses:[Analysis for CONSTRAINT 1] . . . [Analysis for CONSTRAINT n]
Circuit structure:[Circuit topology structure]
Few-Shot ICL
Fig. 11. LLM conversation: from performance specification to design strategy (shortened, full version in the Appendix section B). The
green text presents CoT prompt.
4 AMSgen: AMSnet-KG Driven Automated Circuit Generation
The data support from AMSnet-KG enables us to fully automate AMS circuit design, from performance specification
to fully sized netlists. In this section, we explore our LLM-assisted, data-driven design pipeline, as shown in Fig. 10,
which autonomously selects circuit topology and simulation testbenches in section 4.1, and optimizes transistor sizing
in section 4.2. In the case where the initial topology fails to reach performance goals after sizing, we automatically
regenerate the topology in section 4.3.
4.1 Topology Selection via KG-RAG
Traditionally, analog / mixed-signal (AMS) circuits are designed manually by engineers, starting with circuit topology
and the testbench selection. At the outset, the engineer considers a set of desired performance specifications. Drawing
from their experience or supplementary materials, they identify circuit topologies that meet the design goals and select
testbenches to evaluate each performance metric. However, this process is highly labor-intensive, time-consuming,
and experience-driven, making it difficult to automate or abstract into code such as RTL. With the LLM technology
available today in addition to our AMSnet-KG data support, we are able to fully automate this process and achieve
the same goals. Given a set of performance metrics, we obtain a design strategy from the LLM which outlines the
circuit architecture as a set of circuit components. Then, we select the corresponding circuits and testbenches from
AMSnet-KG, before assembling them for simulation.
We design a comprehensive prompt engineering framework, incorporating in-context learning (ICL) [8, 14] and
chain of thought (CoT) [32], for generating circuit topology design strategy, a short verion is shown in Fig. 11, while
full versions are shown in the Appendix section B. The state-of-art LLMs possess sufficient knowledge to provide a
Manuscript submitted to ACM

## Page 11

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 11
Prompt
Task : Extract the requirements for circuit characteristics from the following [CIRCUIT REQUIREMENTS] and extract the relationships
in the form of a triple of [SUBJECT, RELATION, OBJECT].
Here is an example：
Task : Extract the requirements for the circuit from the following [CIRCUIT REQUIREMENTS] and form them into a triplet in the
form of [Subject, Relation, Object] .
Text : Description of Circuit Characteristics.
Triplets : < Subject , Relation, Object >,… , < Subject , Relation, Object >
Few-Shot ICL
Fig. 12. LLM conversation: from circuit characteristics to extract triplets (shortened, full version in the Appendix section B)
design strategy for desired circuits. However, they answer questions in natural text for human users, which may be
difficult to extract and use programmably. To standardize response format, we use an ICL strategy with an example(s),
encouraging the LLM to emulate this output format, which facilitates the further application of the response. We also
employ CoT prompting methods as they proves to effectively enhance the performance of LLMs. Specifically, the model
is instructed by example to first analyze the requirements in the specifications qualitatively, explain a thought process,
before providing the actual circuit.
After receiving the LLM’s analysis and response regarding the design specifications, we format the LLM’s responses
into relation query triplets to facilitate the retrieval of relevant circuits from AMSnet-KG, as illustrated in Fig. 12.
Similarly, to standardize the output format and reduce mismatches during the retrieval of triplets in the knowledge
graph, we design few-shot examples and input them to the LLM along with the intended question. After obtaining the
triplets, we perform searches within the knowledge graph using matching queries, such as those formulated in the
Cypher language for use in Neo4j as follows:
MATCH (node:circuit{input:’Differential’}) RETURN node ORDER BY ...
Notably, this step serves a similar purpose to the embed-and-rerank step in standard RAG. With the retrieved circuits
and testbenches, we make use of net functions preserved in local annotations, and assemble the full circuits by matching
net names with each other (i.e. first stage “Vout” to second stage “Vin”). A detailed example is shown in section 5.2.1.
This builds a fully functional netlist for simulation, and prepares us for the upcoming sizing step.
4.2 Constraint-Augmented Sizing
Now that the circuit and testbenches have been fixed, we optimize component parameters to obtain maximal performance
achievable with the given topology. In this section, we formulate sizing as a black-box optimization problem and resolve
it through Bayesian optimization (BO) with local annotations retrieved from AMSnet-KG. The details of BO are included
in Appendix section A.
4.2.1 Problem Formulation. The standard constrained optimization problem is shown in Equation (1):
maximize 𝐹𝑜𝑀(𝑥)
s.t. 𝑔𝑗 (𝑥) ≤ 0, ∀𝑗 ∈ {1, . . . , 𝑝}
(1)
Manuscript submitted to ACM

## Page 12

12
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
where figure of merits (FoM) is the objective function, 𝑔𝑗 (𝑥) is the 𝑗-th performance constraint. Following [31, 33], we
define FoM as weighted square sum of the normalized performance value as shown in (2):
𝐹𝑜𝑀 =
𝑁
∑︁
𝑖=0
𝑤𝑖 ×
min(𝑓𝑖 (𝑥), 𝑓 bound
𝑖 ) − 𝑓 min
𝑖
𝑓 max
𝑖 − 𝑓 min
𝑖
(2)
where 𝑓𝑖 (𝑥) is 𝑖-th simulated performance, 𝑓 bound
𝑖 is performance limit which we do not get additional merit after it
has been reached, 𝑓 min
𝑖 and 𝑓 max
𝑖 are normalization factors obtained through an initial set of random samples, and 𝑤𝑖 is
the term weight to control the importance of each performance metric.
BO is a powerful black-box optimization method to search for the optimal transistor sizing[33], by balancing between
exploration and exploitation. BO uses a series of initial data to define a surrogate model (typically a Gaussian process),
selects the next sampling point by maximizing the acquisition function, and updates the surrogate model using the new
sampled data. Exhaustively searching parameters for all transistors is inefficient, and therefore we use expert insights
as constraints to the search space. The local annotations stored in AMSnet-KG can effectively reduce the parameter
search space during exploration and exploitation. For instance, with the annotation that a set of transistor parameters
should maintain equality or a certain ratio, we reduce the number of parameters accordingly. The algorithm flow for
constraint-augmented sizing is as follows:
Algorithm 1: Constraint-Augmented Sizing for Analog Circuit Design
1 inputs: initial data size 𝑁𝑖𝑛𝑖𝑡 , number of iterations 𝑁𝑖𝑡𝑒𝑟 , constraint-augmented parameter design space 𝑆𝑝𝑎𝑟𝑎𝑚;
2 𝑥𝑖𝑛𝑖𝑡 ← sample(𝑆𝑝𝑎𝑟𝑎𝑚, 𝑁𝑖𝑛𝑖𝑡 );
3 𝑦𝑖𝑛𝑖𝑡 ← performance output from simulation with 𝑥𝑖𝑛𝑖𝑡 ;
4 fit GP(𝑥𝑖𝑛𝑖𝑡,𝑦𝑖𝑛𝑖𝑡 );
5 for 𝑡 ← 0 to 𝑁𝑖𝑡𝑒𝑟 do
6 // select next points for simulation via acquisition function 𝛼(𝑥)
7 𝑥𝑛𝑒𝑥𝑡 ← arg max𝑥 𝛼(𝑥);
8 𝑦𝑛𝑒𝑥𝑡 ← performance output from simulation with 𝑥𝑛𝑒𝑥𝑡 ;
9 fit GP(𝑥𝑛𝑒𝑥𝑡,𝑦𝑛𝑒𝑥𝑡 );
10 end
11 return optimal sizing 𝑥*;
4.3 Topology Regeneration
In the case where an initial topology design does not satisfy the desired metrics after a set amount of sizing effort (or
it costs too much circuit area for the desired results), we consider this a topology issue and begin regeneration. The
topology regeneration process is very similar to the initial topology design process. The only difference is that instead
of using a different circuit application in the fewshot examples, here we use actual performance specifications obtained
from the previous design(s) to give the LLM additional quantitative knowledge.
With the previous design strategy proposed by the LLM and the optimal performance during simulation, we reverse
the cause-effect and pretend that we initially wanted to achieve the performance metrics, and then correctly obtained
the previous design. For example, suppose we desire an OPAMP with a DM gain of 60 dB, the LLM initially proposed a
design strategy to retrieve circuit components and assemble circuit A. Then we go through the process outlined in
section 4.2, and finally ended with a fully sized design that only achieves a DM gain of 40 dB. We would now inject
Manuscript submitted to ACM

## Page 13

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 13
a fewshot example where we desire a design that achieves a DM gain of 40 dB, and then obtained circuit A; and then
prompt the LLM for a new design that achieves 60 dB instead. The LLM would then understand to propose a topology
with higher expected gain.
This approach is especially effective since initially the state-of-art LLM does not possess sufficient quantitative
knowledge. It is unable to accurately infer the performance of a specific circuit topology on a specific technology
node, and therefore generally begins with a balanced design that has no particular weakness. However, it does possess
qualitative and comparative knowledge between different topology, as they generally hold across all technology nodes.
In this example where we require a higher gain, it is often able to correctly suggest a new topology that specializes
in higher gain. With the new topology, we simply replay netlist retrieval, assembly, and sizing for the same set of
performance specifications. This process is repeated until we either achieve the intended performance goals or exhaust
all possible topologies, where we either finalize our design or label the input as impossible to achieve.
5 Experiments
5.1 Experiment Setup
We use YOLO-V8 for object detection during AMSnet-KG construction, Neo4j for knowledge graph implementation,
and GPT-4 for various inquiries throughout dataset construction and circuit design. Our experiments are performed
using a 28nm technology from SMIC, which restricts our transistor length, width, and number of fingers to [30nm,
1𝜇m], [100nm, 3𝜇m], and [1, 100] respectively. We use Bayesian optimization (BO) implemented by the Optuna [2]
library, and limit our total simulation count to 2000.
5.2 Case Study of OPAMP Topology Design
5.2.1 Topology selection.
Initially, we request an OPAMP design to achieve the performance goals in Equation 3.
Gain > 80 dB, CMRR > 80 dB, PSRR > 80 dB, GBW > 10 MHz, PM > 60◦
, 𝐶𝐿 = 100 pF (3)
In order to keep any incorrect numerical information from the in-context learning, we use a different type of circuit
to offer the reasoning format and chain of thought. Specifically, we use a comparator design flow as a fewshot example.
Fig. 19 and Fig. 20 in the Appendix section B.1 presents the full conversation. In this case, the LLM responds with the
design strategy of a two-stage OPAMP, where the first stage is a differential amplifier with a current mirror load, the
second stage is a common-source amplifier, and a Miller compensation is to be added between the outputs of the first
and second stage.
We then convert this design strategy to a set of relation query triplets, Fig. 21 and Fig. 22 in the Appendix section B.1
presents the full conversation. The triplets are then used to query AMSnet-KG and retrieve circuits. For example, <,
input, differential input pair>, <, load, PMOS current mirror> retrieves a single stage 5-transistor OPAMP. Similarly,
we retrieve the common source amplifier, the bias circuitry with a current source and an NMOS transistor, and the
sequentially connected resistor-capacitor pair for Miller compensation. Using the local annotations on each circuit
component, we connect the output from the first stage to the input of the second stage, the bias voltage of both stages
to the bias circuitry, and a Miller compensation to the output of the first stage and the output of the second stage. The
fully assembled 2-stage OPAMP is shown in Fig. 13.
Manuscript submitted to ACM

## Page 14

14
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
VDD
IREF
f -1
(IREF)
M1
VDD
IREF
f -1
(IREF)
M1
M3 M4
M1 M2
M5
Vin
Vip
VDD
Vout
M6
I1
M8
M7
R C
M3 M4
M1 M2
M5
Vin Vip
Vb
VDD
Vout
VDD
Vb
Vin
Vout
M1
M2
(a) (b) (d) (e)
(c)
R C
Fig. 13. Visualization of the retrieved circuit components followed by the assembled result, the purple arrows illustrate circuit
component -level connectivity. (a) bias circuitry, (b) first stage, (c) Miller compensation, (d) second stage, (e) assembled 2-stage
OPAMP, symmetric transistors identified from local annotations are highlighted in the same color.
Similarly, we convert the required performance metrics to another set of relation triplets. Using this information,
we retrieve three testbench circuits to test the desired metrics, including DM gain, CM gain, PS gain. The testbench
circuits are connected to our 2-stage OPAMP by matching net annotations similar to above. We setup SPICE simulation
with the Spectre circuit simulator by Cadence, and ensure that the invocation can be done programmably to support
the acquisition function used by BO in the next step. At this point, we are ready to perform simulation and collect
performance metrics.
5.2.2 Sizing. The given specifications forms the FoM in Equation 4, where 𝑓𝑔𝑎𝑖𝑛 (𝑥), 𝑓𝑐𝑚𝑟𝑟 (𝑥), 𝑓𝑝𝑠𝑟𝑟 (𝑥), 𝑓𝑔𝑏𝑤 (𝑥), and
𝑓𝑝𝑚 (𝑥) each represent the simulated gain, CMRR, PSRR, gain-bandwidth product, and phase margin. Each 𝑓 𝑚𝑖𝑛 and
𝑓 𝑚𝑎𝑥 represent the corresponding minimum and maximum metric for normalization purposes, and are observed within
an initial set of 100 simulations. Note that since the GBW distribution is exponential, we first take the decimal log of
each value before using it for FoM to prevent it from overtaking other values. As for phase margin, instead of requiring
a high value, we actually want the value to be as close to 60 degrees as possible, therefore we introduce its distance
away from 60 degrees as a penalty.
𝐹𝑜𝑀 =
min(𝑓𝑔𝑎𝑖𝑛 (𝑥), 80) − 𝑓 min
𝑔𝑎𝑖𝑛
𝑓 max
𝑔𝑎𝑖𝑛 − 𝑓 min
𝑔𝑎𝑖𝑛
+
min(𝑓𝑐𝑚𝑟𝑟 (𝑥), 80) − 𝑓 min
𝑐𝑚𝑟𝑟
𝑓 max
𝑐𝑚𝑟𝑟 − 𝑓 min
𝑐𝑚𝑟𝑟
+
min(𝑓𝑝𝑠𝑟𝑟 (𝑥), 80) − 𝑓 min
𝑝𝑠𝑟𝑟
𝑓 max
𝑝𝑠𝑟𝑟 − 𝑓 min
𝑝𝑠𝑟𝑟
+
min(lg(𝑓𝑔𝑏𝑤 (𝑥)), 7) − lg(𝑓 min
𝑔𝑏𝑤
)
lg(𝑓 max
𝑔𝑏𝑤
) − lg(𝑓 min
𝑔𝑏𝑤
)
−
|𝑓𝑝𝑚 (𝑥) − 60|
𝑓 max
𝑝𝑚 − 𝑓 min
𝑝𝑚
(4)
As shown in Fig. 13(e), our design consists of 5 NMOS transistors and 3 PMOS transistors, each of which is described
by two parameters: gate length (𝐿) and gate width (𝑊 ). In addition, we have a current source, a capacitor, and a resistor,
each described by a single parameter, for a total of 19 free parameters.
Insights from experienced engineers could effectively reduce the complexity of the sizing process, such as the
parameter search space. We make use of the local annotations stored in AMSnet-KG, as shown in Fig. 13. Transistors
with the same highlight color share the same set of 𝐿 and 𝑊 . This way, our effective number of free parameters reduces
from 19 to 15, and is more manageable.
Fig. 14 shows the optimization trajectory for the initial design, comparing between constraint-augmented sizing
versus vanilla sizing. We can see that with a smaller number of free parameters, convergence happens much earlier. A
Manuscript submitted to ACM

## Page 15

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 15
0 500 1000 1500 2000
Iteration
0.0
1.0
2.0
3.0
FoM
Initial design optimization trajectory
With local annotations
Without local annotations
Fig. 14. Bayesian optimization trajectory
Topology BO constraint FoM ↑ DM gain (dB)↑ GBW (Hz)↑ CMRR (dB)↑ PSRR (dB)↑ Phase margin (◦)
Specification - - - 80.00 1.00 × 107 80.00 80.00 60.00
(a) 2-stage W/o local annotations 2.73 60.06 3.91 × 107 17.15 34.47 73.15
(b) 2-stage W/ local annotations 3.40 66.21 3.19 × 108 54.20 69.82 63.63
(c) 2-stage high-gain W/o local annotations 2.55 45.95 5.05 × 107 20.62 59.89 63.24
(d) 2-stage high-gain W/ local annotations 3.60 80.85 1.43 × 107 99.04 91.76 60.48
Table 1. Designs optimized by Bayesian optimization
final design with the highest FoM is shown in Table 1 rows (a) and (b) for the two attempts. We can see that attempt (b)
significantly outperforms attempt (a), which demonstrates that the retrieved local annotations yields final performance
improvements in addition to convergence time.
Unfortunately for this initial topology, the optimal sizing we obtained is not enough to satisfy our specification
requirements. The gain value is only 66.21 dB, noticeably lower than the required 80 dB. Additionally, the CMRR and
PSRR values also do not meet requirements. While it is possible that our sizing step did not reach the global optimal set
of parameters for this topology, the more straightforward approach would be to regenerate our circuit topology.
5.2.3 Topology Regeneration. The regeneration process for LLM to propose another design strategy is very similar to
the initial strategy proposal. Here, instead of using for performance specification requirements and design structure from
other applications such as comparators in fewshot examples, we are now able to simply use the previous, unsuccessful
design. This allows us to make a numerical comparison between the two circuits for a specification, and thus leverage the
LLM’s vast knowledge on comparative performance between circuits. As shown in Fig. 23 and Fig. 24 in the Appendix
section B.2, we reverse the cause-effect order by pretending that we initially wanted a design to achieve a DM gain of
66.21 dB, gain-bandwidth product of 319MHz, etc. The analysis and design strategy would be the same as the previous
circuit topology. Afterwards, by asking the same question, the LLM naturally gains a quantitative comparison between
the existing performance specification in the fewshot example and the actual prompt. This resolves our previous issue
where the LLM has insufficient knowledge regarding the specific technology node we are using.
Manuscript submitted to ACM

## Page 16

16
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
M7 M8
M5 M6
Vb0
Vin
Vip
Vout
M3 M4
M1 M2
M9
VDD
I1
R
C
M10
M12
M11
M7 M8
M5 M6
Vb0
Vin Vip
Vb
Vout
M3 M4
M1 M2
M9
VDD
VDD
Vb
Vin
Vout
M1
M2
VDD
Vb
Vin
Vout
M1
M2
(a)
(b) (d) (e)
VDD
IREF
f -1
(IREF)
M1
VDD
IREF
f -1
(IREF)
M1 (c)
R C
Fig. 15. Visualization of the retrieved circuit components followed by the assembled result, the purple arrows illustrate circuit
component -level connectivity. (a) bias circuitry, (b) first stage, (c) Miller compensation, (d) second stage, (e) regenerated 2-stage
OPAMP, symmetric transistors are highlighted in the same color.
0 500 1000 1500 2000
Iteration
0.0
1.0
2.0
3.0
FoM
Regenerated design optimization trajectory
With local annotations
Without local annotations
Fig. 16. Bayesian optimization trajectory for regenerated design for higher DM gain
In our case, the DM gain value was too low, and therefore the natural reaction of an engineer would be to select a
different topology that specializes in higher DM gain. The LLM follows a similar thought process and decides to update
the first stage from the five-transistor design to a telescopic cascode, and keeps the rest of the design as-is. We then use
this information in the same AMSnet-KG retrieval process as before, followed by an assembly process based on local
annotations. The result circuit is illustrated in Fig. 15.
The sizing process is mostly the same as before, the only difference being that now there is a number of new
transistors and a bias voltage. We first identify symmetry constraints, reduce the number of free parameters from 27
to 19. As we can see in Fig. 16, the optimization trajectory is very similar in shape as that of the previous topology.
Quantitatively, the FoM score we achieve is much better at almost zero, indicating that we met every design goal. On
the other hand, convergence happens later due to the higher parameter count and stricter specifications. Once again,
Manuscript submitted to ACM

## Page 17

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 17
VDD
M0
clk
Vin Vip
M1 M2
M3 M4
M9 M6 M8
M6 M8
M5
M7 M5
M7 M10
Von
Vop
clk clk
Fig. 17. Visualization of the retrieved strong-arm latch comparator, symmetric transistors are highlighted in the same color.
the sizing attempt without constraining parameters takes much longer to gain FoM, and did not produce an acceptable
design at the end of our 2000 iteration limit.
The final performance is shown in Table 1 row (d), as we can see we met every design goal initially intended, proving
our process successful.
5.3 Case Study of Comparator Topology Design
5.3.1 Topology selection. In order to evaluate the AMSgen workflow on different applications, we introduce another
case study on comparators. Similar to generating OPAMPs, we begin by requesting a comparator design from LLMs,
using the same PDK and constraints, and a comparator-specific set of performance specifications as follows:
Sampling frequency = 1GHz, Offset voltage < 100 uV, Propagation delay ≤ 1 ns, Power ≤ 100 uW
The LLM first responds with the use of a latch comparator. Similar to the OPAMP case study, we then take this
response, convert it into relation query triplets, and search within AMSnet-KG for matching topologies.
This time, we arrive at two different types of comparator topologies: strong-arm latch comparator and double-tail
latch comparator. For tie-breaking purposes, we feed the two names back into the LLM to make a final selection, and
arrive at the strongarm latch comparator. The conversations are shown in the Appendix section B.3 in Fig. 27 through
Fig. 31.
5.3.2 Sizing.
𝐹𝑜𝑀 = −
max(lg(𝑓𝑜𝑣 (𝑥)), −4) − lg(𝑓 max
𝑜𝑣 )
lg(𝑓 max
𝑜𝑣 ) − lg(𝑓 min
𝑜𝑣 )
−
max(lg(𝑓𝑝𝑑 (𝑥)), −9) − lg(𝑓 max
𝑝𝑑
)
lg(𝑓 max
𝑝𝑑
) − lg(𝑓 min
𝑝𝑑
)
−
max(lg(𝑓𝑝 (𝑥)), −4) − lg(𝑓 max
𝑝 )
lg(𝑓 max
𝑝 ) − lg(𝑓 min
𝑝 )
(5)
Similar to the OPAMP case study, we employ Bayesian optimization to obtain the optimal sizing using the FoM in
Equation 5, where 𝑓𝑜𝑣 (𝑥), 𝑓𝑝𝑑 (𝑥), 𝑓𝑝 (𝑥) each represent the simulated offset voltage, propagation delay, and power for
given sizing 𝑥. Every metric is viewed on a logarithmic scale, similar to GBW in the OPAMP case study. A similar
set of local annotations are used to guide sizing constraints. As shown in Fig. 17, there are a total of 5 PMOS and 6
NMOS transistors. Each of them is parameterized by a length and a width for a total of 22 parameters. After considering
symmetry, the number of free parameters drop to 10.
Manuscript submitted to ACM

## Page 18

18
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
0 500 1000 1500 2000
Iteration
0.0
0.5
1.0
1.5
2.0
2.5
3.0
FoM
Optimization trajectory
With local annotations
Without local annotations
Fig. 18. Bayesian optimization trajectory for strong-arm latch comparator
Topology BO constraint FoM ↑ Offset voltage (V)↓ Propagation delay (s)↓ Power (W)
Specification - - - 1.00 ×10−4 1.00 ×10−9 1.00 ×10−4
(a) Strong-arm W/o local annotations 1.61 7.98 ×10−4 5.81 ×10−10 3.42 ×10−4
(b) Strong-arm W/ local annotations 2.89 3.50 ×10−5 1.19 ×10−11 8.84 ×10−5
Table 2. Designs optimized by Bayesian optimization
The optimization trajectory is shown in Fig. 18. Again similar to the OPAMP case study, having local annotations
significantly improves convergence time as well as final performance. As shown in Table 2(b), all desired performance
specs have been met. This case study provides another data point to prove the generality of AMSgen.
6 Conclusions and Disscussions
In this paper, we introduce a new high-quality dataset for AMS circuits, titled AMSnet-KG, which includes schematics,
netlists, and manual annotations, all presented as a knowledge graph. We also propose an AMS circuit topology design
process based on LLM and KG-RAG. This process begins from input performance specifications into LLM to obtain
a design strategy (e.g., circuit architecture), which is then transformed into relation query triplets. Relevant circuit
components and testbenches are then retrieved from the knowledge graph. After assembling a complete circuit using
retrieved components, parameter sizing is finalized using Bayesian optimization based on design constraints. If the
resulting design fails to meet specifications (or costs too much circuit area to meet specifications), the design strategy
is adjusted and the topology is refined. We have experimented two case studies and obtained desired OPAMP and
Comparator designs.
In the future, we plan to enrich AMSnet-KG with additional information, such as specific parameter sets with
corresponding circuit performance. We also plan to introduce new circuits types and additional topologies. The
increasing size and dimension could support more application scenarios, such as training AMS-specific foundation
models. We will also develop more efficient performance modeling/prediction with reduced SPICE simulation runs or
no SPICE simulation for the sizing procedure, and develop better sizing algorithms compared to existing algorithms.
Manuscript submitted to ACM

## Page 19

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 19
References
[1] Josef SLEZ AK and Jirı PETR ˇZELA. 2014. Evolutionary synthesis of cube root computational circuit using graph hybrid estimation of distribution
algorithm. Radioengineering 23, 1 (2014), 549.
[2] Takuya Akiba, Shotaro Sano, Toshihiko Yanase, Takeru Ohta, and Masanori Koyama. 2019. Optuna: A next-generation hyperparameter optimization
framework. In Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining. 2623–2631.
[3] Hao Chen, Keren Zhu, Mingjie Liu, Xiyuan Tang, Nan Sun, and David Z Pan. 2021. Universal symmetry constraint extraction for analog and
mixed-signal circuits with graph neural networks. In 2021 58th ACM/IEEE Design Automation Conference (DAC). IEEE, 1243–1248.
[4] Zihao Chen, Jiangli Huang, Yiting Liu, Fan Yang, Li Shang, Dian Zhou, and Xuan Zeng. 2024. Artisan: Automated Operational Amplifier Design via
Domain-specific Large Language Model. In 2024 61th ACM/IEEE Design Automation Conference (DAC). 1–6.
[5] Tianchen Gu, Jiaqi Wang, Zhaori Bi, Changhao Yan, Fan Yang, Yajie Qin, Tao Cui, and Xuan Zeng. 2024. tSS-BO: Scalable Bayesian Optimization for
Analog Circuit Sizing via Truncated Subspace Sampling. In 2024 Design, Automation & Test in Europe Conference & Exhibition (DATE). IEEE, 1–6.
[6] Biao He, Shuhan Zhang, Fan Yang, Changhao Yan, Dian Zhou, and Xuan Zeng. 2020. An efficient bayesian optimization approach for analog circuit
synthesis via sparse gaussian process modeling. In 2020 Design, Automation & Test in Europe Conference & Exhibition (DATE). IEEE, 67–72.
[7] Nuno Horta. 2002. Analogue and mixed-signal systems topologies exploration using symbolic methods. Analog Integrated Circuits and Signal
Processing 31 (2002), 161–176.
[8] Yushi Hu, Chia-Hsuan Lee, Tianbao Xie, Tao Yu, Noah A Smith, and Mari Ostendorf. 2022. In-context learning for few-shot dialogue state tracking.
arXiv preprint arXiv:2203.08568 (2022).
[9] Infineon. [n. d.]. adsbenchmark. https://rb.gy/1m2263.
[10] Yao Lai, Sungyoung Lee, Guojin Chen, Souradip Poddar, Mengkang Hu, David Z Pan, and Ping Luo. 2024. AnalogCoder: Analog Circuit Design via
Training-Free Code Generation. arXiv preprint arXiv:2405.14918 (2024).
[11] Tuotian Liao and Lihong Zhang. 2020. Efficient Parasitic-aware gm/ID-based Hybrid Sizing Methodology for Analog and RF Integrated Circuits.
ACM Trans. Des. Autom. Electron. Syst. 26, 2, Article 10 (Oct. 2020), 31 pages. https://doi.org/10.1145/3416946
[12] Bo Liu, Yan Wang, Zhiping Yu, Leibo Liu, Miao Li, Zheng Wang, Jing Lu, and Francisco V Fernández. 2009. Analog circuit optimization system
based on hybrid evolutionary algorithms. Integration 42, 2 (2009), 137–148.
[13] Chengjie Liu, Yijiang Liu, Yuan Du, and Li Du. 2024. LADAC: Large Language Model-driven Auto-Designer for Analog Circuits. Authorea Preprints
(2024).
[14] Jiachang Liu, Dinghan Shen, Yizhe Zhang, Bill Dolan, Lawrence Carin, and Weizhu Chen. 2021. What Makes Good In-Context Examples for GPT-3?
arXiv preprint arXiv:2101.06804 (2021).
[15] Yao Lu, Shang Liu, Qijun Zhang, and Zhiyao Xie. 2024. Rtllm: An open-source benchmark for design rtl generation with large language model. In
2024 29th Asia and South Pacific Design Automation Conference (ASP-DAC). IEEE, 722–727.
[16] Trent McConaghy, Pieter Palmers, Michiel Steyaert, and Georges GE Gielen. 2011. Trustworthy genetic programming-based synthesis of analog
circuit topologies using hierarchical domain-specific building blocks. IEEE Transactions on Evolutionary Computation 15, 4 (2011), 557–570.
[17] Asal Mehradfar, Xuzhe Zhao, Yue Niu, Sara Babakniya, Mahdi Alesheikh, Hamidreza Aghasi, and Salman Avestimehr. 2024. AICircuit: A Multi-Level
Dataset and Benchmark for AI-Driven Analog Integrated Circuit Design. arXiv preprint arXiv:2407.18272 (2024).
[18] Petra Michel, Ulrich Lauther, and Peter Duzy. 1992. The synthesis approach to digital system design. Vol. 170. Springer Science & Business Media.
[19] Karthik Somayaji NS and Peng Li. 2024. Pareto Optimization of Analog Circuits Using Reinforcement Learning. ACM Trans. Des. Autom. Electron.
Syst. 29, 2, Article 37 (Feb. 2024), 14 pages. https://doi.org/10.1145/3640463
[20] Rodney Phelps, Michael Krasnicki, Rob A Rutenbar, L Richard Carley, and James R Hellums. 2000. Anaconda: simulation-based synthesis of analog
circuits via stochastic pattern search. IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems 19, 6 (2000), 703–717.
[21] ALIGN Project. [n. d.]. ALIGN Public Repository. https://github.com/ALIGN-analoglayout/ALIGN-public.
[22] Rob A. Rutenbar. 2015. Analog Circuit and Layout Synthesis Revisited. In Proceedings of the 2015 Symposium on International Symposium on Physical
Design, Azadeh Davoodi and Evangeline F. Y. Young (Eds.). ACM, 83. https://doi.org/10.1145/2717764.2717780
[23] Enes Saundefinedlican and Engin Afacan. 2023. MOEA/D vs. NSGA-II: A Comprehensive Comparison for Multi/Many Objective Analog/RF
Circuit Optimization through a Generic Benchmark. ACM Trans. Des. Autom. Electron. Syst. 29, 1, Article 15 (Nov. 2023), 23 pages. https:
//doi.org/10.1145/3626096
[24] Enes Saǧlican and Engin Afacan. 2023. MOEA/D vs. NSGA-II: A Comprehensive Comparison for Multi/Many Objective Analog/RF Circuit
Optimization through a Generic Benchmark. ACM Transactions on Design Automation of Electronic Systems 29, 1 (2023), 1–23.
[25] Ling-Yen Song, Chih-Yun Chou, Tung-Chieh Kuo, Chien-Nan Liu, and Juinn-Dar Huang. 2022. Machine Learning Assisted Circuit Sizing Approach
for Low-Voltage Analog Circuits with Efficient Variation-Aware Optimization. ACM Trans. Des. Autom. Electron. Syst. 28, 2, Article 18 (Dec. 2022),
22 pages. https://doi.org/10.1145/3567422
[26] Zhuofu Tao, Yichen Shi, Yiru Huo, Rui Ye, Zonghang Li, Li Huang, Chen Wu, Na Bai, Zhiping Yu, Ting-Jung Lin, and Lei He. 2024. AMSNet: Netlist
Dataset for AMS Circuits. arXiv preprint arXiv:2405.09045 (2024).
[27] Shailja Thakur, Baleegh Ahmad, Zhenxing Fan, Hammond Pearce, Benjamin Tan, Ramesh Karri, Brendan Dolan-Gavitt, and Siddharth Garg. 2023.
Benchmarking large language models for automated verilog rtl code generation. In 2023 Design, Automation & Test in Europe Conference & Exhibition
(DATE). IEEE, 1–6.
Manuscript submitted to ACM

## Page 20

20
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
[28] Shailja Thakur, Baleegh Ahmad, Hammond Pearce, Benjamin Tan, Brendan Dolan-Gavitt, Ramesh Karri, and Siddharth Garg. 2024. VeriGen:
A Large Language Model for Verilog Code Generation. ACM Trans. Des. Autom. Electron. Syst. 29, 3, Article 46 (April 2024), 31 pages. https:
//doi.org/10.1145/3643681
[29] Yannick Uhlmann, Till Moldenhauer, and Jürgen Scheible. 2023. Differentiable Neural Network Surrogate Models for gm/ID-based Analog IC Sizing
Optimization. In 2023 ACM/IEEE 5th Workshop on Machine Learning for CAD (MLCAD). 1–6. https://doi.org/10.1109/MLCAD58807.2023.10299834
[30] R Acar Vural and Tülay Yildirim. 2012. Analog circuit sizing via swarm intelligence. AEU-International journal of electronics and communications 66,
9 (2012), 732–740.
[31] Hanrui Wang, Kuan Wang, Jiacheng Yang, Linxiao Shen, Nan Sun, Hae-Seung Lee, and Song Han. 2020. GCN-RL circuit designer: Transferable
transistor sizing with graph neural networks and reinforcement learning. In 2020 57th ACM/IEEE Design Automation Conference (DAC). IEEE, 1–6.
[32] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting
elicits reasoning in large language models. Advances in Neural Information Processing Systems 35 (2022), 24824–24837.
[33] Wei W Xing, Weijian Fan, Zhuohua Liu, Yuan Yao, and Yuanqi Hu. 2024. KATO: Knowledge Alignment and Transfer for Transistor Sizing of
Different Design and Technology. arXiv preprint arXiv:2404.14433 (2024).
[34] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2022. React: Synergizing reasoning and acting in
language models. arXiv preprint arXiv:2210.03629 (2022).
[35] Yuxuan Yin, Yu Wang, Boxun Xu, and Peng Li. 2024. ADO-LLM: Analog Design Bayesian Optimization with In-Context Learning of Large Language
Models. arXiv preprint arXiv:2406.18770 (2024).
[36] Aidong Zhao, Tianchen Gu, Zhaori Bi, Fan Yang, Changhao Yan, Xuan Zeng, Zixiao Lin, Wenchuang Hu, and Dian Zhou. 2024. D3PBO: Dynamic
Domain Decomposition-based Parallel Bayesian Optimization for Large-scale Analog Circuit Sizing. ACM Trans. Des. Autom. Electron. Syst. 29, 3,
Article 44 (March 2024), 25 pages. https://doi.org/10.1145/3643811
[37] Ruizhe Zhong, Xingbo Du, Shixiong Kai, Zhentao Tang, Siyuan Xu, Hui-Ling Zhen, Jianye Hao, Qiang Xu, Mingxuan Yuan, and Junchi Yan. 2023.
Llm4eda: Emerging progress in large language models for electronic design automation. arXiv preprint arXiv:2401.12224 (2023).
Manuscript submitted to ACM

## Page 21

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 21
A Background
A.1 Bayesian Optimization
Bayesian optimization (BO) is a strategy for global optimization, particularly suitable for optimizing black-box functions,
usually employed when the function evaluation is expensive or the search space is too large to exhaust. BO treats
the black-box function to be optimized as a stochastic process, typically modeled using a Gaussian Process (GP) as a
surrogate model. The GP provides a flexible way to describe the distribution of the black-box function and updates
predictions based on existing data points.
By sampling initial data points, the GP model is trained to fit the black-box function. An acquisition function is
selected to balance exploration and exploitation, determining the next sampling point. In the input space, the point
that maximizes the acquisition function is found. The selected new point is then evaluated through simulation, and its
result is added to the existing dataset, updating the GP model. By balancing exploration and exploitation, BO can avoid
getting trapped in local optima and find the global optimum.
A.2 Gaussian Process
A GP is a stochastic process defined over an input space, where any finite subset of random variables follows a
multivariate Gaussian distribution. Given a training dataset {(𝑥𝑖,𝑦𝑖)}𝑛
𝑖=1, where 𝑥𝑖 are the inputs and 𝑦𝑖 are the target
values. A GP is completely defined by its mean function 𝜇(𝑥) and covariance function 𝑘(𝑥,𝑥′). The mean function
represents the expected value of the function at a given input, and the covariance function (also called the kernel
function) represents the correlation or similarity between any two points. Generally, the mean function of a GP is
assumed to be zero:
𝑚(𝑥) = E[𝑓 (𝑥)] = 0 (6)
The kernel function 𝑘(𝑥,𝑥′) is defined as:
𝑘(𝑥,𝑥′
) = E[(𝑓 (𝑥) − 𝑚(𝑥))(𝑓 (𝑥′
) − 𝑚(𝑥′
))] (7)
For a new input point 𝑥∗, the predictive distribution is also a Gaussian distribution. The mean and variance of the
prediction can be computed as follows:
𝜇∗ = 𝑘(𝑥∗,𝑋)𝑇
𝐾−1
𝑦 (8)
𝜎2
∗ = 𝑘(𝑥∗,𝑥∗) − 𝑘(𝑥∗,𝑋)𝑇
𝐾−1
𝑘(𝑥∗,𝑋) (9)
where 𝑘(𝑥∗,𝑋) is the covariance vector between the new input point and all training data points, and 𝑦 is the vector of
target values for the training data.
A.3 Acquisition Function
The acquisition function is a key component in BO, used to select the next evaluation point. It balances exploration and
exploitation, making trade-offs between exploring unknown regions and exploiting the known best regions.
Expected Improvement (EI): EI measures the expected amount of improvement over the current best point. The
formula is:
𝛼𝐸𝐼 (𝑥) = E[max(0, 𝑓 (𝑥) − 𝑓 (𝑥+
))] (10)
where 𝑓 (𝑥+) is the current best observed value.
Manuscript submitted to ACM

## Page 22

22
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
By modeling our custom FoM as a black-box function of the design parameters, we are able to use a GP to fit the
simulation and use BO to optimize our parameters without explicit knowledge.
B Complete LLM Conversations
B.1 Opamp Initial Design
Prompt: Initial Design Strategy
Task: Generate an operational amplifier. First analyze each constraint and provide a topology design following the given example.
Requirements: Using 28nm process with the following requirements:
DC gain > 80dB, GBW > 10MHz, phase margin > 60 degrees, CMRR > 80dB, PSRR > 80dB.
Follow the formats in the following example:
Task: Generate a comparator. First analyze each constraint and provide a topology design.
Requirements: Using 28nm process with the following requirements:
Propagation delay < 2.5ns, power dissipation < 5mW, Offset Voltage < 50mV, Hysteresis Voltage < 50mV.
Analyses:
1. The requirement of propagation delay is smaller than 2.5ns, which means that the comparator must respond quickly to input
changes, indicating high-speed circuitry and minimized capacitances. In 28nm process, this means that we can use the minimum
sizes transistors to minimizes the MOS parasitic capacitances, thus benefiting the transient response.
2. Power Dissipation<5mW means that efficient biasing and low-power design techniques are necessary.
3. The offset voltage and the hysteresis Voltage needs to be lower than 50mV, indicating that the comparator needs a pre-amplifier to
obtain high DC gain in order to reduce the comparator input-referred offset, low kick-back noise, and high speed, while keeping low
parasitic input capacitances.
Circuit structure:
Input stage:
-Differential pair input stage: Use a pair of MOSFETs (M1 and M2) as the differential pair input stage. Choose two PMOS with a
suitable aspect ratio (W/L) to provide high gain.
-Current mirror load: Use current mirrors (M3 – M5) as loads to achieve controllable current and bias.
Main comparator stage:
-Latch as comparator: Using a regenerative latch as a comparator. resistive comparing circuit for regenerative mode is used in series
with NMOS transistor to get a low power consumption. Besides that, PMOS transistor for pre charging circuit is absent during reset
mode and NMOS transistor for output pass transistor is nearly to Vdd/2 for the equalization of both voltage. Transistor M6 – M16 is
used to achieve the previous requirements.
Output buffer stage:
-Buffer: The output buffer stage (M17-M24) consists of a self-biased differential amplifier followed by an inverter which gives the
digital output.
Speed optimization:
-Fast transient response design: Use the minimized-size transistor to minimize the parasitic capacitance, but at the same time paying
attention to the sensitivity of small-size transistors to process variations.
Few-Shot ICL
Fig. 19. LLM prompt: from performance specification to initial design strategy
Manuscript submitted to ACM

## Page 23

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 23
LLM Response: Initial Design Strategy
Task: Generate an Operational Amplifier
Analyze each constraint and provide a topology design following the given example.
Requirements:
Using a 28nm process with the following specifications:
DC Gain > 80dB, GBW > 10MHz, Phase Margin > 60, degrees CMRR > 80dB, PSRR > 80dB
Analyses:
DC Gain > 80dB: This high gain requirement suggests the need for a robust first stage with a good intrinsic gain. Using a differential
pair with active loads can help achieve high initial gain.
GBW > 10MHz: A high gain-bandwidth product indicates the necessity of a carefully designed compensation strategy. This typically
involves placing a compensation capacitor between the output of the first stage and the input of the second stage to ensure stability
and adequate bandwidth.
Phase Margin > 60 Degrees: Maintaining a high phase margin requires careful feedback and compensation network design to ensure
stability, especially at high frequencies. This often involves selecting appropriate values for the compensation capacitors and possibly
incorporating additional poles or zeros strategically.
CMRR > 80dB and PSRR > 80dB: Both requirements point towards a need for excellent matching and symmetry in the differential
input stage and minimizing supply and common-mode noise influence. Techniques like using matched transistor pairs and common
centroid layout can help improve CMRR and PSRR.
Circuit Structure:
Input Stage: Differential Pair with Active Load:
Utilize a pair of NMOS transistors (M1 and M2) as the differential input pair. Choose an appropriate aspect ratio (W/L) to maximize the
transconductance and, consequently, the gain. Employ a PMOS current mirror (M3 and M4) as the active load to enhance the output
impedance and thus the gain of this stage.
Gain Stage (Second Stage): Common-Source Stage with Active Load:
The second stage can be a common-source amplifier (M5) driven by the output of the first stage. This stage further amplifies the
voltage and drives the output load. Another PMOS can be used as the active load (M6) to further improve the gain.
Compensation: Place a compensation capacitor (C1) and a resistor (R1) between the output of the first stage and the output of the
second stage to stabilize the two-stage amplifier configuration and set the dominant pole for desired phase margin and bandwidth.
Speed and Stability Optimization:
Minimize Parasitic Effects: use minimum length for transistors where possible to reduce parasitic capacitances, enhancing speed.
Enhanced Matching: Implement layout techniques such as common centroid and interdigitated fingers for critical matched transistors
to improve CMRR and PSRR by reducing mismatches.
This topology should meet the stringent requirements set for the operational amplifier while utilizing the advanced capabilities of the
28nm process technology.
Fig. 20. LLM response: from performance specification to initial design strategy
Manuscript submitted to ACM

## Page 24

24
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
Prompt: Design Strategy to Graph Query Triplets
Task: Extract the following circuit requirements and extract the relationships into triplets for knowledge graph query, each triplet should
be in the form of: <SUBJECT, RELATION, OBJECT>.
Circuit Structure:
Input Stage: Differential Pair with Active Load:
Utilize a pair of NMOS transistors (M1 and M2) as the differential input pair. Choose an appropriate aspect ratio (W/L) to maximize the
transconductance and, consequently, the gain. Employ a PMOS current mirror (M3 and M4) as the active load to enhance the output
impedance and thus the gain of this stage.
Gain Stage (Second Stage): Common-Source Stage with Active Load:
The second stage can be a common-source ampli
fi
er (M5) driven by the output of the
fi
rst stage. This stage further ampli
fi
es the voltage
and drives the output load. Another PMOS can be used as the active load (M6) to further improve the gain.
Compensation: Place a compensation capacitor (C1) and a resistor (R1) between the output of the
fi
rst stage and the output of the
second stage to stabilize the two-stage ampli
fi
er con
fi
guration and set the dominant pole for desired phase margin and bandwidth.
Speed and Stability Optimization:
Minimize Parasitic Effects: use minimum length for transistors where possible to reduce parasitic capacitances, enhancing speed.
Enhanced Matching: Implement layout techniques such as common centroid and interdigitated
fi
ngers for critical matched transistors to
improve CMRR and PSRR by reducing mismatches.
This topology should meet the stringent requirements set for the operational ampli
fi
er while utilizing the advanced capabilities of the
28nm process technology.
Here is an example：
CIRCUIT REQUIREMENTS :
Input stage:
-Differential pair input stage: Use a pair of MOSFETs (M1 and M2) as the differential pair input stage. Choose two PMOS with a
suitable aspect ratio (W/L) to provide high gain.
-Current mirror load: Use current mirrors (M3 – M5) as loads to achieve controllable current and bias.
Main comparator stage:
-Latch as comparator: Using a regenerative latch as a comparator. resistive comparing circuit for regenerative mode is used in series
with NMOS transistor to get a low power consumption. Besides that, PMOS transistor for pre charging circuit is absent during reset
mode and NMOS transistor for output pass transistor is nearly to Vdd/2 for the equalization of both voltage. Transistor M6 – M16 is
used to achieve the previous requirements.
Output buffer stage:
-Buffer: The output buffer stage (M17-M24) consists of a self-biased differential amplifier followed by an inverter which gives the
digital output.
Speed optimization:
-Fast transient response design: Use the minimized-size transistor to minimize the parasitic capacitance, but at the same time paying
attention to the sensitivity of small-size transistors to process variations.
Triplets :
<{}, input, differential pair>; <{}, load, current mirror>; <{}, name, regenerative latch>; <{}, power consumption, low>; <{}, name,
differential amplifier>; <{}, name, inverter>; <{}, type, buffer>
Few-Shot ICL
Fig. 21. LLM prompt: from initial design strategy to graph query triplets
Manuscript submitted to ACM

## Page 25

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 25
LLM Response: Extract Graph Query Triplets
Based on the detailed description of the circuit structure and requirements provided, here are the extracted triplets suitable for
constructing a knowledge graph:
Circuit Structure Triplets
1. <{}, uses, NMOS transistors (M1 and M2)>
2. <{}, input, differential input pair>
3. <{}, aspect ratio, maximize transconductance>
4. <{}, load, PMOS current mirror (M3 and M4)>
5. <{}, load, active load>
6. <{}, name, common-source amplifier (M5)>
7. <{}, driven by, output of first stage>
8. <{}, uses, PMOS (M6)>
9. <{}, load, active load>
10. <{}, includes, compensation capacitor (C1)>
11. <{}, includes, resistor (R1)>
12. <{}, function, stabilize amplifier configuration>
13. <{}, function, set dominant pole>
14. <{}, minimizes, parasitic effects>
15. <{}, uses, minimum length for transistors>
16. <{}, includes, enhanced matching techniques>
17. <{}, improve, CMRR and PSRR>
18. <{}, uses, common centroid and interdigitated fingers>
Fig. 22. LLM response: from initial design strategy to graph query triplets
Manuscript submitted to ACM

## Page 26

26
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
B.2 Opamp Regenerated Design
Prompt: Regenerate Design Strategy
Task: Generate a comparator. First analyze each constraint and provide a topology design following the given example..
Requirements：Using 28nm process with the following requirements:
propagation delay below 1ns, offset voltage below 100uV, sampling frequency 1GHz, power dissipation below 100uW.
Follow the formats in the following example:
Task: Generate an Operational Amplifier Analyze each constraint and provide a topology design following the given example.
Requirements:
Using a 28nm process with the following specifications:
DC Gain > 80dB, GBW > 10MHz, Phase Margin > 60 degrees, CMRR > 80dB, PSRR > 80dB
Analyses:
DC Gain > 80dB:
This high gain requirement suggests the need for a robust first stage with a good intrinsic gain. Using a differential pair with active loads can
help achieve high initial gain.
GBW > 10MHz:
A high gain-bandwidth product indicates the necessity of a carefully designed compensation strategy. This typically involves placing a
compensation capacitor between the output of the first stage and the input of the second stage to ensure stability and adequate bandwidth.
Phase Margin > 60 Degrees:
Maintaining a high phase margin requires careful feedback and compensation network design to ensure stability, especially at high frequencies.
This often involves selecting appropriate values for the compensation capacitors and possibly incorporating additional poles or zeros
strategically.
CMRR > 80dB and PSRR > 80dB:
Both requirements point towards a need for excellent matching and symmetry in the differential input stage and minimizing supply and
common-mode noise influence. Techniques like using matched transistor pairs and common centroid layout can help improve CMRR and PSRR.
Circuit Structure:
Input Stage: Differential Pair with Active Load:
Utilize a pair of NMOS transistors (M1 and M2) as the differential input pair. Choose an appropriate aspect ratio (W/L) to maximize the
transconductance and, consequently, the gain. Employ a PMOS current mirror (M3 and M4) as the active load to enhance the output impedance
and thus the gain of this stage.
Gain Stage (Second Stage): Common-Source Stage with Active Load:
The second stage can be a common-source amplifier (M5) driven by the output of the first stage. This stage further amplifies the voltage and
drives the output load. Another PMOS can be used as the active load (M6) to further improve the gain. Compensation: Place a compensation
capacitor (C1) and a resistor (R1) between the output of the first stage and the output of the second stage to stabilize the two-stage amplifier
configuration and set the dominant pole for desired phase margin and bandwidth.
Speed and Stability Optimization:
Minimize Parasitic Effects: use minimum length for transistors where possible to reduce parasitic capacitances, enhancing speed. Enhanced
Matching: Implement layout techniques such as common centroid and interdigitated fingers for critical matched transistors to improve CMRR
and PSRR by reducing mismatches. This topology should meet the stringent requirements set for the operational amplifier while utilizing the
advanced capabilities of the 28nm process technology.
Few-Shot ICL
Fig. 23. LLM prompt: from performance specification to revised design strategy
Manuscript submitted to ACM

## Page 27

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 27
LLM Response: Regenerated Design Strategy
Task: Generate a Comparator Analyze each constraint and provide a topology design following the given example.
Requirements: Using a 28nm process with the following specifications:
Propagation delay < 1ns, Offset voltage < 100uV, Sampling frequency = 1GHz, Power dissipation < 100uW
Analyses:
Propagation Delay < 1ns:
This extremely low delay requirement indicates the need for a fast response time, which can be achieved by a simple topology with minimal
stages and using devices with high transconductance. Minimizing the capacitive load on the comparator output is also crucial.
Offset Voltage < 100uV:
Achieving such a low offset voltage requires excellent matching of the input transistors. Techniques like common centroid layout and
dummy transistors can be used to enhance matching. Using a preamplifier stage with auto-zeroing techniques may also be beneficial to
mitigate offset.
Sampling Frequency 1GHz:
High-frequency operation demands the comparator to be optimized for speed. This involves reducing the total parasitic capacitance and
ensuring that the current drive capability of the output stage is sufficient to handle the load quickly.
Power Dissipation < 100uW:
Low power consumption can be maintained by optimizing the biasing currents and utilizing minimum size devices where possible,
balancing speed requirements.
Circuit Structure:
Input Stage: Differential Pair with Cross-Coupled Load:
Use a pair of NMOS transistors (M1 and M2) for the differential input. These should have minimum channel length to maximize speed and
carefully sized to ensure good matching. Implement a cross-coupled pair of PMOS transistors (M3 and M4) as the active load. This
configuration helps in rapidly pulling the output to the supply rails, enhancing the switching speed.
Latch Stage: Positive Feedback Latch:
A latch stage can be incorporated to quickly amplify the differential signal from the input stage and drive the output decisively to the
digital logic levels. This stage uses additional NMOS and PMOS transistors to form a regenerative loop, ensuring fast decision times.
Output Buffer:
An output buffer may be necessary to drive the external load, especially at high frequencies. This stage should be designed to minimize
power consumption while still being capable of fast switching.
Speed and Stability Optimization:
Minimize Parasitic Effects: Employ minimum length for all transistors and minimize routing capacitances to enhance the speed of the
comparator.
Enhanced Matching: Use layout techniques such as common centroid and interdigitated fingers for the input matched transistors to reduce
mismatches which contribute to offset voltage.
Power Optimization: Choose a bias current that balances speed and power requirements, avoiding excessive current that leads to high
power dissipation.
Fig. 24. LLM response: from performance specification to revised design strategy
Manuscript submitted to ACM

## Page 28

28
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
Prompt: Design Strategy to Graph Query Triplets
Task : Extract the following circuit requirements and extract the relationships into triplets for knowledge graph query, each triplet should
be in the form of: <SUBJECT, RELATION, OBJECT>.
Circuit Structure:
Input Stage: Cascode Differential Pair:
Use NMOS transistors (M1, M2) for the differential input, with a cascode con
fi
guration (M3, M4) to boost the gain and output resistance.
This setup helps in achieving higher DC gain crucial for meeting the 80dB requirement.
Second Stage: Gain Boosting Stage:
A second ampli
fi
cation stage might involve an additional common-source stage (M5) with an active load (M6) to further increase the
gain. This stage drives the output while maintaining the linearity and dynamic range.
Compensation: Miller Compensation:
Implement a Miller compensation capacitor (C1) between the output of the
fi
rst stage and the input of the second stage. This
con guration sets the dominant pole and manages the frequency response to ensure the GBW and phase margin targets are met.
Layout and Optimization:
Minimize Parasitic Capacitances: Employ minimum length for transistors in high-frequency paths to reduce parasitic capacitances and
enhance response speed.
Improve Matching and Symmetry:
Implement advanced layout techniques such as common centroid and shielded routing for critical matched components to optimize
CMRR and PSRR.
This design approach should meet the speci
fi
ed requirements for the operational ampli
fi
er using the capabilities of the 28nm process
technology, focusing on high performance in DC gain, bandwidth, and noise rejection capabilities.
CIRCUIT REQUIREMENTS :
Input stage:
-Differential pair input stage: Use a pair of MOSFETs (M1 and M2) as the differential pair input stage. Choose two PMOS with a
suitable aspect ratio (W/L) to provide high gain.
-Current mirror load: Use current mirrors (M3 – M5) as loads to achieve controllable current and bias.
Main comparator stage:
-Latch as comparator: Using a regenerative latch as a comparator. resistive comparing circuit for regenerative mode is used in series
with NMOS transistor to get a low power consumption. Besides that, PMOS transistor for pre charging circuit is absent during reset
mode and NMOS transistor for output pass transistor is nearly to Vdd/2 for the equalization of both voltage. Transistor M6 – M16 is
used to achieve the previous requirements.
Output buffer stage:
-Buffer: The output buffer stage (M17-M24) consists of a self-biased differential amplifier followed by an inverter which gives the
digital output.
Speed optimization:
-Fast transient response design: Use the minimized-size transistor to minimize the parasitic capacitance, but at the same time paying
attention to the sensitivity of small-size transistors to process variations.
Triplets :
<{}, input, differential pair>; <{}, load, current mirror>; <{}, name, regenerative latch>; <{}, power consumption, low>; <{}, name,
differential amplifier>; <{}, name, inverter>; <{}, type, buffer>
Few-Shot ICL
Fig. 25. LLM prompt: from revised design strategy to graph query triplets
Manuscript submitted to ACM

## Page 29

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 29
LLM Response: Extract Graph Query Triplets
To extract relationships and form them into triplets for knowledge graph representation from the detailed description of the circuit
structure you provided, we'll identify the key components, their relationships, and their properties or functionalities. Here are the triplets
formed based on the given structure:
Circuit Structure Triplets
1. <{}, type, NMOS transistor>
2. <{}, type, NMOS transistor>
3. <{}, input, differential input>
4. <{}, type, NMOS transistor>
5. <{}, type, NMOS transistor>
6. <{}, load, cascode>
7. <{}, characteristic, higher DC gain>
8. <{}, requirement, 80dB gain>
9. <{}, configuration, common-source stage>
10.<{}, function, active load>
11.<{}, role, amplification>
12.<{}, function, gain increase>
13.<{}, characteristic, maintains linearity>
14.<{}, characteristic, maintains dynamic range>
15.<{}, type, Miller compensation capacitor>
16.<{}, location, between output of first stage and input of second stage>
17.<{}, function, sets dominant pole>
18.<{}, function, manages frequency response>
19.<{}, requirement, meets GBW>
20.<{}, requirement, meets phase margin>
21.<{}, characteristic, minimum length>
22.<{}, role, reduce parasitic capacitances>
23.<{}, role, enhance response speed>
These triplets provide a structured way to represent relationships and properties within the circuit design suitable for input into a
knowledge graph or similar data representation systems.
Fig. 26. LLM response: from revised design strategy to graph query triplets
Manuscript submitted to ACM

## Page 30

30
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
B.3 Comparator Design
Prompt: Design Strategy
Task: Generate a comparator. First analyze each constraint and provide a topology design following the given example..
Requirements：Using 28nm process with the following requirements:
propagation delay below 1ns, offset voltage below 100uV, sampling frequency 1GHz, power dissipation below 100uW.
Follow the formats in the following example:
Task: Generate an Operational Amplifier Analyze each constraint and provide a topology design following the given example.
Requirements:
Using a 28nm process with the following specifications:
DC Gain > 80dB, GBW > 10MHz, Phase Margin > 60, degrees CMRR > 80dB, PSRR > 80dB
Analyses:
DC Gain > 80dB:
This high gain requirement suggests the need for a robust first stage with a good intrinsic gain. Using a differential pair with active
loads can help achieve high initial gain.
GBW > 10MHz:
A high gain-bandwidth product indicates the necessity of a carefully designed compensation strategy. This typically involves placing a
compensation capacitor between the output of the first stage and the input of the second stage to ensure stability and adequate
bandwidth.
Phase Margin > 60 Degrees:
Maintaining a high phase margin requires careful feedback and compensation network design to ensure stability, especially at high
frequencies. This often involves selecting appropriate values for the compensation capacitors and possibly incorporating additional
poles or zeros strategically.
CMRR > 80dB and PSRR > 80dB:
Both requirements point towards a need for excellent matching and symmetry in the differential input stage and minimizing supply
and common-mode noise influence. Techniques like using matched transistor pairs and common centroid layout can help improve
CMRR and PSRR.
Circuit Structure:
Input Stage: Differential Pair with Active Load:
Utilize a pair of NMOS transistors (M1 and M2) as the differential input pair. Choose an appropriate aspect ratio (W/L) to maximize
the transconductance and, consequently, the gain. Employ a PMOS current mirror (M3 and M4) as the active load to enhance the
output impedance and thus the gain of this stage.
Gain Stage (Second Stage): Common-Source Stage with Active Load:
The second stage can be a common-source amplifier (M5) driven by the output of the first stage. This stage further amplifies the
voltage and drives the output load. Another PMOS can be used as the active load (M6) to further improve the gain. Compensation:
Place a compensation capacitor (C1) and a resistor (R1) between the output of the first stage and the output of the second stage to
stabilize the two-stage amplifier configuration and set the dominant pole for desired phase margin and bandwidth.
Speed and Stability Optimization:
Minimize Parasitic Effects: use minimum length for transistors where possible to reduce parasitic capacitances, enhancing speed.
Enhanced Matching: Implement layout techniques such as common centroid and interdigitated fingers for critical matched transistors
to improve CMRR and PSRR by reducing mismatches. This topology should meet the stringent requirements set for the operational
amplifier while utilizing the advanced capabilities of the 28nm process technology.
Few-Shot ICL
Fig. 27. LLM prompt: from performance specification to design strategy
Manuscript submitted to ACM

## Page 31

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 31
LLM Response: Design Strategy
Task: Generate a Comparator Analyze each constraint and provide a topology design following the given example.
Requirements: Using a 28nm process with the following specifications:
Propagation delay < 1ns, Offset voltage < 100uV, Sampling frequency = 1GHz, Power dissipation < 100uW
Analyses:
Propagation Delay < 1ns:
This extremely low delay requirement indicates the need for a fast response time, which can be achieved by a simple topology with
minimal stages and using devices with high transconductance. Minimizing the capacitive load on the comparator output is also crucial.
Offset Voltage < 100uV:
Achieving such a low offset voltage requires excellent matching of the input transistors. Techniques like common centroid layout and
dummy transistors can be used to enhance matching. Using a preamplifier stage with auto-zeroing techniques may also be beneficial to
mitigate offset.
Sampling Frequency 1GHz:
High-frequency operation demands the comparator to be optimized for speed. This involves reducing the total parasitic capacitance
and ensuring that the current drive capability of the output stage is sufficient to handle the load quickly.
Power Dissipation < 100uW:
Low power consumption can be maintained by optimizing the biasing currents and utilizing minimum size devices where possible,
balancing speed requirements.
Circuit Structure:
Input Stage: Differential Pair with Cross-Coupled Load:
Use a pair of NMOS transistors (M1 and M2) for the differential input. These should have minimum channel length to maximize speed
and carefully sized to ensure good matching. Implement a cross-coupled pair of PMOS transistors (M3 and M4) as the active load. This
configuration helps in rapidly pulling the output to the supply rails, enhancing the switching speed.
Latch Stage: Positive Feedback Latch:
A latch stage can be incorporated to quickly amplify the differential signal from the input stage and drive the output decisively to the
digital logic levels. This stage uses additional NMOS and PMOS transistors to form a regenerative loop, ensuring fast decision times.
Output Buffer:
An output buffer may be necessary to drive the external load, especially at high frequencies. This stage should be designed to minimize
power consumption while still being capable of fast switching.
Speed and Stability Optimization:
Minimize Parasitic Effects: Employ minimum length for all transistors and minimize routing capacitances to enhance the speed of the
comparator.
Enhanced Matching: Use layout techniques such as common centroid and interdigitated fingers for the input matched transistors to
reduce mismatches which contribute to offset voltage.
Power Optimization: Choose a bias current that balances speed and power requirements, avoiding excessive current that leads to high
power dissipation.
Fig. 28. LLM response: from performance specification to design strategy
Manuscript submitted to ACM

## Page 32

32
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
Prompt: Design Strategy to Graph Query Triplets
Task : Extract the following circuit requirements and extract the relationships into triplets for knowledge graph query, each triplet should
be in the form of: <SUBJECT, RELATION, OBJECT>.
Circuit Structure:
Input Stage: Differential Pair with Cross-Coupled Load:
Use a pair of NMOS transistors (M1 and M2) for the differential input. These should have minimum channel length to maximize speed
and carefully sized to ensure good matching.
Implement a cross-coupled pair of PMOS transistors (M3 and M4) as the active load. This configuration helps in rapidly pulling the output
to the supply rails, enhancing the switching speed.
Latch Stage: Positive Feedback Latch:
A latch stage can be incorporated to quickly amplify the differential signal from the input stage and drive the output decisively to the
digital logic levels. This stage uses additional NMOS and PMOS transistors to form a regenerative loop, ensuring fast decision times.
Output Buffer:
An output buffer may be necessary to drive the external load, especially at high frequencies. This stage should be designed to minimize
power consumption while still being capable of fast switching.
Speed and Stability Optimization:
Minimize Parasitic Effects: Employ minimum length for all transistors and minimize routing capacitances to enhance the speed of the
comparator.
Enhanced Matching: Use layout techniques such as common centroid and interdigitated fingers for the input matched transistors to
reduce mismatches which contribute to offset voltage.
Power Optimization:
Choose a bias current that balances speed and power requirements, avoiding excessive current that leads to high power dissipation.
Here is an example：
CIRCUIT REQUIREMENTS :
Input stage:
-Differential pair input stage: Use a pair of MOSFETs (M1 and M2) as the differential pair input stage. Choose two PMOS with a
suitable aspect ratio (W/L) to provide high gain.
-Current mirror load: Use current mirrors (M3 – M5) as loads to achieve controllable current and bias.
Main comparator stage:
-Latch as comparator: Using a regenerative latch as a comparator. resistive comparing circuit for regenerative mode is used in series
with NMOS transistor to get a low power consumption. Besides that, PMOS transistor for pre charging circuit is absent during reset
mode and NMOS transistor for output pass transistor is nearly to Vdd/2 for the equalization of both voltage. Transistor M6 – M16 is
used to achieve the previous requirements.
Output buffer stage:
-Buffer: The output buffer stage (M17-M24) consists of a self-biased differential amplifier followed by an inverter which gives the
digital output.
Speed optimization:
-Fast transient response design: Use the minimized-size transistor to minimize the parasitic capacitance, but at the same time paying
attention to the sensitivity of small-size transistors to process variations.
Triplets :
<circuit :{} , string : name, string : 5-transistor opamp>, <circuit :{} , string : input, string : differential>, <circuit :{} , string : output, string
: single>, <circuit :{} , string : load, string : current mirror>, <circuit :{} , string : performance, string : low CMRR >, <circuit :{} , string :
performance, string : low PSRR>
Few-Shot ICL
Fig. 29. LLM prompt: from design strategy to graph query triplets
Manuscript submitted to ACM

## Page 33

AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-Design
Using Knowledge Graph RAG 33
LLM Response: Extract Graph Query Triplets
Here are the extracted triplets representing the relationships and entities described in the given circuit requirements:
Circuit Structure Triplets
1. <{}, type, NMOS transistors>
2. <{}, purpose, differential input>
3. <{}, property, minimum channel length>
4. <{}, property, carefully sized>
5. <{}, type, PMOS transistors>
6. <{}, configuration, cross-coupled>
7. <{}, purpose, active load>
8. <{}, function, enhance switching speed>
9. <{}, type, Positive Feedback Latch>
10. <{}, function, amplify differential signal>
11. <{}, component, additional NMOS and PMOS transistors>
12. <{}, feature, regenerative loop>
13. <{}, purpose, drive external load>
14. <{}, condition, high frequencies>
15. <{}, goal, minimize power consumption>
Speed and Stability Optimization Triplets:
1. <{}, property, minimum length>
2. <{}, goal, minimize routing capacitances>
3. <{}, outcome, enhance speed>
4. <{}, technique, common centroid>
5. <{}, technique, interdigitated fingers>
6. <{}, outcome, reduce mismatches>
7. <{}, speed and power requirements>
8. <{}, consequence, high power dissipation>
General Configuration and Goal Triplets:
1. <{}, technology, 28nm>
2. <{}, characteristic, stringent requirements compliance>
3. <{}, strategy, advanced techniques employment>
These triplets structure the information in a way that could be input into a knowledge graph for querying or further analysis related to
the circuit design and its components.
Fig. 30. LLM response: from design strategy to graph query triplets
Manuscript submitted to ACM

## Page 34

34
Yichen Shi, Zhuofu Tao, Yuhao Gao, Tianjia Zhou, Cheng Chang, Yaxing Wang, Bingyu Chen, Genhao Zhang, Alvin
Liu, Zhiping Yu, Ting-Jung Lin, and Lei He
LLM Prompt: Topology Selection
Task: Our database returns 2 matching topologies:
- Strong-arm latch
- Double tail latch
Select a single topology between the above.
LLM Response: Topology Selection
Answer: Given the requirements and the matching topologies available from your database, I would recommend selecting the
Strong-arm latch topology for your comparator design.
Here's why:
Speed and Power Efficiency:
The Strong-arm latch is known for its excellent speed characteristics and low power consumption. This fits well with your requirement of
having a propagation delay below 1ns and power dissipation below 100uW.
Offset Voltage Management:
The Strong-arm topology, due to its symmetric design, can be optimized for low offset voltage through careful layout and transistor
matching, addressing the requirement of keeping offset voltage below 100uV.
High Sampling Frequency Compatibility:
This topology is effective in handling high sampling frequencies, in your case, 1GHz, due to its inherent quick reset and set-up
characteristics, making it suitable for high-speed applications.
Simplicity and Robustness:
Strong-arm latches are relatively simple and robust, providing a good balance between performance and design complexity, which can
be advantageous in a 28nm process where layout density and variability might be concerns.
This topology should serve well in meeting the stringent performance criteria set by your specifications while utilizing the advanced
capabilities of the 28nm process technology.
Fig. 31. LLM prompt and response: tie breaker between valid topologies, continuing on the context of Fig. 28
Manuscript submitted to ACM

