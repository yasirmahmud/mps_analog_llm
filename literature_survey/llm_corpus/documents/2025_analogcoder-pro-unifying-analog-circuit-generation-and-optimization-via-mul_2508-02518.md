---
title: "AnalogCoder-Pro: Unifying Analog Circuit Generation and Optimization via Multi-modal LLMs"
authors: "Lai, Yao; Poddar, Souradip; Lee, Sungyoung; Chen, Guojin; Hu, Mengkang; Yu, Bei; Luo, Ping; Pan, David Z."
date: "2025/08/04"
category: "end_to_end"
source_url: "https://arxiv.org/abs/2508.02518"
pdf: "papers/2025_analogcoder-pro-unifying-analog-circuit-generation-and-optimization-via-mul_2508-02518.pdf"
sha256: "b02e8e7a7aa78dc78b5c70ca4f63f6d5d86390f1ae0ba1cf38854edfe9ec5080"
pages_extracted: 13
---

# AnalogCoder-Pro: Unifying Analog Circuit Generation and Optimization via Multi-modal LLMs

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AnalogCoder-Pro: Unifying Analog Circuit
Generation and Optimization via Multi-modal LLMs
Yao Lai, Souradip Poddar, Sungyoung Lee, Guojin Chen, Mengkang Hu,
Bei Yu, Ping Luo, David Z. Pan
Abstract—Despite recent advances, analog front-end design still
relies heavily on expert intuition and iterative simulations, which
limits the potential for automation. We present AnalogCoder-
Pro, a multimodal large language model (LLM) framework that
integrates generative and optimization techniques. The frame-
work features a multimodal diagnosis-and-repair feedback loop
that uses simulation error messages and waveform images to
autonomously correct design errors. It also builds a reusable
circuit tool library by archiving successful designs as modular
subcircuits, accelerating the development of complex systems.
Furthermore, it enables end-to-end automation by generating cir-
cuit topologies from target specifications, extracting key parame-
ters, and applying Bayesian optimization for device sizing. On a
curated benchmark suite covering 13 circuit types, AnalogCoder-
Pro successfully designed 28 circuits and consistently outper-
formed existing LLM-based methods in figures of merit. The
codes will be released at github.com/laiyao1/AnalogCoderPro.
Index Terms—Analog circuit design, large language model,
topology generation, device sizing.
I. INTRODUCTION
Large Language Models (LLMs) [1] have recently demon-
strated strong potential for automating chip design workflows
[2]–[4]. Remarkable progress has been made in digital circuit
design, including RTL code generation and correction [5]–[19],
script generation [19]–[21], and EDA documentation question
answering [22]. While digital design tasks often benefit from
standardized representations and abundant datasets, analog
circuit design presents a fundamentally different set of hurdles
that limit the direct transfer of such automation advances.
Analog circuit design automation, especially when applying
LLMs, faces several key challenges, summarized as follows.
(1) Complexity. Analog circuit design is inherently complex
due to diverse device behaviors and multidimensional per-
formance trade-offs [41]–[44]. Design flows rely heavily on
expert heuristics and extensive simulations, since even minor
adjustments in topology or device sizing can drastically alter
circuit functionality, leading to a combinatorial explosion in the
search space. (2) Abstraction gap. Digital design languages
such as Verilog [45] provide high-level functional abstractions,
enabling concise descriptions (e.g., a digital adder can be spec-
ified in a single line of code). In contrast, analog design inher-
Yao Lai, Mengkang Hu, and Ping Luo are with the Department of Computer
Science, The University of Hong Kong, Hong Kong SAR.
Souradip Poddar, Sungyoung Lee, and David Z. Pan are with the Department
of Electrical and Computer Engineering, The University of Texas at Austin,
United States.
Guojin Chen and Bei Yu are with the Department of Computer Science and
Engineering, The Chinese University of Hong Kong, Hong Kong SAR.
TABLE I LLM-based Methods for Analog Design
Work
Multiple Training- MLLM Circuit Circuit Open-
Types1
Free Debug2
Gen. Opt. Source
CktGNN [23] • • •
LADAC [24] • • •
ADO-LLM [25] • • •
LaMAGIC [26], [27] • •
AnalogCoder [28] • • ◦ • •
SPICEPilot [29] • • ◦ •
LEDRO [30] • •
Aritsan [31] • •
AmpAgent [32] • •
Atelier [33] • • •
AnalogXpert [34] • ◦ •
Malasa-Chai [35] • • •
AnalogGenie/Lite [36], [37] • • • •
AnalogFed [38] • • •
AutoCircuit-RL [39] • •
SPICEAssistant [40] • ◦ • •
AnalogCoder-Pro • • • • • •
1
Whether the work supports multiple circuit type designs.
2
• - Full multi-modal debugging capability by signal images (e.g., waveform, frequency
response), ◦ - Text-only debugging.
ently operates at the device level, where even basic functions
such as addition require explicitly wiring together multiple
transistors and passive components. This abstraction gap limits
the transfer of automation advances from digital to analog
design. (3) Data scarcity. Public code repositories contain
very little hardware description language content—Verilog, for
instance, accounts for less than 0.1% of GitHub projects [46].
SPICE [47], the predominant analog design language, has
an even smaller footprint, making it particularly challenging
for LLMs to acquire analog design rules effectively. (4)
Underuse of multimodal information. Simulation outputs
such as waveforms and frequency responses contain critical
information for analog circuit analysis. Yet, these non-textual
modalities are rarely integrated into automated processing
pipelines. This underutilization limits the ability of LLMs to
diagnose issues and refine designs effectively. (5) Lack of uni-
fied topology–sizing optimization. Topology generation and
device sizing are typically conducted as separate, sequential
stages, which can lead to suboptimal results. When inherent
topological constraints cannot be addressed through parameter
tuning, costly redesign cycles often follow.
Conventional analog automation tools provide valuable so-
lutions but still face limitations. Modern device sizing algo-
rithms [48]–[57] improve sample efficiency yet are mostly
restricted to predefined architectures. Topology exploration
methods face high simulation costs [58], [59], depend on
expert-crafted equations [60]–[62], offer limited diversity [63],
and carry the risk of generating invalid structures [64]. In
1
arXiv:2508.02518v2
[cs.LG]
31
Aug
2025

## Page 2

addition, automated synthesis approaches such as [65] still
demand substantial setup effort, limiting the applicability to
predefined libraries.
Recent LLM-based approaches for analog circuit design,
partly shown in Table I, have emerged in three main directions,
including topology generation via code or netlist synthe-
sis [28], [29], [34], [35], LLM-assisted circuit sizing [24],
[25], [31]–[33], and question–answering assistants [66]–[69].
While promising, these methods lack comprehensive solutions
that integrate multimodal feedback and unified topology–sizing
optimization. For example, AnalogCoder [28] and AnalogX-
pert [34] have not yet incorporated waveform images for circuit
analysis. AnalogGenie [36] supports end-to-end topology gen-
eration through pre-training and fine-tuning, yet still depends
on manual data collection and per-class fine-tuning, which
restricts scalability.
To overcome these limitations, we propose AnalogCoder-
Pro, a training-free and end-to-end multimodal LLM frame-
work that jointly performs topology generation and device
sizing via Python code generation. To the best of our knowl-
edge, AnalogCoder-Pro is the first framework in this direction.
Given a natural-language description of the target circuit’s
functionality, structure, and performance goals, the framework
autonomously generates, verifies, and optimizes analog cir-
cuits. For instance, when asked to “Design a multi-stage op-
amp that optimizes the FoM (GBW/Power),” it can produce
and iteratively refine a functional design. The key innovations
of AnalogCoder-Pro include domain-specific design prompts,
a feedback-enhanced flow that leverages multimodal inputs
such as waveforms, a dedicated circuit tool library that enables
flexible reuse of fundamental analog blocks, and LLM-assisted
parameter extraction for circuit sizing. Together, these com-
ponents allow the framework to deliver high-quality, verified
analog front-end designs with minimal human intervention.
The main contributions of this work are:
• Introduction of AnalogCoder-Pro, the first training-free
multimodal LLM framework for end-to-end analog circuit
design, capable of transforming natural-language design
objectives into optimized netlists.
• Development of a feedback-enhanced multimodal design
flow that leverages waveforms, simulation logs, and code
diagnostics to iteratively refine designs and correct func-
tional errors.
• Creation of a circuit tool library that enables flexible
reuse and composition of fundamental analog blocks for
complex circuit design.
• Proposal of an integrated parameter optimization frame-
work that combines LLM-generated initial topologies
with automated refinement for effective circuit sizing.
• Experimental validation showing that AnalogCoder-Pro
outperforms existing LLM-based methods in both design
success rate and circuit performance across diverse analog
circuit types.
𝐕𝐢𝐧
𝐑𝐃
𝐕𝐃𝐃
𝐌𝟏
* Common-Source Amp
* Power supply
VDD Vdd 0 5V
* Input voltage signal
VIN Vin 0 2.5V AC 1uV
* N-channel MOSFET
M1 Vout Vin 0 0 NMOS
W=50u L=1u
* Load resistor
RD Vdd Vout 10k
.end
SPICE Code
from PySpice.Spice.Netlist import Circuit
circuit = Circuit('Common-Source Amplifier')
circuit.V('dd', 'Vdd', 0, '5V')
circuit.V('in', 'Vin', 0, '2.5V AC 1uV')
circuit.M('1', 'Vout', 'Vin', 0, 0,
model='nmos', w=50e-6, l=1e-6)
circuit.R('1', 'Vdd', 'Vout', 10k)
Python
Code
0
Vin
Vout
Vdd
Circuit Diagram
Please design a common-source amplifier
with resistive load with Python code.
NodeComponent
Fig. 1 Code-based Analog Circuit Representation. Given a
design task, the LLM generates Python code with the PySpice
library. The circuit can be equivalently represented as (1) a
circuit diagram, (2) SPICE code, or (3) PySpice-based Python.
II. PRELIMINARIES
A. Analog Circuit Design Challenges and Representation
Design Challenges. Analog circuit design is one of the most
intricate stages in the integrated circuit flow, involving both
front-end and back-end tasks. In the front-end, designers must
synthesize suitable topologies and optimize device parameters
to meet stringent, often conflicting performance criteria such
as gain, bandwidth, and power consumption. Achieving a
high figure of merit (FoM) requires navigating large, high-
dimensional design spaces and managing complex trade-offs,
which are further complicated by the strong coupling between
topology and sizing decisions, where an optimal choice in one
aspect may be suboptimal in another. The back-end translates
these logical designs into manufacturable layouts [70]–[73],
ensuring process compliance.
Recent advances in artificial intelligence have brought new
tools to analog design, including reinforcement learning [48],
[50], [74], graph-based generative models [23], [75], and
language model-based approaches [24], [25], [28]–[30], [32],
[34], [35], [55]. Nevertheless, many of these techniques tackle
topology generation and parameter optimization in isolation,
which can result in incompatibilities that hinder overall per-
formance and necessitate iterative redesigns.
To mitigate these issues, recent research has explored uni-
fied frameworks that jointly consider topology and sizing.
AnalogGenie [36], [37] first pre-trains on a manually collected
set of circuit topologies, then fine-tunes the model specif-
ically on high-performance topologies. The method in [65]
automatically ranks candidate topologies according to user re-
2

## Page 3

quirements, then performs sizing and verification on each, but
remains limited to a predefined set of topologies. Artisan [31]
focuses on behavioral-level synthesis for op-amps design,
omitting device-level refinement. Thus, despite these advances,
a truly scalable, general-purpose methodology for end-to-end,
performance-driven analog design automation remains elusive.
Circuit Representation. SPICE [47] is the industry-
standard netlist format for circuit-level modeling and simula-
tion, in which circuits are described as interconnected elements
(e.g., resistors, voltage sources, MOSFETs) with associated
parameters, as illustrated by the amplifier example in Fig. 1. In
our framework, we integrate PySpice [76], a Python interface
to SPICE simulators (e.g., Ngspice) that enables circuit con-
struction, simulation control, and results parsing within a uni-
fied scripting environment. This approach leverages Python’s
programmability and ecosystem for efficient circuit generation,
simulation, and analysis, and aligns with the strong capabil-
ity of LLMs in Python code generation [77]. Consequently,
PySpice serves as a natural entry point for LLM-based circuit
synthesis, replacing manual netlist creation.
B. Multimodal Large Language Models
Multimodal large language models (MLLMs) extend tradi-
tional LLMs by jointly processing visual and textual inputs. An
MLLM typically includes a vision encoder and a text encoder,
which produce feature tokens from images I and text T:
zvision = Evision(I), ztext = Etext(T), y = LM([zvision, ztext]),
where y is the output sequence.
In EDA, MLLMs enable richer interpretation of circuit
diagrams and waveform plots, outperforming text-only models
in pattern recognition tasks. For example, Fig. 2 shows that
a noisy triangular waveform is misclassified by an LLM (text
input only) as a damped sinusoid, whereas an MLLM correctly
identifies it from the waveform image. This visual reason-
ing capability is crucial for reliable waveform identification,
thereby improving circuit debugging and validation.
III. FRAMEWORK AND METHODS
A. Framework Overview
AnalogCoder-Pro is a multimodal LLM-based framework
that interprets natural-language task descriptions to automat-
ically generate Python code representing functional, optimized
analog circuits. As shown in Fig. 3, it consists of mul-
tiple modules, including prompt engineering, a multimodal
feedback-enhanced flow, and a circuit tool library, followed
by device sizing. Prompt engineering enhances the design rea-
soning through strategic, problem-solving prompts. The mul-
timodal feedback-enhanced flow incorporates multiple checks,
including waveform validation, to feed error messages back to
the LLM, enabling it to correct failed designs. The circuit tool
library serves as a modular repository that organizes designed
circuits into reusable components, enabling efficient retrieval
and integration into more complex designs. Following topology
generation, it conducts device sizing, including parameter
extraction and optimization, to finalize the circuit design.
What type of waveform is it?
Vout: [-0.85964 -0.81450 -0.77124 -0.73029
… -0.87874 -0.93567 -0.94588 -0.92499]
Time: [0. 0.00251 0.00501 0.00752 …
0.99248 0.99499 0.99749 1. ]
a damped sinusoidal waveform …
What type of waveform is it?
a triangular waveform …
LLM
Multimodal
LLM
Fig. 2 LLMs vs. MLLMs in Waveform Identification. LLMs
provided with textual data (top) misidentify a noisy triangular
waveform, whereas MLLMs, given the corresponding wave-
form image (bottom), correctly identify its type.
B. Prompt Design
To leverage the code generation capabilities of LLMs
for circuit design, we adopt a systematic prompt design
strategy, comprising: (1) programming language selection,
(2) in-context learning [78], and (3) Chain-of-Thought
reasoning [79]. Given the superior performance of LLMs in
Python code generation, owing to the prevalence of Python
in model pre-training [77], we instruct the models to produce
executable Python code compatible with the PySpice library,
thereby overcoming the lack of SPICE code in common
training datasets [46]. For in-context learning, we provide a
single detailed example of a two-stage amplifier, enabling the
LLM to generalize and standardize its output across diverse
design tasks. To further enhance design quality, we employ
Chain-of-Thought prompting [79], which guides the LLM
to generate a step-by-step design plan before producing the
corresponding code.
C. Multimodal Feedback-enhanced Flow
To ensure the correctness of analog circuit designs, the veri-
fication in feedback flow is divided into five stages as shown
in Fig. 4: (1) Requirement Check, which ensures basic design
elements such as the input voltage (Vin) and output voltage
(Vout) are present, and validates the inclusion of essential
components; (2) Simulation and Operating Point Check, where
the circuit is simulated to detect issues like floating nodes
and verify MOSFET operating conditions (e.g., Vgs > Vth);
(3) DC Sweep Check, which involves varying Vin to observe
changes in Vout, ensuring signal path integrity and identifying
optimal bias points; (4) Function Check, where simulations
validate functionalities such as gain and periodicity; and (5)
Waveform Check: simulation data are plotted as waveforms
(e.g., transient, AC sweep) using the Matplotlib package,
analogous to an oscilloscope, to visually verify the output
3

## Page 4

Basic Circuit
Design Task
Design
Instruction
Feedback-
Enhanced Flow
Circuit Tool
Library
Functional
&
Optimized
Circuit
Composite
Circuit
Design Task
Retrieval
Instruction
Add to library
Design
Instruction
Feedback-
Enhanced Flow
Query Retrieval
Circuit Design
Task
Design
Instruction
Correct
Circuit
Wrong
Circuit
Normal
Flow
Max Attempts=3
Max Attempts=3
(a)
(b)
(c)
Functional
Circuit
LLM
Multimodal LLM
1.Topology
Generation
2. Circuit Device Sizing
Parameter
Extraction
&
Device
Sizing
Optimizer
Fig. 3 Framework Overview. AnalogCoder-Pro consists of two main stages: (1) topology generation and (2) circuit device
sizing. (a) Standard method. A direct design flow often results in failed circuits. (b) AnalogCoder-Pro for basic circuits.
design prompts are processed through a multimodal feedback-enhanced flow, where LLMs iteratively correct errors; successful
circuits are stored in the circuit tool library. (c) AnalogCoder-Pro for composite circuits. Relevant subcircuits are retrieved
from the library and incorporated into the design prompt to facilitate composite circuit generation. After topology generation,
the device sizing is conducted, which gives the final functional and optimized circuit.
signal. The used waveform types for different circuit types
are shown in Table II.
If errors are detected during these stages, diagnostic mes-
sages (e.g., “Missing Vin node” or “Gain is less than 1e−5”)
are generated from predefined templates to guide iterative re-
finement. The framework permits up to three design iterations,
each informed by error feedback and waveform analysis, to
progressively improve the design.
TABLE II Waveform Types for Different Circuit Types
Circuit Type Abbr. Transient AC FFT DC Sweep DC Transfer
Mixer Mix • •
Comparator Comp •
Filter Filt •
Oscillator Osc •
Integrator Int •
Differentiator Diff •
Schmitt trigger Sch • •
D. Circuit Tool Library
As analog circuit design tasks grow more complex, it be-
comes increasingly challenging for LLMs to generate correct
circuits. To address this, basic circuits can be modularized
into subcircuits, facilitating their integration into composite
designs. Inspired by tool-based LLM studies [80], [81], we
developed a circuit tool library to store correctly designed
subcircuits for reuse in more complex designs. As illustrated
in Fig. 5, our approach comprises two key processes: adding
circuits to the library and retrieving circuits from the library
for composite designs. After AnalogCoder-Pro completes a
basic circuit design, the corresponding circuit representations
and specifications (e.g., gain) are stored in the library. For
repeated tasks, the library retains the optimal design based on
specifications. Subcircuit descriptions and circuit information
serve as query keys, while circuit representations and usage
instructions serve as values. During composite circuit design,
the task description is used to query the library, retrieving
the required subcircuits along with their specifications. The
design prompt then integrates this information with the task
description, enabling AnalogCoder-Pro to generate the final
design. For example, when designing an op-amp integrator, the
process involves retrieving the single-stage op-amp subcircuit
and integrating it into the design.
E. Topology Generation Example
Below is a complete example conversation for circuit topology
generation in AnalogCoder-Pro. In this example, the LLM is
tasked with designing an RC-shift oscillator, with the op-amp
subcircuit from the circuit tool library provided for reuse. In
addition, the MLLM feedback-enhanced flow is introduced for
circuit diagnosis and repair.
Instruction (Round 1; LLM)
You aim to design a topology for a given circuit described in the text.
Please ensure your designed circuit topology works properly and achieves
the design requirements.
To make the task easier, you can directly use the opamp subcircuits by
inserting:
from opamp import *
# Declare the operational amplifier subcircuit
circuit.subcircuit(Opamp())
# Create a subcircuit instance
# Parameter order: instance name, subcircuit name,
# the non-inverting input, the inverting input,
# the output
circuit.X('1', 'Opamp', 'Vinp', 'Vinn', 'Vout')
The DC bias voltage for both input terminals (Vinn and Vinp) is 2.5V. For
AC coupling/grounding purposes, these terminals should be referenced
to the 2.5V power supply rather than ground. Here is an example:
...(An example code for an opamp circuit with resistive load)
As you have seen, the output of your designed topology should be in
a complete Python code, describing the topology of integrated analog
circuits according to the design plan.
Please make sure your Python code is compatible with PySpice.
4

## Page 5

8. Fuctional Circuits
Vin
RD
M1
VDD
Vout
6. Function Check
The circuit doesn’t function correctly. The gain is less than 1e-5.
The circuit doesn’t function correctly. Common-Mode gain is larger than Differential-Mode.
The circuit doesn’t function correctly. Not enough peaks were detected for periodicity.
Error Feedback Examples
1. Check the function of the designed circuit.
python circuit_function.py > function_result.txt
Build simulation script:
circuit_function.py
5. DC Sweep Check
Error Feedback Examples
Build simulation script:
circuit_dc_sweep.py
python circuit_dc_sweep.py
1. Check how the output changes with the input.
2. Find and replace the optimal input.
>_ dc_sweep_result.txt
According to dc sweep analysis, changing the input voltage does not change the output.
Reference operating point: [Vin 1.00 Vout 0.00 ...]
Suggestion: Please check the netlist.
4. Simulation and Operating Point Check
For [NMOS] [M1], the gate node voltage is lower than the source node voltage.
Suggestion: Please set [NMOS] [M1] with an activated state by increasing the gate voltage
or decreasing the source voltage and make sure Vgs> Vth.
Error Feedback Examples
Build simulation script:
circuit_sim.py
python circuit_sim.py
1. Check all MOSFETs: Vgs > Vth, Vds > Vgs-Vth
>_ operating_point_result.txt
3. Requirement Check
The given input node ([Vin]) is not found in the Netlist.
Suggestion: You can replace the nodes actually used for input/output with the given names.
For a common-drain amplifier, the [Vout] should be connected to source.
Suggestion: Please connect the [Vout] to the source node.
There is no resistance in the netlist.
Suggestion: Please add a resistance load in the netlist.
Error Feedback Examples
1. “Vin”, “Vout” exist in the netlist.
2. Netlist satisfies the description.
2. Design Code
(Re-)Generation
1. Analog Design Task
Please design a single-stage common source amp with resistant load R in Python.
Input: Vin
Output: Vout
>_ circuit.py
7. Waveform Check
Get the generated
waveform from
previous analysis:
Waveform.png
Get waveform analysis from multimodal LLMs:
Analyze the attached waveform image.
0 10 20 30 40 50
Time (ms)
0
1
2
3
4
5
Voltage
(V)
Schmitt Trigger Response
Vin
Vout
The waveform shows a distorted pattern
with irregular transitions. The output
voltage (Vout) remains predominantly
high, close to 5V…
File output
Check files Checklist
LLM Build file
Input
Run python file
Fig. 4 Multimodal Feedback-Enhanced Design Flow. A feedback-enhanced flow in which the LLM integrates textual error
messages and waveform anomalies to iteratively refine designs without human intervention. Error messages are generated from
predefined templates, ensuring applicability across diverse circuit types. Waveform images are automatically produced from
simulation data using the matplotlib package. For illustration, the waveform example is derived from a Schmitt trigger circuit.
Please give the runnable code without any placeholders.
There are some tips you should remember all the time:
• For the MOSFET definition circuit.MOSFET(name,
drain, gate, source, bulk, model, w=w1,
l=l1), be careful about the parameter sequence.
• ...
5

## Page 6

from PySpice.Spice.Netlist
import Circuit from PySpice.Unit import *
circuit = Circuit('Common-Source Amp')
circuit.model('nmos_model', 'nmos', level=1, kp=1e-8,
vto=0.5)
circuit.V('dd', 'Vdd', circuit.gnd, 5.0)
circuit.V('in', 'Vin', circuit.gnd, 1.5)
circuit.MOSFET('1', 'Vout', 'Vin', circuit.gnd,
circuit.gnd, model='nmos_model', w=50e-6, l=1e-6)
circuit.R('1', 'Vout', 'Vdd', 1@u_kΩ)
A single-stage common-source amplifier
with resistive load R
Gain = 14 dB, Phase Diff = 180°, …
Value
Key
Compare
&
Add
Circuit Tool library - Keys
Id Circuit Gain …
1
1-stage
common-source
amplifier
14 …
2
3-stage
common-source
amplifier
44 …
10
2-stage
amplifier with
Miller C
6 …
… … … …
An op-amp integrator
Circuit Tool library - Values
Id Call Method
Cod
e
1
circuit.X('1',
'CommonSou
rceAmp', 'Vin',
'Vout')
…
2
circuit.X('1',
'ThreeStageA
mp', 'Vin',
'Vout')
…
… … …
1. Query: Search for
necessary libraries
[11, opmap, Gain=194 dB, Phase Diff=180°]
circuit.X(‘op', 'Opamp', 'Vinp', 'Vinn', 'Vout')
2. Retrieve
3. Retrieve
,
Basic Design Task
Please design [an op-amp integrator].
You can directly use the following subcircuits.
[Opamp, Gain=194dB, Input=Vinn/Vinp, …] by
inserting: [circuit.subcircuit(Opamp())
circuit.X(‘op', 'Opamp', 'Vinp', 'Vinn', 'Vout')]
Code for Design
Composite Design Task
Specifications after Simulation
Fig. 5 Circuit Tool library. Top: Addition of new tools derived from successfully designed basic circuits. Here, descriptions
and specifications are keys, while design codes are stored as values. Bottom: Retrieval of tools from the library for designing
composite circuits. The process begins with the LLM querying the necessary tools using the task description. Subsequently, the
keys and values of the retrieved tools, with the task description, are employed as instructions for circuit design.
Please first give a detailed design plan and then write the code.
Question
Design an RC phase-shift oscillator.
Input node name: -.
Output node name: Vout.
Answer
Response (Round 1; LLM)
An RC phase-shift oscillator uses an amplifier with a feedback network
consisting of three RC stages. Each RC stage provides approximately 60°
phase shift, for a total of 180° through the feedback network. Combined
with the 180° phase shift from the inverting amplifier, this creates a 360°
phase shift around the loop, satisfying the phase criterion for oscillation.
Design Requirements:
• Create an oscillator circuit using an operational amplifier.
• Implement three RC stages for phase shifting.
• Set appropriate gain to ensure oscillation.
• No input node is needed as this is an oscillator (self-generating
signal).
• Output node should be named Vout.
Circuit Components:
• Operational amplifier in inverting configuration.
• Three RC stages for phase shifting.
• Feedback resistors to set appropriate gain.
Design Details:
• For an RC phase-shift oscillator to work, the amplifier gain must
be at least 29 (approximately 29.3) to overcome losses in the RC
network.
• Three identical RC stages will be used, with R = 10 kΩ and
C = 10 nF.
• The oscillation frequency will be approximately f = 1
2πRC
√
6
≈
650 Hz.
• The feedback resistor Rf will be set to provide sufficient gain
(> 29).
• The input resistor Rin will be chosen to achieve this gain.
Let’s implement this design:
from PySpice.Spice.Netlist import Circuit
from PySpice.Unit import *
from opamp import *
circuit = Circuit('RC Phase-Shift Oscillator')
# Define the MOSFET models
circuit.model('nmos_model', 'nmos',
level=1, kp=100e-6, vto=0.5)
circuit.model('pmos_model', 'pmos',
level=1, kp=50e-6, vto=-0.5)
# Power supply
circuit.V('dd', 'Vdd', circuit.gnd, 5@u_V)
# Set reference voltage (2.5V) as virtual ground
circuit.V('ref', 'Vref', circuit.gnd, 2.5@u_V)
# Declare the opamp subcircuit
circuit.subcircuit(Opamp())
# Create opamp instance (inverting configuration)
circuit.X('op', 'Opamp', 'Vref', 'feedback', 'Vout')
# RC phase-shift network (three stages)
# First RC stage
circuit.R('1', 'Vout', 'node1', 10@u_kOhm)
circuit.C('1', 'node1', 'Vref', 10@u_nF)
# Second RC stage
circuit.R('2', 'node1', 'node2', 10@u_kOhm)
circuit.C('2', 'node2', 'Vref', 10@u_nF)
# Third RC stage
circuit.R('3', 'node2', 'feedback', 10@u_kOhm)
circuit.C('3', 'feedback', 'Vref', 10@u_nF)
# Feedback resistor and input resistor to set gain
# Gain needs to be > 29 for oscillation
# Feedback resistor
circuit.R('f', 'feedback', 'Vout', 300@u_kOhm)
# Input resistor to set gain
circuit.R('in', 'feedback', 'Vref', 10@u_kOhm)
simulator = circuit.simulator()
During functional testing, the oscillator failed to maintain
stable oscillation—the amplitude gradually decreased to zero.
The waveform, generated with Matplotlib, was submitted to
a multimodal LLM for further analysis. We give an example
6

## Page 7

as follows. The parts highlighted in orange will be updated
according to the circuit type.
Instruction (Multimodal LLM)
Analyze the attached waveform output image from an RC phase-
shift oscillator circuit.
Vout should exhibit periodic oscillations. The actual output
waveform differs.
The testbench initializes the op-amp non-inverting input
(Vinp) to 2.51V and the inverting input (Vinn) to 2.5V.
0.0000 0.0050 0.0100 0.0150 0.0200
Time [s]
2.50
2.50
2.50
2.50
2.50
2.50
2.50
2.50
Voltage
[V]
Output Signal
vout
0.0000 0.0050 0.0100 0.0150 0.0200
Time [s]
2.50
2.50
2.50
2.50
2.50
Voltage
[V]
Input, Reference and Feedback Signals
feedback
vref
node2
node1
Response (Multimodal LLM)
The waveform shows a damped oscillation that quickly settles
to a steady DC level rather than exhibiting the expected
sustained periodic oscillations for an RC phase-shift oscillator.
The output signal (vout) initially displays several rapidly
decaying oscillations with a peak amplitude deviation of
approximately ±0.01V around the 2.5V level, occurring within
the first 0.001 seconds of simulation time.
The waveform analysis from the MLLM and the error mes-
sages obtained during the check process are jointly fed back
to the LLM, enabling it to revise the design and regenerate a
correct circuit implementation.
Instruction (Round 2; LLM)
I am encountering an error when running the PySpice code.
Below is the error message:
Average oscillation period: 0.000002 s
Maximum amplitude: 0.000000 V
The oscillation amplitude is too small.
Waveform Analysis:
The waveform shows a dumped oscillation ... (the output of
MLLM waveform analysis)
Identify any structural errors in this circuit’s topology (e.g., miss-
ing components, incorrect connections) and rewrite the complete
corrected code from scratch, with each modification marked and
explained in comments.
Response (Round 2; LLM)
Corrected RC Phase-Shift Oscillator Design
The main issues with the previous design were:
• Insufficient gain to sustain oscillations
• Improper RC network configuration
• Lack of initial condition to start oscillations
Let me redesign the circuit with proper topology and gain:
from PySpice.Spice.Netlist import Circuit
# ... (defining MOSFET models and power supply is omitted)
# Declare the opamp subcircuit
circuit.subcircuit(Opamp())
# Create opamp instance (inverting configuration)
# CORRECTION: Using proper non-inverting input
# and inverting input connections
circuit.X('op', 'Opamp', 'Vref', 'feedback', 'Vout')
# CORRECTION: Proper RC ladder network
# for phase-shift oscillator
# First RC stage - from output to first node
circuit.R('1', 'Vout', 'node1', 10@u_kOhm)
circuit.C('1', 'node1', 'Vref', 10@u_nF)
# Second RC stage
circuit.R('2', 'node1', 'node2', 10@u_kOhm)
circuit.C('2', 'node2', 'Vref', 10@u_nF)
# Third RC stage
circuit.R('3', 'node2', 'feedback', 10@u_kOhm)
circuit.C('3', 'feedback', 'Vref', 10@u_nF)
# CORRECTION: Significantly increased
# gain to ensure oscillation. For RC phase-shift
# oscillator, gain must be at least 29
# Using much higher gain (100)
# to ensure strong oscillation
# Feedback resistor (increased)
circuit.R('f', 'feedback', 'Vout', 1000@u_kOhm)
# Input resistor
circuit.R('in', 'feedback', 'Vref', 10@u_kOhm)
# CORRECTION: Add a small initial condition
# to kickstart oscillation
# This is needed because SPICE simulations
# start from a steady state
circuit.C('kickstart', 'Vout', 'Vref', 1@u_nF,
initial_condition=0.1@u_V)
simulator = circuit.simulator()
Key corrections:
• Increased gain: The gain of the amplifier is now much higher
(Rf /Rin = 1000k/10k = 100), well above the minimum
required gain of 29 for an RC phase-shift oscillator.
• Proper RC network: The RC network is configured correctly with
each capacitor connected to the reference voltage.
• Initial condition: Added a capacitor with an initial condition to
kickstart the oscillation.
• Feedback path: Ensured proper feedback from the last RC stage
to the inverting input of the op-amp.
This circuit should now produce sustained oscillations at approximately
650 Hz.
Based on the analysis of the waveforms, the oscillator was
revised primarily by adjusting the feedback resistor value to
increase the loop gain. This modification enables the circuit to
sustain stable oscillations. The post-simulation waveform of
the revised oscillator is shown in Fig. 6.
0.0000 0.0050 0.0100 0.0150 0.0200
Time [s]
1.80
2.00
2.20
2.40
2.60
2.80
3.00
3.20
3.40
Voltage
[V]
Output Signal
vout
0.0000 0.0050 0.0100 0.0150 0.0200
Time [s]
2.30
2.40
2.50
2.60
2.70
Voltage
[V]
Input, Reference and Feedback Signals
feedback
vref
node2
node1
Fig. 6 Waveform of a functional RC Phase-Shift Oscillator
F. LLM-assisted Circuit Device Sizing
Upon completion of circuit topology generation, the resulting
code contains nominal parameters (e.g., transistor sizing, bias
voltages, capacitances, resistances) that ensure functionality
7

## Page 8

but may be suboptimal in performance. We address this by
instructing the LLM to: (1) analyze the generated netlist, (2)
extract all tunable parameters with appropriate search ranges,
and (3) reformat the circuit into a parameterized template
for use with optimization frameworks (e.g., Optuna). We give
an example instruction and response as follows. In this case,
the LLM extracts the load resistance value and the transistor
width as tunable parameters, producing a parameterized circuit
implementation that can be directly invoked by an optimization
framework.
Instruction (LLM)
I need to convert my PySpice circuit netlist into a parameterized form
for automated optimization. Please transform the following original cir-
cuit description into a parameter-based implementation with appropriate
search ranges.
For reference, here is an example of what an original circuit might look
like: ...
Here is my actual circuit code:
circuit = Circuit('Single-Stage Amplifier')
# MOSFET model
circuit.model('nmos_model', 'nmos', **nmos_params)
# ...
circuit.MOSFET('1', 'Vout', 'Vin', circuit.gnd,
circuit.gnd, model='nmos_model', w=5e-6, l=0.045e-6)
circuit.R('1', 'Vout', 'Vdd', 10@u_kOhm)
Required Output
1) A circuit creation function that uses a params dictionary.
2) A parameter search range dictionary (for Optuna or similar opti-
mization tools).
3) Reasonable parameter constraints with tailored search ranges:
• Transistor width (W) should be within 1–500× the corre-
sponding length (L).
• ...
Response (LLM)
def create_circuit(params):
circuit = Circuit('Single-Stage Amplifier')
circuit.model('nmos_model', 'nmos', **nmos_params)
# ...
circuit.MOSFET('1', 'Vout', 'Vin', circuit.gnd,
circuit.gnd, model='nmos_model',
w=params['w_M1'], l=0.045e-6)
circuit.R('1', 'Vout', 'Vdd', f"{params['r_load']}k")
return circuit
param_ranges_definition = {
'w_M1': {'min': 0.045e-6, 'max': 22.5e-6, 'log': True},
'r_load': {'min': 5, 'max': 20, 'log': True},
}
initial_params = {'w_M1': 5e-6, 'r_load': 10}
With the tunable parameters, we use the Bayesian optimizer
for circuit sizing. For each parameter set proposed by the op-
timizer, the input voltage bias is not treated as an optimization
variable, because the output voltage is highly sensitive to the
bias point: even small changes in input voltage can cause large
variations in output voltage. Therefore, we propose a novel
multi-resolution input bias search process to determine the
optimal input bias (Vin) before the main optimization loop.
As illustrated in Fig. 7, this algorithm performs DC sweeps
at progressively refined resolutions to locate the bias point
that sets the output voltage close to VDD/2 in single-ended,
single-supply configurations, providing a balanced operating
point and placing the key transistors in their intended operating
0.0 0.2 0.4 0.6 0.8 1.0 1.2
Input Bias Voltage (V)
0.00
0.25
0.50
0.75
1.00
1.25
Output
Voltage
(V)
Coarse Bias Search
VDD/2 = 0.6V
Coarse Search Points
Best Coarse Point
Fine Search Region
0.500 0.525 0.550 0.575 0.600 0.625 0.650 0.675 0.700
Input Bias Voltage (V)
0.2
0.4
0.6
0.8
1.0
Output
Voltage
(V)
High Sensitivity Region
(Small Vin Large Vout)
Fine Bias Search
Fine Bias Search
Best Coarse Point
Optimal Bias Point
Fig. 7 Multi-Resolution Search for Input Bias. It conducts a
coarse-to-fine DC sweep to automatically locate the input bias
that sets the output voltage closest to VDD/2, ensuring proper
circuit biasing while reducing optimization dimensionality.
regions. Once this optimal bias is established, we proceed with
a comprehensive circuit simulation to evaluate the figure of
merit (FoM), which then provides the reward signal for the op-
timizer. This decoupling strategy reduces the dimensionality of
the optimization space, improves convergence efficiency, and
ensures proper circuit biasing for each parameter configuration
under evaluation.
IV. EXPERIMENTAL RESULTS
A. Experimental settings
Benchmark. For topology generation, we consider 28 de-
sign tasks spanning 13 circuit types, including amplifiers,
operational amplifiers, current mirrors, mixers, filters, and
comparators, as summarized in Table III. Each circuit type
is accompanied by functional verification code that validates
the generated netlists through simulation results. The tasks are
categorized into three difficulty levels (easy, medium, and hard)
according to the number of components and the complexity of
their interconnections. A circuit is considered composite when
it involves the reuse of subcircuits. In our task design, Tasks
9 and 22–28 are composite circuits that utilize the circuit tool
library, whereas all other tasks correspond to basic circuits.
For circuit device sizing, we evaluated the ability of LLMs to
generate performance-driven analog circuit topologies using 12
distinct design instructions (see Fig. 8).
Metrics. For topology generation, we adopt the widely-
used Pass@k, which measures the probability that at least one
correct solution appears in k sampled generations. Formally,
it is defined as Pass@k = 1 − Ck
n−c/Ck
n, where n denotes
the total number of attempts, c denotes the number of correct
generations, and Ck
n denotes the binomial coefficient. In our
experiments, we set n = 30. Note that we use the feedback-
enhanced flow, allowing each generation task to be revised
up to two additional times based on error information. For
8

## Page 9

circuit device sizing, we align with AnalogGenie [36], using
FoM = GBW × CL/Power as the primary optimization
target. For completeness, we also evaluate Gain and GBW as
alternative optimization goals.
Design a
Single-Stage Amplifier
Multi-Stage Op-amp
topology that
optimizes
Gain
GBW
FoM
(GBW*CL/Power)
Target Guide …
Fig. 8 Unified Generation and Optimization Prompt Strate-
gies. All prompt variants illustrated here are evaluated in
Table VII, with the target-guided objective (e.g., Gain, GBW,
or user-defined FoM) being optional for topology optimization.
Hyper-parameters. For topology generation, we set the
temperature to 0.5 and top p to 1.0 to encourage output
diversity, while for parameter extraction, the temperature is
fixed at 0.0. For circuit device sizing, we use Bayesian opti-
mization with the Optuna package [82] (TPE sampler, 1000
trials), including 25% random initialization (250 trials) and
multivariate sampling to account for parameter dependencies.
The feedback flow allows up to three attempts, balancing
token cost and success rate since most errors are corrected
within this range. For multimodal LLMs, we use the native ver-
sion of each model when available (e.g., GPT-4.1); otherwise,
we employ alternative variants (GPT-4o for DeepSeek-V3,
Qwen-VL for Qwen-Coder). All experiments are conducted
with the BPTM 45nm PDK, consistent with AnalogGenie [37].
B. Main Results
For the topology generation task, we evaluate the functional
correctness of the generated netlists. Each task is attempted
30 times per model and counted as solved if at least one
valid netlist is produced. Fig. 9 summarizes the number of
solved tasks as a leaderboard. In just over two years, the
number of circuits successfully designed by LLMs doubled
from 14 with GPT-3.5 to 28 with GPT-5, highlighting the rapid
advancement of LLMs in circuit design. Table IV provides a
quantitative comparison of multiple LLMs evaluated in the
same AnalogCoder-Pro framework under identical settings.
The results indicate that the best proprietary models outper-
form open-source alternatives, with GPT-5 achieving the best
performance (28/28 tasks, 98.6% Pass@5). Moreover, there
is a clear positive correlation between general-purpose LLM
capability and analog circuit generation performance, with
more advanced models consistently achieving higher success
rates across all circuit types.
C. Unified Generation and Optimization
For each circuit type, we compared two prompt strate-
gies: one including an explicit design objective (e.g., “De-
sign a multi-stage op-amp topology that optimizes FoM
(GBW*CL/Power)”) and one omitting it (e.g., “Design a
multi-stage op-amp”), to evaluate the effectiveness of unified
design. Optimization is performed with 1000 samples per trial,
and the best value from each run is recorded. The unified
generation and optimization process is repeated 30 times, and
Fig. 9 Leaderboard of LLM analog circuit design.
we report the best value among all 30 trials. As shown in
Table VII, each base model is reported with two rows (without
vs. with target-guided prompts), and the target-guided setting
consistently improves objective-specific results, demonstrating
that the LLM can effectively incorporate optimization goals
during topology generation.
Beyond the analysis of unified prompt strategies, we further
evaluate the performance of AnalogCoder-Pro in terms of
FoM. According to Table VII, AnalogCoder, aided by multi-
resolution input bias search, generates circuits that achieve
competitive baseline performance even with only the initial
LLM-assigned sizes and no further optimization. In contrast,
AnalogCoder-Pro combines LLM-based parameter extraction
with Bayesian optimization, yielding substantial gains over
AnalogGenie, including up to 2× better op-amp FoM. To
directly assess the role of multi-resolution input bias search,
we compared it with several alternatives, including single-
resolution search, fixed bias, and treating input bias as an
optimizable parameter, using the best FoM netlist generated by
Claude-3.7 (Fig. 10(a)). For each method, the optimizer was
run using five random seeds, constraining the input bias range
to 0.25Vdd–0.75Vdd to avoid supply rail nonlinearities; vari-
ance across runs is shown as error bars. The single-resolution
baseline employed 2000 uniform steps, while the multi-
resolution strategy used 20, 200, and 2000 steps at coarse,
medium, and fine stages. As shown in Fig. 11, the multi-
resolution search consistently outperforms all baselines across
every evaluation metric. Additional amplifier topologies gen-
erated by AnalogCoder-Pro are provided in Fig. 10(b) and (c).
D. Ablation study
Our ablation study investigates the contribution of different
components in the AnalogCoder-Pro framework. Specifically,
we examine the effects of (i) having the LLM directly generate
SPICE netlists instead of Python code, (ii) removing in-context
learning from the design prompts, (iii) removing chain-of-
thought guidance from the design prompts, (iv) excluding the
feedback-enhanced workflow, and (v) omitting the circuit tool
library, in comparison with (vi) the full framework. The results
9

## Page 10

TABLE III Benchmark Descriptions. Difficulties are distinguished by background colors (easy, medium, and hard).
ID Type Circuit Description ID Type Circuit Description
1 Amplifier Common-source amp. with R load 15 Amplifier 2-stage amplifier with Miller compensation
2 Amplifier 3-stage common-source amplifer with R loads 16 Current Mirror Cascode current mirror
3 Amplifier Common-drain amp. with R load 17 Opamp Op-amp with active current mirror loads
4 Amplifier Common-gate amp. with R load 18 Opamp Common-source op-amp with R loads
5 Amplifier Cascode amp. with R load 19 Mixer A Gilbert cell mixer
6 Inverter NMOS inverter with R load 20 Opamp Cascode op-amp with cascode loads
7 Inverter Logical inverter 21 Opamp 2-stage op-map with active loads
8 Current Mirror NMOS constant current source with R load 22 Oscillator Wien Bridge oscillator
9 Comparator an op-amp comparator 23 Oscillator RC Shift oscillator
10 Filter a passive low-pass filter 24 Integrator Op-map integrator
11 Filter a passive high-pass filter 25 Differentiator Op-map differentiator
12 Filter a passive band-pass filter 26 Adder Op-map adder
13 Filter a passive band-stop filter 27 Subtractor Op-map substrator
14 Amplifier Common-source amp. with diode-connected load 28 Schmitt trigger Non-inverting Schimiit trigger
TABLE IV Main results. All LLMs are evaluated using the complete AnalogCoder-Pro framework
Model
Circuit Type
Qwen-2.5-Coder-32B GPT-4.1-nano Gemini-2.5-Flash DeepSeek-V3 GPT-4.1 Claude-3.7-Sonnet GPT-5
Task ID Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5 Pass@1 Pass@5
1 Amplifier 80.0 100.0 73.3 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
2 Amplifier 80.0 100.0 6.7 31.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
3 Amplifier 70.0 99.9 33.3 89.1 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
4 Amplifier 66.7 99.8 90.0 100.0 80.0 100.0 96.7 100.0 100.0 100.0 100.0 100.0 100.0 100.0
5 Amplifier 20.0 70.2 26.7 81.5 93.3 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
6 Inverter 100.0 100.0 86.7 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
7 Inverter 100.0 100.0 86.7 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
8 Current Mirror 73.3 100.0 30.0 85.7 76.7 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
9 Comparator 53.3 98.6 73.3 100.0 3.3 16.7 100.0 100.0 100.0 100.0 100.0 100.0 53.3 98.6
10 Filter 80.0 100.0 93.3 100.0 96.7 100.0 100.0 100.0 96.7 100.0 100.0 100.0 100.0 100.0
11 Filter 13.3 53.8 86.7 100.0 100.0 100.0 100.0 100.0 90.0 100.0 100.0 100.0 100.0 100.0
12 Filter 16.7 62.7 13.3 53.8 100.0 100.0 86.7 100.0 80.0 100.0 100.0 100.0 96.7 100.0
13 Filter 0.0 0.0 6.7 31.0 100.0 100.0 36.7 91.8 36.7 91.8 66.7 100.0 100.0 100.0
14 Amplifier 43.3 95.7 36.7 91.8 26.7 81.5 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
15 Amplifier 30.0 85.7 66.7 99.8 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
16 Current Mirror 0.0 0.0 16.7 62.7 100.0 100.0 100.0 100.0 76.7 100.0 100.0 100.0 96.7 100.0
17 Opamp 0.0 0.0 3.3 16.7 30.0 85.7 60.0 99.4 66.7 99.8 6.7 33.3 66.7 99.8
18 Opamp 0.0 0.0 16.7 62.7 96.7 100.0 96.7 100.0 93.3 100.0 100.0 100.0 100.0 100.0
19 Mixer 20.0 70.2 6.7 31.0 3.3 16.7 66.7 99.8 50.0 97.9 100.0 100.0 96.7 100.0
20 Opamp 6.7 31.0 0.0 0.0 100.0 100.0 76.7 100.0 90.0 100.0 100.0 100.0 100.0 100.0
21 Opamp 0.0 0.0 0.0 0.0 90.0 100.0 6.7 31.0 46.7 96.9 86.7 100.0 56.7 99.1
22 Oscillator 16.7 62.7 16.7 62.7 66.7 99.8 10.0 43.4 46.7 96.9 46.7 98.1 16.7 62.7
23 Oscillator 0.0 0.0 0.0 0.0 76.7 100.0 30.0 85.7 73.3 100.0 100.0 100.0 100.0 100.0
24 Integrator 13.3 53.8 80.0 100.0 100.0 100.0 96.7 100.0 96.7 100.0 100.0 100.0 100.0 100.0
25 Differentiator 30.0 85.7 40.0 94.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0 100.0
26 Adder 60.0 99.4 63.3 99.7 100.0 100.0 100.0 100.0 30.0 85.7 100.0 100.0 80.0 100.0
27 Subtractor 0.0 0.0 26.7 81.5 93.3 100.0 93.3 100.0 73.3 100.0 100.0 100.0 96.7 100.0
28 Schmitt trigger 20.0 70.2 20.0 70.2 93.3 100.0 40.0 94.0 33.3 89.1 66.7 100.0 90.0 100.0
Avg 35.5 62.1 39.3 69.5 83.1 92.9 82.0 94.5 81.4 98.5 91.9 97.5 91.1 98.6
# Solved 21 21 25 25 28 28 28 28 28 28 28 28 28 28
TABLE V Ablation Study Results.
Backbone Ablation Pass@1 Pass@5 # Solved
GPT-4.1-nano
SPICE netlist 14.4 33.2 20
w/o in-context learning 25.8 47.3 22
w/o chain-of-thought 25.0 50.8 22
w/o feedback flow 18.8 45.0 19
w/o library tool 24.0 45.1 20
full framework 39.3 69.5 25
DeepSeek-V3
SPICE netlist 77.6 89.8 27
w/o in-context learning 64.2 83.0 27
w/o chain-of-thought 75.2 89.4 26
w/o feedback flow 69.8 80.3 27
w/o library tool 75.0 84.8 24
full framework 82.0 94.5 28
TABLE VI Comparison of Multimodal Feedback-Enhanced
Flow. Each model is evaluated with and without Multi-
modal LLM assistance (with • denoting assistance) in average
Pass@1. The Avg. reports the mean score across categories.
Circuit type abbreviations follow Table II.
Model Multimodal Osc Int Diff Sch Comp Mix Filt Avg.
Qwen2.5 1.7 6.7 40.0 20.0 20.0 16.7 16.7 17.4
-Coder-32B • 8.3 13.3 30.0 20.0 53.3 20.0 27.5 24.6
GPT-4.1
43.3 93.3 96.7 36.7 93.3 43.3 75.0 68.8
• 60.0 96.7 100.0 33.3 100.0 50.0 75.8 73.7
DeepSeek-V3
8.3 100.0 96.7 50.0 100.0 63.3 77.5 70.8
• 20.0 96.7 100.0 40.0 100.0 66.7 80.8 72.0
Claude-3.7 70.0 100.0 100.0 26.7 100.0 100.0 91.7 84.1
-Sonnet • 73.3 100.0 100.0 66.7 100.0 100.0 91.7 90.2
10

## Page 11

TABLE VII Performance Comparison for Unified Circuit Generation and Optimization. For each instruction in Fig. 8, 30
netlist generation attempts are performed. Each correct netlist is optimized with 1000 samples, and the best performance among
the 30 trials is reported. Maximum values are in bold. AnalogGenie and CktGNN results are from the original papers.
Method
Base Model
Target- Single-Stage Amp Single-Stage Opamp Multi-Stage Amp Multi-Stage Opamp Avg.
# of Simulations guided2
Gain GBW FoM1
Gain GBW FoM Gain GBW FoM Gain GBW FoM Rank
CktGNN [23] GNN • - - - - - - - - - - - 10.9 -
AnalogGenie(>1k) [36] GPT-2 • - - - - - - - - - - - 36.5 -
AnalogCoder(1) [28]
GPT-4o
13.9 93.2 1.7 12.8 113.1 0.6 40.6 2076.2 5.7 18.0 154.6 0.5 10.5
• 16.2 191.7 1.1 13.4 85.1 0.8 43.8 4195.4 8.8 35.5 880.8 2.4 9.1
DeepSeek-V3
0.7 23.6 1.7 13.5 147.7 0.9 37.5 5330.6 8.5 27.8 1050.9 4.1 9.4
• 26.2 198.5 1.0 29.7 153.5 2.6 37.5 9808.7 13.1 7.8 443.1 4.2 7.6
Claude-3.7
16.0 49.4 1.9 14.5 130.0 1.1 35.8 314.8 1.2 67.9 2384.8 12.1 8.2
• 30.5 157.4 1.0 23.1 13.7 1.1 0.0 4081.3 165.9 40.0 899.0 0.5 7.9
AnalogCoder-Pro(1k)
GPT-4o
18.6 416.8 2.0 20.6 299.1 2.9 56.3 23111.9 133.2 33.3 1406.1 9.5 4.9
• 18.9 443.3 2.4 20.4 260.3 3.6 55.7 35873.3 124.0 50.2 1122.2 37.4 4.5
DeepSeek-V3
16.1 50.4 1.9 20.5 279.9 3.0 55.9 22579.2 127.3 36.4 2346.0 14.1 5.7
• 27.5 418.8 2.5 36.2 363.3 14.5 56.4 41316.7 185.7 36.6 2243.9 36.2 2.5
Claude-3.7
16.1 100.6 1.9 27.8 281.3 8.5 53.2 5298.2 135.1 67.9 6279.4 43.4 4.3
• 45.3 413.3 3.2 39.2 266.6 7.4 46.1 17366.0 413.2 74.9 3193.0 73.9 2.6
1
Gain (dB) = 20 × log10(Vout/Vin), GBW (MHz) = Gain (linear) × Bandwidth, FoM (MHz·pF/µW) = GBW × CL/Power, where CL = 100 pF.
2
Target-guided design indicates that specific design objectives (e.g., maximizing the Figure of Merit, FoM) were set for the LLM, as shown in Fig. 8.
(a)
(b) (c)
Gain = 49.3dB, BW = 0.34MHz,
Power = 0.13mW, CL = 100pF,
FoM = 73.9
Fig. 10 Topologies generated by AnalogCoder-Pro. (a) Op-
amp with the highest FoM, also evaluated in Fig. 11; (b), (c)
Amplifiers generated with target-guided design prompts.
in Table V show that removing any component reduces per-
formance. To further evaluate the role of MLLM, we compare
the results with and without multimodal waveform analysis
in the feedback flow. As shown in Table VI, using MLLM
consistently improves circuit topology generation success rate.
V. CONCLUSION
In this work, we introduce AnalogCoder-Pro, a unified, end-
to-end multimodal LLM framework for automated analog
circuit design. By combining multimodal diagnosis-and-repair
mechanisms with a reusable circuit tool library and inte-
grated performance-driven topology generation, AnalogCoder-
Pro achieves state-of-the-art performance and substantially
improves success rates across diverse circuit design tasks.
REFERENCES
[1] Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge
Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt,
Sam Altman, Shyamal Anadkat, et al. Gpt-4 technical report. arXiv,
2023.
[2] Ruizhe Zhong, Xingbo Du, Shixiong Kai, Zhentao Tang, Siyuan Xu,
Hui-Ling Zhen, Jianye Hao, Qiang Xu, Mingxuan Yuan, and Junchi Yan.
Llm4eda: Emerging progress in large language models for electronic
design automation. arXiv, 2023.
[3] Jingyu Pan, Guanglei Zhou, Chen-Chia Chang, Isaac Jacobson, Jiang
Hu, and Yiran Chen. A survey of research in large language models for
electronic design automation. ACM Transactions on Design Automation
of Electronic Systems (TODAES), 30(3):1–21, 2025.
[4] Wenji Fang, Jing Wang, Yao Lu, Shang Liu, Yuchao Wu, Yuzhe Ma,
and Zhiyao Xie. A survey of circuit foundation model: Foundation ai
models for vlsi circuit design and eda. arXiv, 2025.
[5] Jason Blocklove, Siddharth Garg, Ramesh Karri, and Hammond Pearce.
Chip-chat: Challenges and opportunities in conversational hardware
design. In Workshop on Machine Learning for CAD (MLCAD), pages
1–6. IEEE, 2023.
[6] Kaiyan Chang, Ying Wang, Haimeng Ren, Mengdi Wang, Shengwen
Liang, Yinhe Han, Huawei Li, and Xiaowei Li. Chipgpt: How far are
we from natural language hardware design. arXiv, 2023.
[7] Shailja Thakur, Baleegh Ahmad, Hammond Pearce, Benjamin Tan,
Brendan Dolan-Gavitt, Ramesh Karri, and Siddharth Garg. Verigen: A
large language model for verilog code generation. ACM Transactions on
Design Automation of Electronic Systems (TODAES), 2023.
[8] Shailja Thakur, Jason Blocklove, Hammond Pearce, Benjamin Tan,
Siddharth Garg, and Ramesh Karri. Autochip: Automating hdl generation
using llm feedback. arXiv, 2023.
[9] Yonggan Fu, Yongan Zhang, Zhongzhi Yu, Sixu Li, Zhifan Ye, Chaojian
Li, Cheng Wan, and Yingyan Celine Lin. Gpt4aigchip: Towards next-
generation ai accelerator design automation via large language models.
In IEEE/ACM International Conference on Computer Aided Design
(ICCAD), pages 1–9. IEEE, 2023.
[10] YunDa Tsai, Mingjie Liu, and Haoxing Ren. Rtlfixer: Automatically
fixing rtl syntax errors with large language models. arXiv, 2023.
[11] Zehua Pei, Hui-Ling Zhen, Mingxuan Yuan, Yu Huang, and Bei Yu.
Betterv: Controlled verilog generation with discriminative guidance.
arXiv, 2024.
11

## Page 12

0 200 400 600 800 1000
Num of Samples
0
15
30
45
60
Gain
(dB)
Gain Optimization
Multi-Resolution DC Sweep
Single-Resolution DC Sweep
Optimizer-Tuned Input Bias
Fixed Input Bias (Vdd/2)
0 200 400 600 800 1000
Num of Samples
0
600
1200
1800
2400
3000
GBW
(MHz)
GBW Optimization
Multi-Resolution DC Sweep
Single-Resolution DC Sweep
Optimizer-Tuned Input Bias
Fixed Input Bias (Vdd/2)
0 200 400 600 800 1000
Num of Samples
0
20
40
60
80
FoM
(MHz/μW)
AnalogGenie: 36.5
CktGNN: 10.9
FOM Optimization
Multi-Resolution DC Sweep
Single-Resolution DC Sweep
Optimizer-Tuned Input Bias
Fixed Input Bias (Vdd/2)
Fig. 11 Comparison of Input Bias Voltage Selection Methods. Each method is evaluated over 5 independent optimization runs.
The proposed multi-resolution bias selection approach achieves faster and more accurate convergence to the optimal operating
points, resulting in improved circuit performance metrics. The circuit topology for evaluation is shown in Fig. 10(a).
[12] Xufeng Yao, Haoyang Li, Tsz Ho Chan, Wenyi Xiao, Mingxuan Yuan,
Yu Huang, Lei Chen, and Bei Yu. Hdldebugger: Streamlining hdl
debugging with large language models. ACM Transactions on Design
Automation of Electronic Systems (TODAES), 2024.
[13] Yaoyu Zhu, Di Huang, Hanqi Lyu, Xiaoyun Zhang, Chongxiao Li,
Wenxuan Shi, Yutong Wu, Jianan Mu, Jinghua Wang, Yang Zhao, et al.
Codev-r1: Reasoning-enhanced verilog generation. arXiv, 2025.
[14] Mohammad Akyash, Kimia Azar, and Hadi Kamali. Rtl++: Graph-
enhanced llm for rtl code generation. arXiv, 2025.
[15] Mingjie Liu, Yun-Da Tsai, Wenfei Zhou, and Haoxing Ren. Craftrtl:
High-quality synthetic data generation for verilog code models with
correct-by-construction non-textual representations and targeted code
repair. arXiv, 2024.
[16] Chenhui Deng, Yun-Da Tsai, Guan-Ting Liu, Zhongzhi Yu, and Haoxing
Ren. Scalertl: Scaling llms with reasoning data and test-time compute
for accurate rtl code generation. arXiv, 2025.
[17] Shang Liu, Wenji Fang, Yao Lu, Jing Wang, Qijun Zhang, Hongce
Zhang, and Zhiyao Xie. Rtlcoder: Fully open-source and efficient llm-
assisted rtl code generation technique. IEEE Trans. Comput.-Aided Des.
Integr. Circuits Syst., 2024.
[18] Yang Zhao, Di Huang, Chongxiao Li, Pengwei Jin, Muxin Song, Yinan
Xu, Ziyuan Nan, Mingju Gao, Tianyun Ma, Lei Qi, et al. Codev: Em-
powering llms with hdl generation through multi-level summarization.
arXiv, 2024.
[19] Mingjie Liu, Teodor-Dumitru Ene, Robert Kirby, Chris Cheng, Nathaniel
Pinckney, Rongjian Liang, Jonah Alben, Himyanshu Anand, Sanmitra
Banerjee, Ismet Bayraktaroglu, et al. Chipnemo: Domain-adapted llms
for chip design. arXiv, 2023.
[20] Haoyuan Wu, Zhuolun He, Xinyun Zhang, Xufeng Yao, Su Zheng,
Haisheng Zheng, and Bei Yu. Chateda: A large language model powered
autonomous agent for eda. IEEE Trans. Comput.-Aided Des. Integr.
Circuits Syst. (TCAD), 2024.
[21] Haoyuan Wu, Haisheng Zheng, Zhuolun He, and Bei Yu. Divergent
thoughts toward one goal: Llm-based multi-agent collaboration system
for electronic design automation. arXiv, 2025.
[22] Yuan Pu, Zhuolun He, Tairu Qiu, Haoyuan Wu, and Bei Yu. Cus-
tomized retrieval augmented generation and benchmarking for eda tool
documentation qa. In Proceedings of the 43rd IEEE/ACM International
Conference on Computer-Aided Design (TODAES), pages 1–9, 2024.
[23] Zehao Dong, Weidong Cao, Muhan Zhang, Dacheng Tao, Yixin Chen,
and Xuan Zhang. Cktgnn: Circuit graph neural network for electronic
design automation. In International Conference on Learning Represen-
tations (ICLR), 2023.
[24] Chengjie Liu, Yijiang Liu, Yuan Du, and Li Du. Ladac: Large language
model-driven auto-designer for analog circuits. Authorea Preprints, 2024.
[25] Yuxuan Yin, Yu Wang, Boxun Xu, and Peng Li. Ado-llm: Analog design
bayesian optimization with in-context learning of large language models.
In International Conference on Computer-Aided Design (ICCAD), 2024.
[26] Chen-Chia Chang, Yikang Shen, Shaoze Fan, Jing Li, Shun Zhang,
Ningyuan Cao, Yiran Chen, and Xin Zhang. Lamagic: Language-model-
based topology generation for analog integrated circuits. arXiv, 2024.
[27] Chen-Chia Chang, Wan-Hsuan Lin, Yikang Shen, Yiran Chen, and Xin
Zhang. Lamagic2: Advanced circuit formulations for language model-
based analog topology generation. In International Conference on
Machine Learning (ICML), 2025.
[28] Yao Lai, Sungyoung Lee, Guojin Chen, Souradip Poddar, Mengkang
Hu, David Z Pan, and Ping Luo. AnalogCoder: Analog Circuit Design
via Training-Free Code Generation. In AAAI Conference on Artificial
Intelligence (AAAI), 2025.
[29] Deepak Vungarala, Sakila Alam, Arnob Ghosh, and Shaahin Angizi.
Spicepilot: Navigating spice code generation and simulation with ai
guidance. arXiv, 2024.
[30] Dimple Vijay Kochar, Hanrui Wang, Anantha Chandrakasan, and Xin
Zhang. Ledro: Llm-enhanced design space reduction and optimization
for analog circuits. arXiv, 2024.
[31] Zihao Chen, Jiangli Huang, Yiting Liu, Fan Yang, Li Shang, Dian Zhou,
and Xuan Zeng. Artisan: Automated operational amplifier design via
domain-specific large language model. In Design Automation Conference
(DAC), 2024.
[32] Chengjie Liu, Weiyu Chen, Anlan Peng, Yuan Du, Li Du, and Jun Yang.
Ampagent: An llm-based multi-agent system for multi-stage amplifier
schematic design from literature for process and performance porting.
arXiv, 2024.
[33] Jinyi Shen, Zihao Chen, Ji Zhuang, Jiangli Huang, Fan Yang, Li Shang,
Zhaori Bi, Changhao Yan, Dian Zhou, and Xuan Zeng. Atelier: An
automated analog circuit design framework via multiple large language
model-based agents. Authorea Preprints, 2024.
[34] Haoyi Zhang, Shizhao Sun, Yibo Lin, Runsheng Wang, and Jiang Bian.
Analogxpert: Automating analog topology synthesis by incorporating
circuit design expertise into large language models. arXiv, 2024.
[35] Jitendra Bhandari, Vineet Bhat, Yuheng He, Hamed Rahmani, Siddharth
Garg, and Ramesh Karri. Masala-chai: A large-scale spice netlist dataset
for analog circuits by harnessing ai, 2025.
[36] Jian Gao, Weidong Cao, Junyi Yang, and Xuan Zhang. Analoggenie: A
generative engine for automatic discovery of analog circuit topologies.
In International Conference on Learning Representations (ICLR), 2025.
[37] Jian Gao, Weidong Cao, and Xuan Zhang. Analoggenie-lite: En-
hancing scalability and precision in circuit topology discovery through
lightweight graph modeling. In International Conference on Machine
Learning (ICML), 2025.
[38] Qiufeng Li, Shu Hong, Jian Gao, Xuan Zhang, Tian Lan, and Weidong
Cao. Analogfed: Federated discovery of analog circuit topologies with
generative ai, 2025.
[39] Prashanth Vijayaraghavan, Luyao Shi, Ehsan Degan, Vandana Mukher-
jee, and Xin Zhang. Autocircuit-rl: Reinforcement learning-driven llm
for automated circuit topology generation. In International Conference
on Machine Learning (ICML), 2025.
[40] Simon Nau, Jan Krummenauer, and André Zimmermann. Spiceassistant:
Llm using spice simulation tools for schematic design of switched-mode
power supplies. arXiv preprint arXiv:2507.10639, 2025.
[41] Behzad Razavi. Design of Analog CMOS Integrated Circuits. McGraw-
Hill, Inc., 2000.
12

## Page 13

[42] Guido Stehr, Helmut Graeb, and Kurt Antreich. Performance trade-off
analysis of analog circuits by normal-boundary intersection. In Design
Automation Conference (DAC), pages 958–963, 2003.
[43] Daniel Mueller-Gritschneder, Helmut Graeb, and Ulf Schlichtmann. A
successive approach to compute the bounded pareto front of practical
multiobjective optimization problems. SIAM Journal on Optimization,
20(2):915–934, 2009.
[44] Fang Liu and Sule Ozev. Statistical test development for analog circuits
under high process variations. IEEE Transactions on Computer-Aided
Design of Integrated Circuits and Systems (TCAD), 26(8):1465–1477,
2007.
[45] Donald Thomas and Philip Moorby. The Verilog® hardware description
language. Springer Science & Business Media, 2008.
[46] Daya Guo, Qihao Zhu, Dejian Yang, Zhenda Xie, Kai Dong, Wentao
Zhang, Guanting Chen, Xiao Bi, Y Wu, YK Li, et al. Deepseek-coder:
When the large language model meets programming–the rise of code
intelligence. arXiv, 2024.
[47] Andre Vladimirescu. The SPICE book. John Wiley & Sons, Inc., 1994.
[48] Hanrui Wang, Kuan Wang, Jiacheng Yang, Linxiao Shen, Nan Sun, Hae-
Seung Lee, and Song Han. Gcn-rl circuit designer: Transferable transistor
sizing with graph neural networks and reinforcement learning. In Design
Automation Conference (DAC), pages 1–6. IEEE, 2020.
[49] Keertana Settaluri, Ameer Haj-Ali, Qijing Huang, Kourosh
Hakhamaneshi, and Borivoje Nikolic. Autockt: Deep reinforcement
learning of analog circuit designs. In Design, Automation & Test in
Europe Conference & Exhibition (DATE), 2020.
[50] Ahmet F Budak, Prateek Bhansali, Bo Liu, Nan Sun, David Z Pan, and
Chandramouli V Kashyap. Dnn-opt: An rl inspired optimization for
analog circuit sizing using deep neural networks. In Design Automation
Conference (DAC), 2021.
[51] Ahmet F Budak, David Smart, Brian Swahn, and David Z Pan. Apostle:
Asynchronously parallel optimization for sizing analog transistors using
dnn learning. In Asia and South Pacific Design Automation Conference
(ASP-DAC), 2023.
[52] Minjeong Choi, Youngchang Choi, Kyongsu Lee, and Seokhyeong Kang.
Reinforcement learning-based analog circuit optimizer using g m/i d for
sizing. In Design Automation Conference (DAC). IEEE, 2023.
[53] Yaguang Li, Yishuang Lin, Meghna Madhusudan, Arvind Sharma,
Sachin Sapatnekar, Ramesh Harjani, and Jiang Hu. A circuit attention
network-based actor-critic learning approach to robust analog transistor
sizing. In Workshop on Machine Learning for CAD (MLCAD), 2021.
[54] Youngmin Oh, Doyun Kim, Yoon Hyeok Lee, and Bosun Hwang.
Cronus: Circuit rapid optimization with neural simulator. In Design,
Automation & Test in Europe Conference & Exhibition (DATE), 2024.
[55] Souradip Poddar, Youngmin Oh, Yao Lai, Hanqing Zhu, Bosun Hwang,
and David Z Pan. INSIGHT: A Universal Neural Simulator Framework
for Analog Circuits with Autoregressive Transformers. In Design
Automation Conference (DAC), 2025.
[56] Bo Liu, Yan Wang, Zhiping Yu, Leibo Liu, Miao Li, Zheng Wang, Jing
Lu, and Francisco V Fernández. Analog circuit optimization system
based on hybrid evolutionary algorithms. Integration, 42(2):137–148,
2009.
[57] Wenjian Yu and Xiren Wang. Advanced field-solver techniques for RC
extraction of integrated circuits. Springer, 2014.
[58] Trent McConaghy, Pieter Palmers, Georges Gielen, and Michiel Steyaert.
Simultaneous multi-topology multi-objective sizing across thousands of
analog circuit topologies. In Design Automation Conference (DAC),
2007.
[59] Pieter Palmers, Trent McConnaghy, Michiel Steyaert, and Georges
Gielen. Massively multi-topology sizing of analog integrated circuits. In
Design, Automation & Test in Europe Conference & Exhibition (DATE).
IEEE, 2009.
[60] Petar Veselinovic, Domine Leenaerts, W Van Bokhoven, Francky Leyn,
Frank Proesmans, Georges Gielen, and Willy Sansen. A flexible topology
selection program as part of an analog synthesis system. In European
Design and Test Conference, 1995.
[61] Trent McConaghy, Pieter Palmers, Georges Gielen, and Michiel Steyaert.
Automated extraction of expert knowledge in analog topology selection
and sizing. In International Conference on Computer-Aided Design
(ICCAD), 2008.
[62] Zhenxin Zhao and Lihong Zhang. An automated topology synthesis
framework for analog integrated circuits. IEEE Trans. Comput.-Aided
Des. Integr. Circuits Syst. (TCAD), 39(12):4325–4337, 2020.
[63] Prabir C Maulik, L Richard Carley, and Rob A Rutenbar. Integer
programming based topology selection of cell-level analog circuits. IEEE
Trans. Comput.-Aided Des. Integr. Circuits Syst., 14(4):401–412, 1995.
[64] Jialin Lu, Yijie Li, Fan Yang, Li Shang, and Xuan Zeng. High-level
topology synthesis method for δ-σ modulators via bi-level bayesian
optimization. IEEE Transactions on Circuits and Systems II: Express
Briefs, 70(12):4389–4393, 2023.
[65] Souradip Poddar, Ahmet Budak, Linran Zhao, Chen-Hao Hsu, Supriyo
Maji, Keren Zhu, Yaoyao Jia, and David Z Pan. A Data-Driven Analog
Circuit Synthesizer with Automatic Topology Selection and Sizing. In
Design, Automation & Test in Europe Conference & Exhibition (DATE),
2024.
[66] Pravallika Abbineni, Saoud Aldowaish, Colin Liechty, Soroosh Noorzad,
Ali Ghazizadeh, and Morteza Fayazi. Muallm: A multimodal large
language model agent for circuit design assistance with hybrid contextual
retrieval-augmented generation. arXiv, 2025.
[67] Chenchen Zhao, Zhengyuan Shi, Xiangyu Wen, Chengjie Liu, Yi Liu,
Yunhao Zhou, Yuxiang Zhao, Hefei Feng, Yinan Zhu, Gwok-Waa Wan,
et al. Mmcircuiteval: A comprehensive multimodal circuit-focused
benchmark for evaluating llms. arXiv preprint arXiv:2507.19525, 2025.
[68] Yichen Shi, Ze Zhang, Hongyang Wang, Zhuofu Tao, Zhongyi Li,
Bingyu Chen, Yaxin Wang, Zhiping Yu, Ting-Jung Lin, and Lei He.
Amsbench: A comprehensive benchmark for evaluating mllm capabilities
in ams circuits. arXiv preprint arXiv:2505.24138, 2025.
[69] Zihao Chen, Ji Zhuang, Jinyi Shen, Xiaoyue Ke, Xinyi Yang, Mingjie
Zhou, Zhuoyao Du, Xu Yan, Zhouyang Wu, Zhenyu Xu, et al.
Analogseeker: An open-source foundation language model for analog
circuit design. arXiv, 2025.
[70] Hao Chen, Mingjie Liu, Biying Xu, Keren Zhu, Xiyuan Tang, Shaolan
Li, Yibo Lin, Nan Sun, and David Z Pan. Magical: An open-source fully
automated analog ic layout system from netlist to gdsii. IEEE Design
& Test, 2020.
[71] Guojin Chen, Keren Zhu, Seunggeun Kim, Hanqing Zhu, Yao Lai, Bei
Yu, and David Z Pan. Llm-enhanced bayesian optimization for efficient
analog layout constraint generation. arXiv, 2024.
[72] Prasanth Mangalagiri, Lynn Qian, Farrukh Zafar, Praveen Mosalikanti,
Phoebe Chang, Arun Kurian, and Vinay Saripalli. Cdls: Constraint
driven generative ai framework for analog layout synthesis. In Design
Automation Conference (DAC), 2024.
[73] Ali Hammoud, Chetanya Goyal, Sakib Pathen, Arlene Dai, Anhang Li,
Gregory Kielian, and Mehdi Saligane. Human language to analog layout
using glayout layout automation framework. In International Symposium
on Machine Learning for CAD (MLCAD), 2024.
[74] Zhenxin Zhao and Lihong Zhang. Deep reinforcement learning for
analog circuit structure synthesis. In Design, Automation & Test in
Europe Conference & Exhibition (DATE), 2022.
[75] Zhao, Zhenxin and Zhang, Lihong. Graph-grammar-based analog circuit
topology synthesis. In IEEE International Symposium on Circuits and
Systems (ISCAS), pages 1–5, 2019.
[76] Fabrice Salvaire. Pyspice, 2021.
[77] Qinkai Zheng, Xiao Xia, Xu Zou, Yuxiao Dong, Shan Wang, Yufei Xue,
Lei Shen, Zihan Wang, Andi Wang, Yang Li, et al. Codegeex: A pre-
trained model for code generation with multilingual benchmarking on
humaneval-x. In ACM SIGKDD Conference on Knowledge Discovery
and Data Mining (SIGKDD), pages 5673–5684, 2023.
[78] Qingxiu Dong, Lei Li, Damai Dai, Ce Zheng, Zhiyong Wu, Baobao
Chang, Xu Sun, Jingjing Xu, and Zhifang Sui. A survey on in-context
learning. arXiv, 2022.
[79] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia,
Ed Chi, Quoc V Le, Denny Zhou, et al. Chain-of-thought prompting elic-
its reasoning in large language models. Advances in neural information
processing systems (NeurIPS), 35:24824–24837, 2022.
[80] Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao,
Yuke Zhu, Linxi Fan, and Anima Anandkumar. Voyager: An open-ended
embodied agent with large language models. In Intrinsically-Motivated
and Open-Ended Learning Workshop@ NeurIPS2023, 2023.
[81] Yujia Qin, Shihao Liang, Yining Ye, Kunlun Zhu, Lan Yan, Yaxi Lu,
Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, et al. Toolllm:
Facilitating large language models to master 16000+ real-world apis.
arXiv, 2023.
[82] Takuya Akiba, Shotaro Sano, Toshihiko Yanase, Takeru Ohta, and
Masanori Koyama. Optuna: A next-generation hyperparameter opti-
mization framework. In ACM SIGKDD international conference on
knowledge discovery & data mining (SIGKDD), pages 2623–2631, 2019.
13

