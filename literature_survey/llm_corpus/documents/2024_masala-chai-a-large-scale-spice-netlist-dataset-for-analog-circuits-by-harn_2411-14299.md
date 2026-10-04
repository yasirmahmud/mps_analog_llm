---
title: "Masala-CHAI: A Large-Scale SPICE Netlist Dataset for Analog Circuits by Harnessing AI"
authors: "Bhandari, Jitendra; Bhat, Vineet; He, Yuheng; Rahmani, Hamed; Garg, Siddharth; Karri, Ramesh"
date: "2024/11/21"
category: "data_knowledge"
source_url: "https://arxiv.org/abs/2411.14299"
pdf: "papers/2024_masala-chai-a-large-scale-spice-netlist-dataset-for-analog-circuits-by-harn_2411-14299.pdf"
sha256: "f94aa113e67e2784ec7a4bf973b080064889b30caeb640ad4f61e85b3a8fcdc3"
pages_extracted: 9
---

# Masala-CHAI: A Large-Scale SPICE Netlist Dataset for Analog Circuits by Harnessing AI

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

Masala-CHAI: A Large-Scale SPICE Netlist Dataset
for Analog Circuits by Harnessing AI
Jitendra Bhandari∗, Vineet Bhat∗, Yuheng He†, Hamed Rahmani∗, Siddharth Garg∗ and Ramesh Karri∗
∗New York University Tandon School of Engineering †Cornell University
Abstract—Masala-CHAI is a fully automated framework leverag-
ing large language models (LLMs) to generate Simulation Programs
with Integrated Circuit Emphasis (SPICE) netlists. It addresses a
long-standing challenge in circuit design automation: automating
netlist generation for analog circuits. Automating this workflow
could accelerate the creation of fine-tuned LLMs for analog circuit
design and verification. In this work, we identify key challenges in
automated netlist generation and evaluate multimodal capabilities
of state-of-the-art LLMs, particularly GPT-4, in addressing them.
We propose a three-step workflow to overcome existing limitations:
labeling analog circuits, prompt tuning, and netlist verification.
This approach enables end-to-end SPICE netlist generation from
circuit schematic images, tackling the persistent challenge of
accurate netlist generation. We utilize Masala-CHAI to collect
a corpus of 7,500 schematics that span varying complexities in 10
textbooks and benchmark various open source and proprietary
LLMs. Models fine-tuned on Masala-CHAI when used in LLM-
agentic frameworks such as AnalogCoder achieve a notable 46%
improvement in Pass@1 scores. We open-source our dataset and
code for community-driven development.
Index Terms—Analog Design, Automation, LLM
I. INTRODUCTION
Large Language Models (LLMs) have received significant
attention due to their wide-ranging applications, from text
summarization to code generation, and have a growing impact
across various fields. For hardware design, LLMs have primarily
demonstrated potential in the digital domain. This includes tasks
such as Verilog code generation [1]–[4], assertion generation [5],
[6], bug fixing [7], [8], and electronic design automation (EDA)
tool scripting [9], [10]. The success of these domain-tailored
LLMs relies on access to large and high-quality datasets.
For example, the Verigen code generation model [4] was
trained on a dataset of 75,000 Verilog files sourced from
GitHub, with additional data extracted from Verilog textbooks
to further enhance its performance in Verilog-related tasks.
Beyond hardware design, textbooks have also been valuable
in other domains, such as systems biology [11] and protein
interaction studies [12], demonstrating their broader utility in
training domain-specific models.
Building on the success of LLMs in the digital domain, it
is natural to extend their application in the analog domain,
particularly in the automated generation of analog circuits from
natural language specifications. Analog circuits are described
in SPICE and its various variants, which serve as the industry-
standard textual representation for analog circuit simulation.
SPICE provides a low-level description that defines the intercon-
nections between analog components like resistors, capacitors,
inductors, and transistors. The availability of open-source
datasets for SPICE remains significantly limited compared to
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
ID
0
20
40
60
80
100
Pass@1
No Finetune
Finetune with 1 textbook
Finetune with 3 textbook
Finetune with 5 textbook
Finetune with 7 textbook
Finetune with 10 textbook
Fig. 1: Pass@1 performance of GPT-4o fine-tuned with Masala
CHAI datasets extracted from between 1-10 textbooks, compared
to the no fine-tuning baseline (task IDs from Table II). Our
largest dataset of 7500 captioned SPICE netlists extracted from
10 textbooks provides more than 40% Pass@1 accuracy even
on challenging tasks like telescoping cascode amplifier and
bandgap reference circuit generation.
Verilog, which poses a major challenge for automating the
learning of frameworks for the generation of net lists [13]–
[17]. However, analog circuit textbooks and research papers are
plentiful and contain a trove of analog circuit diagrams, but
these are usually in image (or figure) format. These images
must then be manually converted to SPICE netlists, which is
painstaking and time-consuming.
Masala-CHAI1
is an automated framework for SPICE netlist
generation from analog circuit schematics. We incorporate
various techniques, including (i) Schematic extractor from
documents, (ii) fine-tuned circuit component detectors to extract
passive and active components, (iii) deep hough transform
priors to identify nets, (iv) extensive prompt tuning for LLMs
to fix common errors in netlist extraction, and (v) post-
extraction verification. We curated the largest open-source
analog circuit dataset, comprising 7,500 examples using Masala-
CHAI, and fine-tuned various LLMs, including CodeLlama,
DeepSeek-34B, and GPT-(3.5,4). As shown in Figure 1,
larger datasets significantly enhance performance, improving
pass@k results across different numbers of textbooks used for
fine-tuning.
Our contributions are as follows:
1Masala is the Hindi word for “SPICE”
arXiv:2411.14299v5
[cs.AR]
23
Mar
2025

## Page 2

1) An empirical case study to understand the limitations of
state-of-the-art multi-modal LLMs like GPT-4o (shown in
AMSNet [13]) in SPICE netlist extraction from schematics.
2) Build Masala-CHAI, the first fully automated framework
that combines LLMs with custom-trained deep network
models for large-scale SPICE netlist extraction from circuit
schematics found in textbooks and research papers.
3) Leverage Masala-CHAI to compile an open-sourced dataset
of ∼7,500 SPICE netlists with metadata (figure captions,
components, and nodes) from 10 textbooks. To the best of
our knowledge, this is the largest curated and labeled SPICE
dataset available currently.
4) Finetune and benchmark four state-of-art LLMs
(CodeLLama-70B, DeepSeek-34B, GPT-3.5, GPT-4o)
on the Masala-CHAI dataset, showcasing its potential for
automated SPICE netlist generation from English prompts.
5) Enhance the AnalogCoder [14] agentic framework by incor-
porating GPT-4o fine-tuned on Masala-CHAI, achieving a
46% absolute improvement in Pass@1 scores.
II. RELATED WORK
LLMs have gained significant attention in chip design [18].
Tremendous progress has been made in improving Verilog code
generation [1]–[4]. These advances demonstrate how LLMs can
streamline and improve digital hardware design workflows. In
addition to code generation, prompt engineering has proven
effective in chip design [19]–[21]. Using LLMs, researchers
have efficiently conceptualized and designed complex digital
hardware architectures, facilitating faster and more accurate
chip development. Beyond code generation, LLMs have found
applications in assistive chatbots, script generation, and bug
analysis [10]. [9] explores LLMs in planning and executing
tasks in the Electronic Design Automation (EDA) flow. The
use of LLMs in generating assertions and testbenches for
verifying the correctness of Integrated Circuit (IC) designs
has also seen notable improvements [5]–[8]. [16], [17] have
explored schematic-to-netlist generation using ML-based ap-
proaches. However, these solutions are not reproducible due to
the unavailability of any open-source implementation. Recent
advances extended LLM applications into analog design. Closed-
loop Python code generation for analog circuits by LLM
agents has shown promising results [14]. LLMs are able to
generate circuit topologies from specifications [22]. A dataset
for exploring SPICE netlist generation was recently released
in [13], summarized in Table I.
TABLE I: Comparison with AMSNet [13]. Here, ‘M’ denotes
Manual, ‘P’ denotes Partial Automation (with some manual
intervention), and ‘A’ denotes Fully Automated.
Schematic Component Nets Textbooks Dataset
AMSNet [13] M A P 1 734
Masala-CHAI A A A 10 7500
III. KEY CHALLENGES IN SPICE NETLIST EXTRACTION
Despite advances in multi-modal LLMs, models like GPT-4o
do not automatically extract accurate SPICE netlists from circuit
schematics [13]. Our work begins with an in-depth analysis of
GPT-4o’s failure modes. Building on these insights, we propose
our solutions and introduce a fully automated SPICE netlist
extraction framework without manual annotations.
A. Can GPT-4o detect electrical components accurately?
c
Prompt:
You are a professional analog design. Given a schematic, you have to identify all the
components.
M1: NMOS Transistor
M2: NMOS Transistor
M3: NMOS Transistor
M1 & M2: NMOS Transistor
M3 & M4: NMOS Transistor
CL: Capapcitor
Iss: Current Source
M5: Not Detected
I1: Not Detected
M1, M2, M5: NMOS Transistor
M3 & M4: PMOS Transistor
R1: Resistor
Fig. 2: GPT-4o responses when asked to list all components in
three sample circuit schematics.
When generating a SPICE netlist, accurately identifying all
circuit components is essential for designers and challenging
for language models. We curated a set of analog schematics
featuring resistors, capacitors, inductors, MOSFETs, and various
sources from a popular analog textbook [23] and evaluated
GPT-4o’s ability to recognize electrical components in cir-
cuit schematics. As shown in Figure 2, GPT-4o frequently
misidentifies components, often confusing NMOS and PMOS
transistors (left and middle schematics) or omitting key elements
like PMOS transistors and current sources. However, in some
cases (final schematic), it accurately identifies all components,
demonstrating potential despite its inconsistencies.
Solution 1: Customized Object Detection Network for Analog
Circuit Components- Recognizing the limitations of GPT-4o in
this task, we propose a dedicated solution using a deep CNN-
based object detection model, using the strong performance of
models such as YOLO [24]. By training a CNN model designed
explicitly for schematic components, we significantly improve
the accuracy in detecting and bounding all circuit elements, a
method detailed in more detail in Section IV.
Solution 2: Prompt Tuning - We developed a targeted prompt
enhancement strategy to refine LLM component differentia-
tion, especially between NMOS and PMOS transistors. By
emphasizing structural differences in prompts (see Figure 3),
we achieved notable improvements in accuracy. Our empirical
findings support this approach in Section IV.
You are an expert analog designer. You will be provided with a
schematic, your task is to follow the below instructions carefully:
1) To identify the NMOS and PMOS MOSFET, follow the
instructions carefully. For NMOS, the arrow on the source
terminal points outwards from the transistor. For PMOS, the
arrow on the source terminal points inward towards the transistor.
2) List all the components correctly.
Fig. 3: Prompt Tuning guides the GPT to differentiate between
NMOS and PMOS.

## Page 3

B. Can GPT-4o accurately connect circuit components?
* Transistors
M1 N001 Vin1 0 0 NMOS L=180n W=360n
M2 N002 Vin2 0 0 NMOS L=180n W=360n
M3 Vout Vb1 VDD VDD PMOS L=180n W=360n
M4 Vout Vb1 VDD VDD PMOS L=180n W=360n
M5 Vout Vb1 VDD VDD PMOS L=180n W=360n
M6 Vout Vb1 VDD VDD PMOS L=180n W=360n
M7 N002 Vb1 0 0 NMOS L=180n W=360n
* Transistors
M1 Vout Vin1 0 0 NMOS L=180n W=1u
M2 Vout Vin2 0 0 NMOS L=180n W=1u
M3 Vout Vb2 VDD VDD PMOS L=180n W=1u
M4 Vout Vb2 VDD VDD PMOS L=180n W=1u
M5 0 Vb1 0 0 NMOS L=180n W=1u
Prompt:
You are a professional analog design. Given a schematic with components
annotated, write a spice netlist
Fig. 4: SPICE netlist generated by the GPT-4o. For brevity, we
only show the part of the netlist that describes transistors.
After detecting all circuit components, the next step is
to ensure their correct connections, so the generated netlist
accurately represents the schematic. While GPT-4o successfully
maps 2-terminal devices, our study uncovered several critical
failure modes that compromise the accuracy of netlist generation.
These failure modes are illustrated in Figure 4. Firstly, GPT-
4o incorrectly assumes that intersecting nets are connected,
even when no connection exists. This error arises partly from
inconsistencies in the definition of connectivity across different
schematic notations. Secondly, GPT-4o often misidentifies
MOSFET terminals, confusing the drain, gate, and source.
Despite the drain and source’s electrical equivalence, this
misidentification affects the accuracy of the final netlist. Finally,
differential input and output voltage pairs pose significant
challenges for GPT-4o. For example, in the left schematic of
Figure 4, while components were correctly identified, the source
connections for (M1 and M2) were incorrect, and the gate
voltage for (M3 and M4) was wrongly assumed to be ‘Vb1’.
Similar errors appeared in the right schematic, where differential
output was mishandled, and M5’s drain was incorrectly mapped.
Solution: Automatic Net Annotation using Deep Hough Trans-
form - Leveraging our knowledge of component locations, we
developed a method to identify and annotating nets using a Deep
Hough transform line priors. Incorporating these annotations into
the prompt (see Section IV), led to improvement in GPT-4o’s
performance in translating schematics to SPICE netlists. This
systematic annotation reduced ambiguities and enhanced the
LLM’s ability to interpret and connect components, particularly
in complex schematics correctly.
IV. METHODOLOGY
This section details the methodology used, addressing the
challenges outlined in Section III. Accurate generation of SPICE
netlists remains a bottleneck for LLMs due to difficulties in
correctly identifying circuit components and establishing net
connections. Our dataset creation framework, shown in Figure 5,
starts by paginating a textbook into individual PDFs (➊), which
are then passed through a YOLOv8 [24], a state-of-the-art object
localization and classification model, to extract schematics from
the page. Along with extracted schematics, we also extract and
summarize text describing the schematic, for example, from the
schematic caption or text on the page referring to the schematic.
These are used as additional context for fine-tuning LLMs,
Section V-B, shown in Figure 5 (➐ - ➒). The extracted schematic
is still in image format and needs to be reliably converted to a
SPICE netlist. To this end, Masala-CHAI follows a three-step
process: (i) labeling analog circuits, (ii) prompt tuning, and (iii)
SPICE netlist verification, starting from schematic extraction
( Figure 5, ➌ - ➏). These steps are described next.
A. Labeling Analog Circuit
Figure 5 (➌) simplifies the LLM’s task of generating SPICE
netlists by detecting components and facilitating net annotation.
As discussed in Section III, SPICE netlist generation is not
straightforward, requiring the designer to feed information
alongside the schematics. However, this manual process hinders
generating large-scale datasets. Thus, we automate the labeling
of all parts of the schematics.
1) Detection of Circuit Components: We train a custom
YoloV8 model [24] specifically for detecting and classifying
circuit components with high precision, similar to prior
work [16], [17]. YoloV8’s Darknet53 backbone [25], enhanced
with a “Cross Stage Partial” bottleneck [26], integrates
high-level semantic information with low-level spatial details,
enabling accurate detection even for small components—a
requirement for component detection in complex circuit
schematics. We fine-tune YOLOv8 on an open-source
dataset [27] consisting of 4,300 circuit diagrams with
annotated bounding boxes across 12 component classes: AC
Source, BJT, Battery, Capacitor, DC Source,
Diode, Ground, Inductor, MOSFET, Resistor,
Current Source, and Voltage Source. Images
were resized to 640x640 and trained for 1000 epochs with a
learning rate of 0.01. The resulting fine-tuned YoloV8 model
localizes each component in the schematic by outputting its
center coordinates, a bounding box, and classification labels, as
illustrated in Figure 6 (➋).
2) Net Detection: We performed net detection in circuit
schematics by leveraging pre-trained deep hough transform
line priors [28], a state-of-the-art approach for line detection
in images. Unlike conventional methods, this model operates
in the Hough domain, parametrizing line segments in polar
coordinates to achieve precise line segmentation. We remove
components from the schematic to address challenges posed
by line segments within components such as capacitors and
MOSFETs, leaving only the lines representing nets, as illustrated
in Figure 6 (➌). This ensures a clean input for subsequent net
analysis and eliminates potential sources of ambiguity.
Following net detection, we propose a simple yet effective
heuristic for clustering line segments into nets. Specifically, we
group all line segments into a single cluster if their endpoints
fall within a radius of 40 pixels, as demonstrated in Figure 6
(➍). To ensure reliability, we tune the radius parameter and
manually verify the consistency of the clustered nets.

## Page 4

SPICE Verification
Write a SPICE netlist for
the provided Schematic
Caption/
Description
Please provide description for
the schematic present in the
page and also KPI.
Labeling Analog
Circuit
(1) Document
LLM
(3) LLM
Caption: "A simple CMOS
amplifier configuration M1 is in a
common-source configuration
while M2 operates as a current
source or active load"
SPICE Netlist:
M2 2 1 0 0 NMOS W=1u L=1u
M1 VDD 4 2 2 NMOS W=1u L=1u
Final Dataset
1
3 4 5 6
7
8
9
Circuit Extractor
(Yolo)
2
Fig. 5: Dataset preparation flow starting from a PDF document consisting of Schematic and related text. Masala-CHAI extracts
the relevant information and generates the SPICE netlist for the schematic.
4
6 7
1 2
3
5
8
9
9
Component Detection
Schematic Component Removal Clustered Nets Annotated Schematic
1 2 3 4 5
Fig. 6: Schematic −→ Annotated Schematic. Colors denote different component types. Similarly, different nets are clustered.
B. Prompt Tuning
Prompts are crucial in guiding LLMs to generate precise, high-
quality outputs. Crafting effective prompts involves meticulous
design of their language, structure, and contextual framing,
enabling models to respond accurately and meaningfully to
user inputs. Through prompt refinement, designers can precisely
control model behavior, ensuring the generation of coherent
and domain-specific responses. In this study, we systematically
explore diverse prompt designs tailored to enhance under-
standing of schematics. Our approach demonstrates significant
effectiveness, as illustrated in Figure 3 and Figure 7. Specifically,
prompt in Figure 7 is used at the stage Figure 5 (➍), facilitating
insightful queries to the GPT model.
You are an expert analog designer. You will be provided with a
schematic, your task is to follow the below instructions carefully:
1) List all components which you can observe from the figure.
2) MOSFETs are 3 terminal devices with (drain, gate, source).
3) For each component, look at the net number in red.
4) To identify source terminal of a MOSFET, choose net highlighted
in red which is nearest to the arrow of the MOSFET.
5) Write a SPICE netlist.
Fig. 7: Prompt GPT to generate SPICE netlist from Schematic.
C. SPICE netlist verification
Our case study revealed that some generated netlists (Figure 5
(➎)) still exhibit issues such as floating nets, posing risks large-
scale datasets where manual inspection is infeasible. To mitigate
this, we introduced a final verification step (Figure 5 (➏)) that
calls an LLM on the extracted SPICE netlist to automatically
identify and correct common errors, for instance, removing
floating nets as mentioned above. These checks are provided
in our open-source code base, and can be further extended to
address other sources of error, as required.
V. RESULTS
A. Dataset Creation
We curated a comprehensive dataset by automating schematic
extraction and SPICE netlist generation from 10 textbooks [29]–
[38]. These textbooks were selected for their high-quality
schematic images, clear context, and detailed explanations.
After collecting ∼7,500 schematic images, we applied the
Masala-CHAI framework (Figure 5) to generate SPICE netlists.
Alongside, we collected captions and descriptions as detailed
in Section IV. Figure 8 demonstrates how Masala-CHAI
translates a schematic into SPICE. Since our goal is to generate
structurally correct netlists, each component is marked with
default parameters.
Our dataset is characterized to highlight its diversity and
complexity in Figure 9: (a) shows the variation in the number
of components within schematics, where a higher count indicates
greater circuit complexity; (b) presents the distribution of nodes,
representing component connectivity. Accurately identifying
and mapping MOSFET terminals to the correct nets is a

## Page 5

Document
From Masala-CHAI
SPICE Netlist
M1 3 1 7 7 NMOS L=1u W=10u
M2 4 2 7 7 NMOS L=1u W=10u
M3 3 6 8 8 PMOS L=1u W=10u
M4 4 6 8 8 PMOS L=1u W=10u
M5 7 5 0 0 NMOS L=1u W=10u
VDD 8 0 DC 1.8V
Vb1 5 0 DC 1.8V
Vb2 6 0 DC 1.8V
CL 6 7 10pF
R1 3 4 1k
Vin1 1 0 DC 1V
Vin2 2 0 DC 1V
1 2
3 4
5
7
6
8
Fig. 8: Automatic SPICE Netlist generation using Masala-CHAI.
1-5 6-10 11-15 16-20 21-25 26-30 30-35
Number of Components
0
500
1000
1500
2000
2500
Number
of
Circuits
(a)
1-5 6-10 11-15 16-20 21-25 26-30 31-35
Number of Nodes
0
500
1000
1500
2000
2500
Number
of
Circuits
(b)
1-3 4-6 7-9 10-12 13-15 16-18 19-20
Number of MOSFET
0
500
1000
1500
2000
Number
of
Circuits
(c)
1-10 11-20 21-30 31-40 41-50
Number of Lines in SPICE Netlist
0
500
1000
1500
2000
2500
3000
Number
of
Circuits
(d)
Fig. 9: Distribution of (a) # Components, (b) # Nodes, (c) #
MOSFETs, and (d) #lines of SPICE code in the dataset.
significant challenge for LLMs; (c) focuses on the number
of MOSFETs, where an increase in count corresponds to higher
complexity in SPICE netlist generation; Finally, (d) illustrates
the distribution of SPICE netlist line counts, emphasizing the
circuit complexities. This characterization provides insights
into the dataset, aiding in the evaluation of automated SPICE
generation techniques. By curating this diverse and complex
dataset, we establish a robust benchmark for evaluating and
improving LLM performance in automated SPICE netlist
generation. For more analysis see Appendix A and B.
B. Finetuning
Fine-tuning LLMs is widely used to adapt models to specific
downstream tasks [3], [4]. The model leverages knowledge
acquired from pre-training on large unsupervised language
learning tasks, allowing its weights to be adjusted using a smaller,
task-specific dataset. This enables the model to capture novel
patterns and improve task-specific reasoning without training
from scratch. Fine-tuning LLMs offers several key advantages:
(i) Reduced training time and computational costs by leveraging
pre-trained foundational knowledge, (ii) Enhanced performance
on downstream tasks, particularly beneficial when data is scarce,
and (iii) Improved transfer learning across subtasks within a
domain, increasing efficiency in model deployment.
Benchmarking LLMs: We evaluated the capabilities of
LLMs in analog circuit design, using both open-source
(CodeLlama-70B [39], DeepSeek-V2 [40]) and proprietary
(GPT-3.5,4o-mini [41]) models. We fine-tuned all baseline
LLMs using the corpus collected through the MASALA-CHAI
framework. Open-source models were fine-tuned on a Nvidia
A100 80GB GPU utilizing LoRA [42], while GPT models were
fine-tuned using their respective fine-tuning playground [43].
Metrics: We adopt the Pass@k metric, widely used in code-
generation tasks [3], [4], [14], as our main evaluation measure.
We define Pass@k = 1 −
(n−c
k )
(n
k)
(for k ∈ {1, 5}), where c is
the number of successful trials out of n total trials. In our
experiments, we set n = 15 for all the scenarios.
Testing: We selected 20 design problems as shown in Table II,
frequently used as standalone circuits or subcomponents in
larger designs, to form our evaluation benchmark. LLMs were
prompted to generate the corresponding SPICE netlists.
Discussion: Table III summarizes the Pass@k performance
of various LLMs for SPICE netlist generation. As antici-
pated, open-source models (DeepSeek and CodeLlama)
underperform relative to the proprietary GPT models, both
before and after fine-tuning. Nonetheless, fine-tuning on the
collected dataset, (ID=1–5) provides a slight increase in the
Pass@k for these open-source models, enabling coverage for
additional design IDs (ID=6–8). Similarly, fine-tuned GPT-3.5
and GPT-4o achieve higher scores (ID=1–13) and non-zero
performance for design IDs (ID=14–20). Notably, the fine-tuned
GPT-4o (“Best Result”) successfully generates SPICE netlists
for all design IDs, highlighting the efficacy of the MASALA-
CHAI framework (also illustrated in Figure 1).
Figure 10 showcases representative examples of both
successful and failed netlists generated by LLMs, illustrating
critical design challenges and the effectiveness of our approach.
Notably, in Figure 10: (1) demonstrates the improper use of
an NMOS as a current source rather than forming the intended
cascode configuration; (2) highlights a fundamental error where
the gate of a diode-connected transistor, expected to connect
to the drain, is incorrectly connected to the source; (3) the
complexity of a two-stage design further amplifies the difficulties
in understanding and replicating the intended topology. These
examples show the limits of unoptimized LLMs. Through
the finetuned GPT-4o model, we resolved the issues such
as eliminating floating nets. For more analysis see Appendix C.

## Page 6

TABLE II: Evaluation benchmarks. We refer each design by its ID.
ID Circuit ID Circuit ID Circuit
1 Common-source amplifier 2 2-stage common src amplifier w/ res. load 3 Common-drain amplifier
4 Common-gate amplifier 5 Single-Stage RC Low-Pass Filter 6 Src Degenerated Amplifier
7 Current Mirror 8 Common-source amplifier using active load 9 Cascode amplifier using NMOS and res. load
10 1-stage differential amplifier 11 Diode-connected Amplifier 12 Buffer design using MOSFET
13 2-input NAND gate 14 2-stage op-amp with Miller compensation 15 SRAM cell with 6 transistors
16 2-stage op-amp with diff. i/p and single-handled o/p 17 Fully Diff. Amplifier w/ Common-Mode Feedback 18 Cross-coupled LC oscillator
19 Telescopic cascode op amplifier 20 Bandgap Reference Amplifier
TABLE III: Results on LLMs and Finetuned (‘FT’) variants on MASALA-CHAI dataset. ‘CL’ is CodeLlama. ‘DS’ is DeepSeek.
ID
CL-70B CL-70B+FT DS-34B DS-34B + FT GPT-3.5 GPT-3.5 + FT GPT-4o GPT-4o + FT
Pass @1 @5 @1 @5 @1 @5 @1 @5 @1 @5 @1 @5 @1 @5 @1 @5
1 13.3 57.1 26.7 84.6 26.7 84.6 40 95.8 66.7 100 93.3 100 66.7 100 100 100
2 0.0 0.0 13.3 57.1 6.7 33.3 26.7 84.6 53.3 99.3 86.7 100 66.7 100 100 100
3 26.7 84.6 33.3 91.6 13.3 57.1 46.7 98.1 66.7 100 100 100 80 100 100 100
4 13.3 57.1 33.3 91.6 26.7 84.6 33.3 91.6 60 99.8 80 100 60 99.8 100 100
5 6.7 33.3 26.7 84.6 6.7 33.3 26.7 84.6 46.7 98.1 86.7 100 60 99.8 100 100
6 0.0 0.0 0.0 0.0 0.0 0.0 13.3 57.1 60.0 99.8 73.3 100 80 100 100 100
7 0.0 0.0 6.7 33.3 0.0 0.0 26.7 84.6 46.7 98.1 86.7 100 53.3 99.3 93.3 100
8 0.0 0.0 0.0 0.0 0.0 0.0 6.7 33.3 40.0 95.8 66.7 100 46.7 98.1 100 100
9 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 46.7 98.1 73.3 100 53.3 99.3 86.7 100
10 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 26.7 84.6 86.7 100 46.7 98.1 93.3 100
11 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 66.7 100 40 95.8 100 100
12 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 13.3 57.1 66.7 100 33.3 91.6 86.7 100
13 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 53.3 99.3 20.0 73.6 80 100
14 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 40 95.8 0.0 0.0 73.3 100
15 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 33.3 91.6 0.0 0.0 86.7 100
16 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 40 95.8 0.0 0.0 53.3 99.3
17 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 20 73.6 0.0 0.0 66.7 100
18 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0 0.0 0.0 0.0 53.3 99.3
19 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 13.3 57.1 0.0 0.0 66.7 100
20 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0 0.0 0.0 0.0 40 95.8
Design a Cascode Amplifier
Design a diode-connected
Amplifier
Design a 2 stage op-amp, with differential
inputs and single-handled output
(1) (2) (3)
Fig. 10: Pass/fail design cases generated by fine-tuned model.
C. Integration into the AnalogCoder [14] Framework
AnalogCoder is a feedback-driven LLM framework for auto-
mated analog circuit design. Table IV reports the improvements
in Pass@1 and Pass@5 on Tasks 12-20 by using our fine-tuned
GPT-4o (Table III) with AnalogCoder instead of the base GPT-
4o model (excluding tasks where AnalogCoder achieves 100%
success). AnalogCoder’s Pass@1 rate on Task 20 improves
from 20% to 60% when used with our fine-tuned GPT-4o
model. The fine-tuned GPT-4o model alone only achieves a
Pass@1 of 40%, suggesting that Masala-CHAI fine-tuning and
AnalogCoder feedback together provide best performance.
TABLE IV: Results with AnalogCoder [14]. Many tasks show
improvement and for others, results are the same (100%).
Task ID
AnalogCoder AnalogCoder w/ FT GPT-4o
Pass@1 Pass@5 Pass@1 Pass@5
12 13.3 57.1 80 100
14 73.3 100 100 100
15 13.3 57.1 86.7 100
16 6.7 33.3 53.3 99.3
17 0.0 0.0 60 99.8
19 60 99.8 80 100
20 20 73.6 60 99.8
VI. CONCLUSION
Masala-CHAI is an automated framework for creating large-
scale SPICE netlist datasets from analog textbooks, resulting in
the largest publicly available dataset of 7500 SPICE netlists with
corresponding captions. Fine-tuning LLMs on Masala-CHAI
demonstrates its use in automated SPICE netlist generation
from natural language prompts. Our dataset and codebase (https:
//masala-chai-llm.github.io/) are fully open-sourced and easily
extensible on additional textbooks and documentation.

## Page 7

REFERENCES
[1] M. Liu, N. Pinckney, B. Khailany, and H. Ren, “Verilogeval: Evaluating
large language models for verilog code generation,” in 2023 IEEE/ACM
International Conference on Computer Aided Design (ICCAD). IEEE,
2023, pp. 1–8.
[2] S. Thakur, J. Blocklove, H. Pearce, B. Tan, S. Garg, and R. Karri,
“Autochip: Automating hdl generation using llm feedback,” arXiv preprint
arXiv:2311.04887, 2023.
[3] Y. Lu, S. Liu, Q. Zhang, and Z. Xie, “Rtllm: An open-source benchmark
for design rtl generation with large language model,” in 2024 29th Asia
and South Pacific Design Automation Conference (ASP-DAC). IEEE,
2024, pp. 722–727.
[4] S. Thakur, B. Ahmad, Z. Fan, H. Pearce, B. Tan, R. Karri, B. Dolan-
Gavitt, and S. Garg, “Verigen: A large language model for verilog code
generation,” ACM TODAES, 2023.
[5] R. Kande, H. Pearce, B. Tan, B. Dolan-Gavitt, S. Thakur, R. Karri, and
J. Rajendran, “Llm-assisted generation of hardware assertions,” arXiv
preprint arXiv:2306.14027, 2023.
[6] W. Fang, M. Li, M. Li, Z. Yan, S. Liu, H. Zhang, and Z. Xie, “Assertllm:
Generating and evaluating hardware verification assertions from design
specifications via multi-llms,” arXiv preprint arXiv:2402.00386, 2024.
[7] R. Qiu, G. L. Zhang, R. Drechsler, U. Schlichtmann, and B. Li, “Autobench:
Automatic testbench generation and evaluation using llms for hdl design,”
in Proceedings of the 2024 ACM/IEEE International Symposium on
Machine Learning for CAD, 2024, pp. 1–10.
[8] J. Bhandari, J. Knechtel, R. Narayanaswamy, S. Garg, and R. Karri, “Llm-
aided testbench generation and bug detection for finite-state machines,”
arXiv preprint arXiv:2406.17132, 2024.
[9] H. Wu, Z. He, X. Zhang, X. Yao, S. Zheng, H. Zheng, and B. Yu,
“Chateda: A large language model powered autonomous agent for eda,”
IEEE Transactions on Computer-Aided Design of Integrated Circuits and
Systems, 2024.
[10] M. Liu, T.-D. Ene, R. Kirby, C. Cheng, N. Pinckney, R. Liang, J. Alben,
H. Anand, S. Banerjee, I. Bayraktaroglu et al., “Chipnemo: Domain-
adapted llms for chip design,” arXiv preprint arXiv:2311.00176, 2023.
[11] K. Sayed, C. A. Telmer, A. A. Butchy, and N. Miskov-Zivanov, “Recipes
for translating big data machine reading to executable cellular signal-
ing models,” in Machine Learning, Optimization, and Big Data: Third
International Conference, MOD 2017, Volterra, Italy, September 14–17,
2017, Revised Selected Papers 3. Springer, 2018, pp. 1–15.
[12] E. Holtzapple, C. A. Telmer, and N. Miskov-Zivanov, “Flute: Fast and
reliable knowledge retrieval from biomedical literature,” Database, vol.
2020, p. baaa056, 2020.
[13] Z. Tao, Y. Shi, Y. Huo, R. Ye, Z. Li, L. Huang, C. Wu, N. Bai, Z. Yu,
T.-J. Lin et al., “Amsnet: Netlist dataset for ams circuits,” arXiv preprint
arXiv:2405.09045, 2024.
[14] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“Analogcoder: Analog circuit design via training-free code generation,”
arXiv preprint arXiv:2405.14918, 2024.
[15] AI@Meta, “Llama 3 model card,” 2024. [Online]. Available: https:
//github.com/meta-llama/llama3/blob/main/MODEL CARD.md
[16] A. E. Sertdemir, M. Besenk, T. Dalyan, Y. D. Gokdel, and E. Afacan,
“From image to simulation: An ann-based automatic circuit netlist
generator (img2sim),” in 2022 18th International Conference on Synthesis,
Modeling, Analysis and Simulation Methods and Applications to Circuit
Design (SMACD), 2022, pp. 1–4.
[17] H. B. Gurbuz, A. Balta, T. Dalyan, Y. D. Gokdel, and E. Afacan, “Img2sim-
v2: A cad tool for user-independent simulation of circuits in image format,”
in 2023 19th International Conference on Synthesis, Modeling, Analysis
and Simulation Methods and Applications to Circuit Design (SMACD),
2023, pp. 1–4.
[18] R. Zhong, X. Du, S. Kai, Z. Tang, S. Xu, H.-L. Zhen, J. Hao, Q. Xu,
M. Yuan, and J. Yan, “Llm4eda: Emerging progress in large language mod-
els for electronic design automation,” arXiv preprint arXiv:2401.12224,
2023.
[19] J. Blocklove, S. Garg, R. Karri, and H. Pearce, “Chip-chat: Challenges
and opportunities in conversational hardware design,” in 2023 ACM/IEEE
5th Workshop on Machine Learning for CAD (MLCAD). IEEE, 2023,
pp. 1–6.
[20] Y. Fu, Y. Zhang, Z. Yu, S. Li, Z. Ye, C. Li, C. Wan, and Y. C. Lin,
“Gpt4aigchip: Towards next-generation ai accelerator design automation
via large language models,” in 2023 IEEE/ACM International Conference
on Computer Aided Design (ICCAD). IEEE, 2023, pp. 1–9.
[21] K. Chang, Y. Wang, H. Ren, M. Wang, S. Liang, Y. Han, H. Li, and
X. Li, “Chipgpt: How far are we from natural language hardware design,”
arXiv preprint arXiv:2305.14019, 2023.
[22] C.-C. Chang, Y. Shen, S. Fan, J. Li, S. Zhang, N. Cao, Y. Chen,
and X. Zhang, “LaMAGIC: Language-model-based topology generation
for analog integrated circuits,” in Proceedings of the 41st International
Conference on Machine Learning, ser. Proceedings of Machine Learning
Research, R. Salakhutdinov, Z. Kolter, K. Heller, A. Weller, N. Oliver,
J. Scarlett, and F. Berkenkamp, Eds., vol. 235. PMLR, 21–27 Jul 2024,
pp. 6253–6262. [Online]. Available: https://proceedings.mlr.press/v235/
chang24c.html
[23] B. Razavi, “Design of analog cmos integrated circuits,” 2000.
[24] G. Jocher, A. Chaurasia, and J. Qiu, “Ultralytics YOLO,” Jan. 2023.
[Online]. Available: https://github.com/ultralytics/ultralytics
[25] A. Bochkovskiy, C.-Y. Wang, and H.-Y. M. Liao, “Yolov4: Optimal speed
and accuracy of object detection,” 2020.
[26] C.-Y. Wang, H.-Y. Mark Liao, Y.-H. Wu, P.-Y. Chen, J.-W. Hsieh, and I.-H.
Yeh, “Cspnet: A new backbone that can enhance learning capability of
cnn,” in Proceedings of the IEEE/CVF Conference on Computer Vision
and Pattern Recognition Workshops, 2020, pp. 390–391.
[27] “v3qwe dataset,” sep 2022. [Online]. Available: https://universe.roboflow.
com/project-gtqqq/v3qwe
[28] Y. Lin, S. L. Pintea, and J. C. Van Gemert, “Deep hough-transform
line priors,” in Computer Vision–ECCV 2020: 16th European Conference,
Glasgow, UK, August 23–28, 2020, Proceedings, Part XXII 16. Springer,
2020, pp. 323–340.
[29] A. Agarwal and J. Lang, Foundations of Analog and Digital Electronic
Circuits (The Morgan Kaufmann Series in Computer Architecture and
Design). San Francisco, CA, USA: Morgan Kaufmann Publishers Inc.,
2005.
[30] P. R. Gray, Analysis and Design of Analog Integrated Circuits, 5th ed.
Wiley Publishing, 2009.
[31] A. S. Sedra and K. C. Smith, Microelectronic Circuits Revised Edition,
5th ed. USA: Oxford University Press, Inc., 2007.
[32] B. Razavi, “Fundamentals of microelectronics,” 2021.
[33] T. C. Carusone, D. A. Johns, and K. W. Martin, “Analog
integrated circuit design. 2nd ed.” 2012. [Online]. Available: https:
//api.semanticscholar.org/CorpusID:199583062
[34] B. Razavi, “Design of analog cmos integrated circuits,” 2000.
[35] R. L. Boylestad, “Electronic devices and circuit theory 11th edition,” 2014.
[Online]. Available: https://api.semanticscholar.org/CorpusID:267935065
[36] D. Bates and A. Malvino, Electronic Principles. McGraw-Hill
US Higher Ed USE Legacy, 2015. [Online]. Available: https:
//books.google.com/books?id=FKtZCgAAQBAJ
[37] T. L. Floyd and D. Buchla, Electronics Fundamentals: Circuits, Devices
& Applications, 8th ed. USA: Prentice Hall Press, 2009.
[38] D. Neamen, Microelectronic Circuit Analysis and Design. McGraw-Hill
Science/Engineering/Math, 2006.
[39] B. Roziere, J. Gehring, F. Gloeckle, S. Sootla, I. Gat, X. E. Tan, Y. Adi,
J. Liu, R. Sauvestre, T. Remez et al., “Code llama: Open foundation
models for code,” arXiv preprint arXiv:2308.12950, 2023.
[40] A. Liu, B. Feng, B. Wang, B. Wang, B. Liu, C. Zhao, C. Dengr, C. Ruan,
D. Dai, D. Guo et al., “Deepseek-v2: A strong, economical, and efficient
mixture-of-experts language model,” arXiv preprint arXiv:2405.04434,
2024.
[41] OpenAI, “GPT-4,” Mar. 2023. [Online]. Available: https://openai.com/
research/gpt-4
[42] E. J. Hu, yelong shen, P. Wallis, Z. Allen-Zhu, Y. Li, S. Wang, L. Wang,
and W. Chen, “LoRA: Low-rank adaptation of large language models,”
in International Conference on Learning Representations, 2022. [Online].
Available: https://openreview.net/forum?id=nZeVKeeFYf9
[43] OpenAI, “GPT finetuning,” Mar. [Online]. Available: https://platform.
openai.com/docs/guides/fine-tuning

## Page 8

APPENDIX
A. Evaluation
We leverage graph-matching concepts to compare the struc-
tural similarity between SPICE netlists. Unlike traditional
approaches, we parse the netlists to extract components and their
connections. We identify components by types (e.g., Resistor,
Capacitor, MOSFET) to ensure that our graph representation
is invariant to component labels used during netlist generation.
This yields a graph “G”, where nodes are circuit components
and edges are interconnections, as illustrated Figure 11. We use
Graph Edit Distance (GED) to compare representations of two
SPICE netlists. GED (G1,G2) calculates the sum of the costs
associated with transforming G1 into G2 through a series of
edit operations on nodes and edges, such as insert, delete, and
substitute. We normalize the GED to yield a final metric in the
range 0 to 100. This normalization is achieved by calculating the
maximum GED, obtained by summing the number of nodes and
edges in both graphs and scaling accordingly. This metric allows
for a consistent comparison across different circuits. Thus, the
similarity score between the two netlists is:
S =

1 −
GED(G1, G2)
GEDmax

× 100% (1)
B. Verifying against AMSNet [13] netlists
We demonstrated the effectiveness of our approach by
evaluating it using the AMSNet [13] dataset. Unlike AMSNet,
which relies on an algorithmic method supplemented with some
manual effort to generate SPICE netlists, our method introduces
a fully automated flow (Figure 5). When comparing the SPICE
netlists generated by our method to those produced by AMSNet,
our approach achieved a 100% similarity score, as defined
earlier. This validates the robustness of our flow and motivates us
to extend the evaluation to additional circuit diagrams. Figure 11
illustrates an example of graphs representation of SPICE netlist
generated by AMSNet (➊) and our Masala-CHAI framework
(➋). The results demonstrate that the number of components and
their neighboring elements is identical across both methods, with
differences in component and net naming conventions. These
findings strengthen Masala-CHAI accuracy and scalability in
generating accurate netlists for various circuit diagrams.
C. Role of high-quality dataset on Finetuning
As a part of our study, we fine-tuned the GPT-4o with
the AMSNet [13] dataset as well and compared the quality of
response with the MASALA-CHAI dataset. Figure 12 shows
the responses for a simple “Common Source Amplifier” (ID=1)
from the fine-tuned model using AMSNet and MASALA-CHAI
dataset, respectively. AMSNet lacks a description of the SPICE
netlist present in the dataset, which hinders its performance for
supervised fine-tuning (SFT), making it not ready to use for
LLM-based tasks that require high-quality description alongside
the code for unlocking its full potential. MASALA-CHAI also
offers such a pipeline to generate description alongside the
SPICE netlist from a document, as shown Figure 5 (➑).

## Page 9

Voltage
Source
(Vdd)
Inductor
(L1)
MOSFET
(M1)
Capacitor
(C1)
Resistor
(Rs)
MOSFET
(M2)
Voltage
Source
(Vb2)
Voltage
Source
(Vb1)
2
3
3
4
VDD
5
3
6
Voltage
Source
(V1)
Inductor
(L1)
MOSFET
(M1)
Capacitor
(C1)
Resistor
(R1)
MOSFET
(M2)
Voltage
Source
(V3)
Voltage
Source
(V2)
4
5
5
2
1
6
5
3
AMSNet Masala-CHAI
1 2
Fig. 11: Graph representation of SPICE netlists generated by AMSNet [13] and Masala-CHAI. They are identical.
Prompt: Write a spice netlist for Common-source amplifier with R load.A basic amplifier where the input signal is
applied to the gate, the source is grounded, and the output is taken from the drain through a resistor load.
AMSNet Masala-CHAI
* NMOS Amplifier Circuit
M1 Vout Vin 0 0 NMOS
* Resistor
RD VDD Vout 10k
* Voltage Sources
Vin Vin 0 DC 1
VDD VDD 0 DC 5
.MODEL NMOS NMOS (LEVEL=1 VTO=1
KP=1.0e-4 LAMBDA=0.02)
* Common-source amplifier with R load
M1 VDD VDD 3 3 NMOS
M2 5 6 0 0 NMOS
R1 4 0 1k
R2 0 3 1k
M3 VDD 2 5 5 NMOS
M4 3 2 VDD VDD NMOS
.MODEL NMOS NMOS (LEVEL=1 VTO=1
KP=1.0e-4 LAMBDA=0.02)
.OP
.END
Fig. 12: Example showing responses from a fine-tuned GPT-4o model with AMSNet and MASALA-CHAI dataset, respectively.

