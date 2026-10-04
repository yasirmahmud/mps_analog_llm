---
title: "Evaluating LLM-based Workflows for Switched-Mode Power Supply Design"
authors: "Nau, Simon; Krummenauer, Jan; Zimmermann, André"
date: "2025/07/14"
category: "topology_schematic"
source_url: "https://arxiv.org/abs/2507.10639"
pdf: "papers/2025_evaluating-llm-based-workflows-for-switched-mode-power-supply-design_2507-10639.pdf"
sha256: "99a39d28b6aae991af47de3b5b1cfeb62a63e682f4229c65730aa2d1bc8e698a"
pages_extracted: 16
---

# Evaluating LLM-based Workflows for Switched-Mode Power Supply Design

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

Evaluating LLM-based Workflows for Switched-Mode Power
Supply Design
Simon Nau1
, Jan Krummenauer2
, and André Zimmermann1,3
1
Robert Bosch GmbH, Cross-Domain Computing Solutions, Leonberg, Germany
2
University of Stuttgart, Institute for Micro Integration (IFM), Stuttgart, Germany
3
Hahn-Schickard, Stuttgart, Germany
Simon.Nau@de.bosch.com
Abstract
Large language models (LLMs) have great potential to enhance productivity in many dis-
ciplines, such as software engineering. However, it is unclear to what extent they can assist
in the design process of electronic circuits. This paper focuses on the application of LLMs to
switched-mode power supply (SMPS) design for printed circuit boards (PCBs). We present multi-
ple LLM-based workflows that combine reasoning, retrieval-augmented generation (RAG), and a
custom toolkit that enables the LLM to interact with SPICE simulations to estimate the impact
of circuit modifications. Two benchmark experiments are presented to analyze the performance
of LLM-based assistants for different design tasks, including parameter tuning, topology adaption
and optimization of SMPS circuits. Experiment results show that SPICE simulation feedback
and current LLM advancements, such as reasoning, significantly increase the solve rate on 269
manually created benchmark tasks from 15% to 91%. Furthermore, our analysis reveals that most
parameter tuning design tasks can be solved, while limits remain for certain topology adaption
tasks. Our experiments offer insights for improving current concepts, for example by adapting
text-based circuit representations.
1 Introduction
Large language models (LLMs) have achieved remarkable results in a broad field of applications.
They revolutionize software development by automating significant parts of the development process,
increasing speed and efficiency. Previous research aims to translate this success into the field of elec-
tronic design automation (EDA) [2, 3, 26, 27, 15, 28, 20, 5, 16, 29, 11, 14, 13, 10, 8, 22, 23, 1], with
a main focus on the development of integrated circuits (ICs). Much of this work centers on digital
ICs, by applying LLMs for hardware description language (HDL) code generation, particularly using
Verilog [2, 3, 26, 27, 15, 28, 20, 5, 16]. Also for analog and mixed-signal IC design, several LLM-based
workflows have been proposed: In [11], an LLM is provided with a custom SPICE interface to obtain
simulation results, while other studies present workflows combining LLMs with custom circuit specific
knowledge libraries [14, 13, 11, 8] or with conventional optimization algorithms [14, 13, 10, 8]. Fur-
thermore, benchmarks have been established [22, 23, 1] to evaluate LLM capabilities in analysis and
design of analog and mixed-signal circuits.
While prior work has focused primarily on the development of ICs, this paper addresses the design
of PCB modules based on switched-mode power supply (SMPS) circuits. PCB module design poses
unique challenges for applying LLMs. First, instead of being able to select component size parameters
freely, PCB design requires the use of given catalogue-devices in specific packages, such as micro-
controllers, power regulators, and passive components. A resulting concern is managing integration
complexity, which involves numerous component pins, multiple voltage domains, diverse interfaces, and
heterogeneous parts from many vendors. Secondly, in contrast to IC design, PCB designs are highly
dependent on exact specifications and constraints from component datasheets and specific application
1
arXiv:2507.10639v2
[cs.AR]
7
Nov
2025

## Page 2

notes. In addition, public data sources often lack critical information or access rights to certain docu-
ments can be restricted. As a result, it must be assumed that specific component information is often
not part of an LLM’s training dataset.
Furthermore, employing LLMs in circuit design faces other challenges. First, LLMs have a limited
ability to interpret results from simulation tools like SPICE [14], which are essential in electronic
development processes. As we demonstrate in Section 2.2, a multimodal LLM (GPT-4o) cannot re-
liably extract information from simulated time-series signals. Second, many tasks require a complex,
multi-step design process that involves information retrieval from datasheets, netlist analysis, netlist
adaptation, solving equations, and integrating simulation feedback. Figure 1 summarizes a reasoning
chain required for an exemplary SMPS design task.
Determine relevant
component: inductor 𝐿
Read formula for current ripple from
datasheet: Δ𝐼𝐿 =
𝑉out
𝐿⋅𝑓s𝑤
(1 −
𝑉out
𝑉in
)
Rearrange
formula
Find missing parameters
for formula: 𝑓sw, 𝑉in, 𝑉out
Read 𝑓sw from datasheet: 2.25 MHz
Read formula for output voltage
from datasheet 𝑉out = 0.6V(1 +
𝑅top
𝑅bot
)
Calculate
𝑉out: 2.5 V
Verify results with LTSpice
simulation. If required,
refine component value
Adapt
netlist
Read 𝑅top, 𝑅bot from given
netlist: 187kΩ, 59kΩ
Read 𝑉in from given netlist: 5.5V
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
LTC3419
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
+ −
× ÷
Calculate
𝐿: 6.1 µH
+ −
× ÷
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
LTC3419
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
LTC3419
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
Adjust the given netlist,
such that the current ripple
has the value 100 mA
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
LTC3419
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
Adjusted
netlist
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
LTC3419
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
Figure 1: Required reasoning chain to solve the following exemplary SMPS design task: ”Adjust
the given netlist, such that the current ripple has the value 100 mA”. The netlist and complete input
prompt can be seen in Figure 6. The controller’s datasheet and SPICE simulations are used as external
information sources.
∆IL: ripple current, fs: switching frequency, Vout: output voltage, Vin: input voltage, Rtop and Rbot:
resistors in feedback path
In this paper, we introduce different LLM-based workflows to address the described challenges and
define a benchmark to investigate their effectiveness and limitations for switched-mode power supply
(SMPS) circuit design.
To construct the LLM-based workflow, we develop a broad selection of domain-specific tools that serve
as an interface between the LLM and the SPICE simulator. These functions reliably extract different
features from the simulated signals and provide them as text-based feedback to the LLM, e.g. the peak-
to-peak value of the output voltage ripple. The LLM conducts multiple iterations with the simulator
and observes the effects of circuit modifications. Additionally, to enhance the LLM’s circuit-specific
domain knowledge, it is enabled to interact with the corresponding datasheet via retrieval-augmented
generation (RAG) [12]. Moreover, a state-of-the-art reasoning LLM is compared with a predecessor
LLM that lacks reasoning capabilities.
To evaluate the LLM-based workflows, we conduct two experiments. The first experiment focuses on
finding idealized component values, which is often an initial step in a design process. In the second
experiment, the goal is to select an optimal set of catalogue components for circuit optimization.
For the first experiment, we create a comprehensive benchmark of 269 design tasks related to SMPS
circuits, which are represented as SPICE netlists. Each task requires adapting a given circuit, see
Figure 4, to meet specific requirements. The benchmark covers circuits of varying complexity and
different design task categories, such as topology adaption and parameter tuning. To test the LLMs’
capability to use simulation feedback, the benchmark contains tasks that explicitly require this feed-
back to compensate for imprecise datasheet equations or specifications. Furthermore, we investigate
the abilities of the LLMs to solve complex multi-step SMPS design tasks, by varying the length and
complexity of the required reasoning chain, see example reasoning chain in Figure 1.
In the second experiment, we test the ability of an LLM-based workflow to select components from a
given set of catalogue components, to optimize the efficiency of an SMPS circuit in a specific operation
mode.
In a comprehensive evaluation of the experiment results, the isolated contributions of the individual
elements of the LLM-based workflow are presented and the performance over different design task
categories is analyzed. In addition, we discuss the computational cost and limitations of the proposed
2

## Page 3

workflows to identify possible improvements, such as adapting the text-based representations of the
SMPS circuits.
To summarize, this paper makes the following key contributions:
• We present different LLM-based workflows for circuit design, which integrate a custom toolkit
for interpreting SPICE simulation feedback, RAG for retrieving information from datasheets,
and reasoning.
• In two comprehensive experiments, including a new benchmark with 269 design tasks, we eval-
uate capabilities and limitations of these workflows to assist in SMPS circuit design for PCB
applications.
2 Background
2.1 Modern Design of Switched-Mode Power Supplies
Switched-mode power supplies (SMPS), such as the common Buck, Boost, and Buck-Boost topologies
[7], convert voltage levels with high efficiency. In a typical buck converter, a pulse-width modulation
(PWM) controller rapidly switches a transistor to create a pulse train. This is subsequently filtered by
an inductor and capacitor, see Figure 4a, to produce a DC output voltage whose level is determined
by the transistor’s duty cycle.
Modern buck converters consist of dedicated ICs with advanced features. They enable different opera-
tion modes, such as burst and pulse-skipping mode, to achieve very high efficiencies and support spread
spectrum operation to lower peak electromagnetic interference (EMI). Other advanced functionalities
include adjustable startup times, adjustable switching frequencies, programmable thresholds for over-
voltage (OVP) and overcurrent protection(OCP), and thermal shutdown. Buck converter topologies
can also be extended to a multi-phase converters, where the ripple components partially cancel out
each other due to phase shift, which reduces the output ripple.
Typical challenges in SMPS circuit design include component selection and sizing, topology design,
efficiency and stability considerations, while other important aspects like physical layout and thermal
management are outside the scope of this work.
2.2 LLMs and Time-Series Input
Table 1 provides the results of a case study, in which we examine the ability of a multimodal LLM
(GPT-4o) to interpret time-series data. Specifically, GPT-4o is tasked with identifying the ripple peak-
to-peak value from either a raw numeric vector or its corresponding image representation. An answer
is considered correct when the LLM’s reading falls within a 10% tolerance range of the ground-truth
value.
Table 1: Case study: Investigating the ability of the multimodal LLM GPT-4o to handle time series
data, examined through the task of reading the ripple in time series provided as numeric vectors or
images
Test
Case
Correct
Answer
Vector
Length
GPT-4o
(Vector)
GPT-4o
(Image)
1 426 µV 1250 350 µV ✗ 0.8 mV ✗
2 673 µV 2040 605 µV ✓ 1.1 mV ✗
3 17 mV 1600 113 mV ✗ 36 mV ✗
4 14.5 mV 650 83.4 mV ✗ 40 mV ✗
5 24.1 mV 950 24.3 mV ✓ 24 mV ✓
The results in table 1 show, that it is not a reliable option to provide the time series as numeric
vectors or images directly to the LLM for interpretation and further processing. While GPT-4o, for
example, fails to extract the correct ripple from the signal in Figure 2a, it succeeds with the test case
depicted in Figure 2b. The failure in Example 2a may be due to the underlying trend in the ripple
3

## Page 4

signal. While future multimodal LLMs might provide more precise answers, we must currently rely on
rule-based extraction functions presented in the next section.
(a) Test case 1: Output voltage of buck converter with
ripple
(b) Test case 5: Output voltage of buck converter with
ripple
Figure 2: Example SPICE simulation signals from the case study
2.3 Reasoning
In this work, we evaluate two LLM models: OpenAI GPT-4o (released in 2024) and the newer LLM
OpenAI o3 (released 2025). The key difference between the two models lies in their so-called reasoning
capabilities. Non-reasoning LLMs, like GPT-4o, generate answers directly based on pattern recognition
and surface-level correlations in text. Reasoning LLMs additionally have the ability to simulate or
represent internal deliberations before giving their final answer.
Reasoning techniques include chain-of-thought (CoT) prompting [31], which instructs the LLM with
examples how to generate intermediate reasoning steps before answering. Tree-of-thoughts (ToT)
[33] expands on CoT by exploring multiple reasoning branches. Another reasoning technique, self-
consistency decoding [30], runs multiple reasoning chains and selects the most consistent answer.
Reasoning may also include an augmentation of LLMs with external tools, prompt rewriting, and
further self-verification mechanisms.
3 Methodology
3.1 LLM-assisted Simulation Workflows
Figure 3 introduces the structure of the LLM-based workflows discussed in this paper. The model
receives the netlist of a typical application circuit from the SMPS controller’s datasheet as input. In
most cases these initial circuits do not meet the desired specifications. Therefore, the main goal is to
modify the netlist to align with specific input requirements given by design engineers.
In the setup shown in Figure 3, the standalone LLM serves as baseline, depicted in gray, which
directly outputs a modified netlist. To optimize the LLMs performance, we employ prompt engineering,
providing a tailored system prompt. For instance, the LLM agent is instructed to always include the
adapted netlist in its responses.
The first extension, depicted in orange, enables the LLM to use RAG to retrieve specific information
from the corresponding SMPS controller datasheet. The RAG system is accessed via an API call
and can perform both keyword and semantic searches in vector stores and running multiple queries
in parallel. The queries are generated by the LLM based on its input and system prompt. The LLM
autonomously defines the number of needed RAG calls.
The second workflow extension, highlighted in blue in Figure 3, allows the LLM agent to receive
simulation feedback on the circuit’s behavior. This feedback enables the LLM to estimate the impact
of netlist modifications and verify suggested adaptations, thereby enhancing response reliability. This
interaction is enabled by a set of reading tools (Python functions) that serve as an interface between
the LLM and the SPICE simulation. As noted in Section 2.2, multimodal LLMs like GPT-4o cannot
reliably extract features, such as output ripple, directly from simulation signals. To overcome this
limitation, a comprehensive set of tools is developed to extract these features and return them to the
LLM in a text-based format. The LLM is provided with a list of JSON files, each documenting a tool’s
4

## Page 5

Adapted
SPICE
Netlist
SPICE Netlist
of Typical
Application
Engineer‘s
Requirement
Datasheet
RAG
Tool Choice
Reading Tools
LTspice Simulation
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
IN N001 IN N003 0
LTC3419
R2 OUT1 N004 59K
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end Chosen Tool
Initial Prompt
Input
LLM-based Workflow
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
IN N001 IN N003 0
LTC3419
R2 OUT1 N004 59K
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
Output
Feedback
Extracted Feature
V1 IN 0 4.2
C3 OUT2 0 10µ
Rload1 OUT1 0 1200
XU1 N004 IN 0 N002
IN N001 IN N003 0
LTC3419
R2 OUT1 N004 59K
R3 N004 0 59K
C2 OUT1 N004 22p
.tran 9m
.end
LLM
Role: You are an expert Electrical Engineer
specializing in circuit design and simulation
using SPICE. Think step-by-step:
Analyze Request: Carefully understand …
Research Datasheets: Access datasheets …
Modify Netlist: Create an updated version…
Verify with Simulation: Use the provided
SPICE simulation tools to test the netlist…
SPICE Interaction Datasheet Interaction Baseline Workflow
System Prompt
Netlist
Suggestion
Keyword search
Semantic search
Query 1
Query 2
Query 3
…
Figure 3: Overview of the LLM-based workflows: The baseline workflow uses an instruction-tuned
LLM (GPT-4o or o3) that receives a reference circuit and requirements as input to directly generate
an adapted circuit netlist. The first extension enhances this workflow with information retrieval from
datasheets using RAG. The second extension equips the LLM-based workflow with a comprehensive
set of feature extraction tools to receive simulation feedback
functionality, name, and required arguments. Based on this information, the LLM autonomously
decides which tool to use. Similarly it determines the number of API tool calls, depending on the
specific benchmark question and the history of previous tool feedback. Examples of these tools include:
• get mean output voltage()
Calculates the mean over a time interval in the steady state of the output voltage.
• get ripple()
Extracts the peak-to-peak value of a periodic signal in the steady state, as discussed in Section
2.2.
• get switching frequency()
First, it performs a Fourier transform of the signal within a steady-state time interval, then
extracts the fundamental frequency by identifying the highest peak in the spectrum.
• get startup time()
Calculates the mean value of the output voltage over a time window in steady state and identifies
the first time the signal reaches 90% of the steady state value.
3.2 Benchmarking
We developed a benchmark to evaluate the ability of LLM-based workflows to assist in the circuit
design process of SMPS. The benchmark tasks are based on SMPS example circuits, featuring three
circuit types with increasing levels of complexity. The most simple circuit is a textbook buck converter,
illustrated in Figure 4a. The medium level is a buck converter using the LTC3419, a dual step-down
5

## Page 6

regulator [17], as shown in Figure 4b. The most complex example is the typical application circuit of
the LTC7802, a 2-phase synchronous step-down controller with advanced features like spread spectrum
(a) Textbook buck converter circuit in LTSpice
(easy)
(b) Typical LTC3419 application circuit in LTSpice,
after [17] (medium)
(c) Typical LTC7802 application circuit in LTSpice, after [18] (hard)
Figure 4: The three SMPS circuit types that serve as the basis for the benchmark, shown in order of
increasing difficulty
operation or a programmable switching frequency, depicted in Figure 4c.
Overall, the benchmark contains 269 test questions: 72 for the textbook buck converter (easy), 72
for the LTC3419 (medium), and 125 for the LTC7802 (hard). The benchmark can be divided into
two design task categories: parameter tuning and topology adaption. The parameter tuning tasks
can further be subdivided according to the specific parameter being adjusted: supply voltage Vin,
output voltage Vout, current ripple ∆IL, output voltage ripple ∆Vout, and for the LTC7802 only, the
switching frequency fsw and startup time tstart. Figure 5 illustrates required circuit modifications and
the resulting effects in the output voltage and inductor current for each design task category.
Solving the Vout-, ∆IL- and ∆Vout- tuning tasks requires the LLM to process a reasoning chain
whose complexity and length increase from task to task. To explain this, we briefly describe the
necessary steps to solve these tasks.
To tune the output voltage Vout in LTC3419 or LTC7802 circuits, the following equation
Vout = Vref

1 +
Rtop
Rbot

(1)
and the corresponding reference voltage Vref must be retrieved from the datasheet, rearranged and
calculated to receive the values for Rtop or Rbot. These feedback resistors must be identified in the
netlist and adapted accordingly. Additional factors, such as ensuring sufficient supply voltage, must
6

## Page 7

𝑉out
𝑡start
Δ𝑉out
1/𝑓sw
.01µ
C5
topology adaption task
C3 C11
100µ
X2
330µ
Δ𝐼L
1.8µ
L1
𝐼L
𝑡
𝑡
𝐼L
68.1K
R3
357K
R2
𝑉OUT2
𝑉OUT2
Freq
INTVcc
(a) (b)
Figure 5: Illustration of a topology adaption task and the different parameter tuning tasks - Vout, ∆IL,
∆Vout, fsw, tstart. For the different task categories affected elements in the circuit excerpt (a) and in
the simulation signal segments (b) are marked in the same color.
be considered.
For current ripple ∆IL adjustment tasks, the following equation
∆IL =
1
fswL
Vout

1 −
Vout
Vin

(2)
must be retrieved, rearranged, and used to calculate the required value for the inductor L. To obtain
the missing parameters for Equation 2, all steps from the previous example must be repeated to get
the output voltage. Additionally, input voltage and switching frequency must be identified from the
datasheet and the circuit configuration in the netlist.
For solving output voltage ripple ∆Vout tasks, first Equation (1) and (2) must be solved again and the
corresponding results used to determine the required ESR with the equation
∆Vout ≈ ∆IL

ESR +
1
8fswCout

. (3)
Typically, for parameter tuning tasks, the LLM assistant must perform several key steps:
• Solve equations
• Retrieve information from datasheet and circuit configuration
• Identify and adapt the relevant components in the netlist
The equations to determine ∆Vout and tstart are only approximations. Consequently, these tasks specif-
ically test the LLM’s ability to integrate simulation feedback.
The second category, topology adaption, includes tasks such as inserting key components into the
converter topology (for example diode, inductor and MOSFETs), adjusting controller channel config-
urations, enabling or disabling spread spectrum operation, or selecting a specific operating mode. For
example, to enable pulse skipping mode for the LTC7802 controller, the peripheral circuit must be
adjusted by connecting the MODE pin to the INTVcc pin through a high-impedance resistor. The
LLM can collect this information by accessing the datasheet via RAG. It would then identify these
two pins in the netlist and modify the circuit accordingly.
The 269 benchmark questions were created by combining different design tasks, varying component
values and the three circuit types. Further data diversification is achieved by varying the appear-
ance of the netlist by deliberately changing node and component names and applying minor topology
7

## Page 8

Target value
You have the following SPICE netlist:
V1 IN 0 5.5
Cin IN 0 10µ
C3 OUT2 0 10µ Rser=0.05
L2 N001 OUT2 3.3µ
Rload2 OUT2 0 3
R5 OUT2 N003 118K
R6 N003 0 59K
C4 OUT2 N003 22p
C1 OUT1 0 10µ Rser=0.05
L1 N002 OUT1 3.3µ
Rload1 OUT1 0 4.16
XU1 N004 IN 0 N002 IN N001 IN N003 0 LTC3419
R2 OUT1 N004 187K
R3 N004 0 59K
C2 OUT1 N004 22p
Please adjust the netlist in order to adapt the ripple current
on OUT1. The ripple current should have the following
value: 100mA.
The allowed tolerance is 5 mA.
SPICE Netlist
of Example
Circuit
Design Task
Tolerance
Figure 6: Exemplary benchmark input
prompt
Output
Netlist
• Netlist
• Design task
• Target value
• Tolerance
• Reading tool
LLM Workflow
Benchmark.json Circuit Simulation &
Feature Extraction Evaluation
For all benchmark questions
Figure 7: The automatic benchmarking involves sim-
ulating the circuit received by the LLM workflow, ex-
tracting the relevant design feature from the simula-
tion using a reading tool, and evaluating it against the
target value within the specified tolerance
changes. Moreover, integrating the design tasks with different target parameters, such as setting the
ripple current to different levels, increases the number of benchmark questions. A complete exemplary
benchmark prompt, using a medium-difficulty LTC3419 netlist as input for the LLM-based workflows
is shown in Figure 6.
The benchmarking process is automated, as visualized in Figure 7, where the same reading tools can
be applied in the LLM-based workflow and the evaluation.
In our first experiment, the LLM is tasked to find ideal component values, which can be used sub-
sequently to find suitable commercially available parts. The second experiment, described in Section
4.4, evaluates the LLM workflow’s performance in selecting real catalogue-components.
4 Experiment Results
4.1 Experimental Setup
The evaluated LLMs, OpenAI GPT-4o and o3, are accessed via the Azure OpenAI REST API [21]
(version 2025-04-01-preview) using the responses and assistant interface. The datasheet of the corre-
sponding controller serves as database for the RAG system. The RAG process is triggered via API
tool calls by the LLM and executes keyword as well as semantic searches in vector stores. For the vec-
tor embedding the text-embedding-3-large model [25] with 256 dimensions is applied. The maximum
number of chunks added as context to the LLM is 20, with each chunk having the size of 800 tokens.
Circuit simulations are performed using the simulator LTSpice [19]. All reported API call results
were generated between March and September 2025.
4.2 Evaluation Metrics
As evaluation metric the solve rate is used, which represents the percentage of design tasks from the
benchmark that are correctly completed. A question is considered to be correctly answered if the
target value falls within a specified tolerance range, such as a ripple of 18 mV ± 0.9 mV. Generally,
the tolerance is set at 5%, except for topology adaption questions, such as ”Adjust the netlist to select
the pulse skipping mode.”, which can only be answered right or wrong.
As supplementary continuous metric we use the absolute percentage error APE = Ai−Fi
Ai
· 100%,
where Ai represents the actual value and Fi the value provided by the LLM agent. Due to the
presence of significant outliers in the APE we use the median of the APE to avoid distortion. For the
median APE, topology adaption questions are excluded since they can only be marked as correct or
incorrect.
8

## Page 9

To receive a better statistical estimation of the performance and to determine what constitutes a
statistically significant change in the performance metrics, we run the entire benchmarking process n
times, with n = 40. This allows us to calculate the sample mean of the evaluation metrics and the
corresponding confidence interval [6] with a significance level of α = 0.05. Due to limitations concerning
the simulation runtime, this statistical evaluation is only performed for benchmark questions targeted
on the simple circuit type of the textbook buck converter.
4.3 Experiment 1: Parameter Tuning and Topology Adaption with Ideal-
ized Components
The benchmarking is conducted for the following workflow configurations: GPT-4o and o3 standalone,
combined with datasheet interaction ( + RAG), with simulation feedback ( + SPICE) and both. The
benchmarking results are presented in Table 2. The boxplot in Figure 8 illustrates the distribution of
the APE across the entire benchmark, using the reasoning LLM o3 for all four tested approaches.
Table 2 shows that the standalone LLMs achieve total solve rates of 14.4% (LLM GPT-4o) and 25.4%
Table 2: Experiment 1 - Benchmarking results: The performance is shown for design tasks related
to the three different circuit types and the total benchmark, using the solve rate and median(APE)
as performance metrics. The methods assessed include GPT-4o and o3 agents, combinations with
datasheet information retrieval via RAG and feedback from SPICE simulations
Textbook Buck
Converter (easy)
LTC3419 (medium) LTC7802 (hard) Total
Method Solve Rate med(APE) Solve Rate med(APE) Solve Rate med(APE) Solve Rate med(APE)
GPT-4o 29.0 ± 1.0* 55.3 ± 3.1* 12.5 56.3 7.1 75.2 14.4 64.8
o3** 44.9 ± 1.2* 13.8 ± 1.0* 18.1 32.8 18.4 31.7 25.4 27.2
GPT-4o + RAG 25.4 ± 1.1* 63.2 ± 2.7* 18.1 55.0 9.6 59.6 16.1 59.3
o3 + RAG** 58.0 ± 1.0* 3.1 ± 0.2* 44.4 12.0 36.0 8.1 44.1 7.8
GPT-4o + SPICE 64.4 ± 1.5* 3.7 ± 0.2* 48.6 4.7 35.4 7.2 46.4 5.6
o3 + SPICE** 96.5 ± 0.6* 1.6 ± 0.1 95.8 0.8 72.0 1.4 84.9 1.3
GPT-4o + RAG + SPICE 64.9 ± 1.8* 3.6 ± 0.3* 50.0 4.6 42.4 4.6 48.7 4.4
o3 + RAG + SPICE** 97.2 ± 1.3* 1.4 ± 0.2* 97.2 0.9 83.2 1.7 90.7 1.4
*The whole benchmarking is performed n=40 times to receive confidence intervals, **Agents using reasoning
(reasoning LLM o3), respectively. In the workflow combined with RAG, the total solve rate increases
to 16.1% (LLM GPT-4o) and 44.1% (reasoning LLM o3), respectively. When simulation feedback is
integrated into the workflow, GPT-4o solves 46.4% and o3 84.9% of all benchmarking tasks. When
using both SPICE and RAG, the performance of GPT-4o increases to 48.7%. The best solve rate of 91%
is achieved by the o3 reasoning model combined with SPICE feedback and RAG. From these results,
we can conclude that the two main factors improving performance are SPICE simulation feedback and
reasoning capabilities of the LLM.
4.3.1 SPICE simulation feedback:
Table 2 shows that the total solve rate of the standalone LLMs significantly increases through integrat-
ing SPICE feedback (LLM vs. LLM + SPICE) by 32% (LLM GPT-4o) and 59.5% (reasoning LLM
o3). The simulation feedback improves the solve rate for all circuit complexities. Similarly, in Figure
8 it can be seen in the boxplot that SPICE feedback reduces the mean and variance of the APE.
This shows that the LLM effectively uses the offered simulation tools.
The simulation feedback increases the solve rate in all cases because it enables the LLM to verify its
choices and correct previous mistakes. A common error are wrong assumptions during formula solving.
For instance, using an incorrect Cout for voltage ripple calculation (Equation (3)) by failing to recognize
all output capacitors from the netlist, or retrieving the wrong reference voltage from the datasheet for
output voltage adjustments (Equation (1)). Such errors can be observed in the simulation response
and iteratively fixed by the LLM .
Another possible source of mistakes is that the LLM agent fails to follow its instructions. SPICE
feedback ensures it follows instructions more closely, minimizing errors such as returning incomplete
netlists.
In some cases, the LLM lacks critical information. With simulation feedback, it can still achieve good
results through initial estimates and iterative refinement, demonstrated by the performance for the o3
+ SPICE workflow without datasheet access.
9

## Page 10

o3 o3+ RAG o3
+ SPICE
o3 + RAG
+ SPICE
0
20
40
60
80
100
APE
(%)
Figure 8: The distribution of the APE on
the total benchmark for the four tested ap-
proaches: o3, o3 + RAG, o3 + SPICE and o3
+ RAG + SPICE. The central box depicts the
range from the first quartile (Q1) to the third
quartile (Q3), with a line marking the median
0 2 4 6 8 10 12
Number of Iterations
0.0
0.2
0.4
0.6
0.8
1.0
Performance
(Solve
Rate)
Textbook Buck Converter (easy)
LTC3419 (medium)
LTC7802 (hard)
Total
Figure 9: Performance of the LLM workflow
o3 + RAG + SPICE over the number of
tool iterations for the circuit types textbook
buck converter (easy), LTC3419 (medium),
LTC7802 (hard) and the total benchmark
In Figure 9, the interaction of the best performing workflow (o3 + RAG + SPICE) with the simulation
tools is analyzed. The performance measured by the solve rate is shown over the number of simulation
feedback iterations. We observe a rapid increase in the solve rate within the first four iterations until a
plateau is reached. This demonstrates that a small number of simulation iterations (≤4) are sufficient
to achieve a significant performance improvement.
4.3.2 Reasoning:
According to Table 2, reasoning improves the performance of the standalone LLM by 9%, with RAG
(LLM + RAG) it improves by 28%, with simulation feedback (LLM + SPICE) by 38.5% and in
combination (LLM + SPICE + RAG) by 42%.
These observations reveal that reasoning is beside simulation feedback a main factor that enhances the
performance of all workflows. Further they show that both SPICE feedback and RAG are used much
more effectively by the reasoning model.
The LLM GPT-4o tends to complete substeps of multi-step benchmark tasks insufficiently, because it
lacks reasoning capabilities. For example, it solves formulas incorrectly or does not adapt the circuit
netlist. It is also prone to making false assumptions, such as guessing a Vout of 5V without calculating
it within the task to determine the current ripple with Equation (2).
In contrast, reasoning LLMs explore a broader range of possible solution paths. For example, in a task
to find a maximum possible ESR to ensure a maximal voltage ripple, the non-reasoning model ignores
two of three cascaded output capacitors, while the o3 reasoning model correctly includes them in the
calculation of Equation (3) and the subsequent component adjustments.
4.3.3 Retrieval-Augmented Generation:
When combining RAG and the non-reasoning model GPT-4o, Table 2 shows only a slight improvement
in the total solve rate by 1.7%. Results are different for the reasoning LLM o3, where using RAG to
retrieve information from the datasheet significantly increases the total solve rate from 25.4% to 44.1%
(o3 vs. o3 + RAG) and from 84.9% to 90.7% (o3 + SPICE vs. o3 + RAG + SPICE).
This means that LLMs, which lack reasoning capabilities, are less able to integrate complex retrieved
information into their problem-solving workflow. We confirm this by an additional small experiment.
The LLM GPT-4o is evaluated on our benchmark using an ”idealized RAG” for the textbook buck
10

## Page 11

converter tasks by manually including all relevant information in the input prompt. However, these
results do not surpass those achieved with the standard RAG.
4.3.4 Increasing circuit complexity:
The performance of the tested workflows in Table 2 declines from the textbook buck converter (easy)
to the LTC3419 (medium) and further to the LTC7802 (hard). This trend can be attributed to several
possible reasons. First, the greater amount of publicly available data about the general textbook buck
converter compared to the specific controllers LTC3419 and LTC7802. Second, the LTC3419, and
especially the LTC7802, have more complex netlists with more components and nodes, an increasing
number of controller pins to configure. Therefore, these text-based circuit netlists are increasingly
difficult to comprehend and leave more possibilities for mistakes compared to the significantly shorter
netlist of the textbook buck converter.
Vout IL Vout topology
adaption
0.0
0.2
0.4
0.6
0.8
1.0
Solve
Rate
Textbook Buck Converter (easy)
Vout IL Vout topology
adaption
0
LTC3419 (medium)
Vout IL Vout fswitch tstart topology
adaption
0 0
LTC7802 (hard)
RAG RAG+SPICE GPT-4o o3 (reasoning)
Figure 10: The solve rate of several LLM-based workflows split by question category. The parameter
tuning design tasks are subdivided into categories according to the target parameters: output voltage
Vout, ripple current ∆IL, output voltage ripple ∆Vout, switching frequency fswitch and the startup time
of the controller tstart. Shown are LLM workflows using GPT-4o and the o3 reasoning model combined
with RAG for information retrieval from the corresponding datasheet and SPICE simulation feedback
4.3.5 Analysis of design task categories:
Section 3.2 introduced the design task categories of topology adaption and parameter tuning. The latter
can be further subdivided by the specific parameter being adjusted: output voltage Vout, current ripple
∆IL, output voltage ripple ∆Vout, and, for LTC7802 only, the switching frequency fsw and startup
time tstart. Figure 10 presents the solve rates of different LLM workflows, broken down by these task
categories and the three circuit types.
The best workflow - o3 (reasoning LLM) + RAG + SPICE - solves most parameter tuning tasks
achieving a solve rate of ≥ 90% for Vout, ∆IL, ∆Vout, tstart tasks and approximately 80% for fsw tasks.
For topology adaption tasks on easy and medium-level circuits, its performance is in the range 80% -
90%. In contrast, the solve rate declines for topology adaption tasks on the complex LTC7802 circuits,
where the performance drops to approximately 50%.
Refinement of analytical solutions via simulation: Figure 10 shows that most output voltage
ripple ∆Vout tuning tasks fail, if no SPICE simulation feedback is integrated into the LLM-based
workflow. This observation also applies to tasks related to adjusting the startup time in LTC7802
11

## Page 12

circuits.
One reason for this are the provided datasheet formulas for the voltage ripple (3) and the startup
time, which are only approximations. In many cases these approximations are not precise enough and
require iterative refinement through simulation.
Reasoning chain length: For the LLM-based workflows without simulation feedback, Figure 10
(orange bars) shows a trend where the solve rate declines from Vout tasks, to ∆IL tasks, and further
to ∆Vout design tasks.
An explanation for this trend are the different ”lengths of the reasoning chain” required to solve these
tasks. As explained in Section 3.2, determining the output voltage Vout requires only one equation
(1), while calculating the current ripple ∆IL requires two equations (1), (2), while the output voltage
ripple ∆Vout requires solving and correctly combining three equations (1) - (3).
…
* --- Pin-7 (RUN1)=0 V → OFF
* --- Pin-18 (RUN2)=RUN → ON
XU1 OUT1 N019 NC_01 0 0 0 RUN NC_02 N005 N003 N011 N009 OUT2 \
N014 N016 N008 N018 0 N001 NC_03 N015 RUN N010 N004 N007 \
N002 N013 N012 N017 LTC7802
…
LLM-Workflow Output
Figure 11: A segment from the LLM output on the task of deactivating the first controller channel
and activating the second. In a comment (lines starting with * ), the LLM states correctly that to
deactivate the first channel, pin 7 should be connected to ground, and to activate the second channel,
pin 18 should be connected to the RUN net. However, the LLM fails to translate this from the natural
language comment into proper SPICE netlist syntax
Topology adaption tasks: In topology adaption, Figure 10 shows that the best workflow (o3 +
RAG + SPICE) achieves a success rate of over 80% on easy and medium circuits, but its performance
drops significantly to approximately 50% on the more complex LTC7802 circuits.
Therefore, topology adaption design tasks represent a main limitation for the investigated LLM-based
workflows. We attribute this poor performance primarily to the SPICE netlist syntax, which is un-
suitable for LLMs for two reasons. First, the netlist encodes the circuit topology in a tabular format
rather than in a language-like structure, see Figure 6. Second, the semantic description of component
functionality is weak. For example, the netlist format misses designators for component pins (e.g.,
”anode,” ”cathode,” ”drain”). This causes several errors such as incorrect diode orientation or faulty
MOSFET pin connections and controller pin configurations. Figure 11 illustrates this problem: the
LLM-based workflow correctly describes a required change in a natural-language comment but then
fails to translate that instruction into the proper netlist syntax.
4.3.6 Computational Cost and Runtime
Figure 12 summarizes the average LLM token usage for the benchmarking tasks in this first experiment,
broken down by circuit type and workflow. Token usage increases with circuit complexity. The use
of RAG and SPICE simulation tools both significantly increase token count, especially over multiple
iterations. This is because RAG adds retrieved document chunks to the input prompt, while each
SPICE iteration requires creating a new netlist and reviewing past changes. Figure 13 shows the
required runtime of the LLM-based workflows. Similar to token usage, the runtime increases with
circuit complexity, the use of RAG, and especially over multiple simulation iterations. For the more
complex LTC7802 circuits, the SPICE simulation time becomes the dominant factor, accounting for
the majority of the total runtime.
12

## Page 13

Textbook Buck
Converter (easy)
LTC3419
(medium)
LTC7802
(hard)
0
10k
20k
30k
40k
50k
60k
70k
Mean
number
of
Tokens
o3
o3 + RAG
o3 + RAG + SPICE
Figure 12: Token usage of different LLM work-
flows across circuit types with varying com-
plexity
Textbook Buck
Converter (easy)
LTC3419
(medium)
LTC7802
(hard)
0
1
2
3
4
5
6
7
8
Mean
Runtime
(min)
o3
o3 + RAG
o3 + RAG + SPICE
LLM time
Simulation time
o3
o3 + RAG
o3 + RAG + SPICE
LLM time
Simulation time
Figure 13: Runtime of LLM and Simulation of
different LLM workflows across circuit types
with varying complexity
4.4 Experiment 2: Catalogue Component Selection for Efficiency Opti-
mization
The first experiment demonstrates that the best workflow (o3 + RAG + SPICE) solves most parameter
tuning tasks using idealized lumped-element components. In this second experiment, the goal is to
select the best combination of real components to optimize the efficiency of an LTC7802 circuit in
light load operation. Instead of using simplified lumped-elements as component models, manufacturer-
provided SPICE simulation models [24, 32] are used. In contrast to previous parameters, such as ripple
or output voltage, there is no analytical equation for efficiency estimation based on the given component
characteristics. The efficiency is influenced by multiple factors, such as I2
R losses (ESR, DCR,
RDS(on)), MOSFET gate capacities, switching frequency, and the operating point of the controller.
The search space consists of a discrete set of 16 inductors and 6 capacitors, with the option of placing
two output capacitors in parallel. The components are presented to the LLM with relevant context
information, as shown in Table 3. To compare the LLM-based results against a reference, we calculated
the efficiency of all possible component combinations via brute force.
Table 3: Example components, which the LLM can choose, with context information
Part Number Inductance (µH) Size Code Rated Current (mA) Sat. Current (mA) DCR (Ω)
LQH32PZ2R2NN0 2.20 3225/1210 1600 1550.0 0.0760
LQH3NPH2R2MME 2.20 3030/1212 2100 1800.0 0.0650
LQH3NPZ1R0MME 1.00 3030/1212 3000 2350.0 0.0250
1274AS-H-1R5N 1.50 100100/3939 8900 15300.0 0.0064
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
Figure 14 shows a histogram of the resulting power conversion efficiencies in 20 independent and
identical experiment runs of the LLM-based workflow against the brute-force results. We observe that
the component selections suggested by the LLM-based workflow have an efficiency of around 65%. The
efficiency of all possible combinations ranges roughly from 35% to 75%. The results suggested by the
LLM-based workflow are better than the average solution. Yet, the workflow does not identify the best
possible component combination, which could be expected from conventional optimization approaches.
While the LLM-based workflow is capable to apply general design rules, such as minimizing ESR and
DCR, it performs an insufficient number of simulation iterations to explore the design space, which
prevents it from discovering the optimal solution for these specific conditions in light load operation
mode. The output of the LLM-based workflow can act as reference point for further improvements, as
13

## Page 14

0.35 0.40 0.45 0.50 0.55 0.60 0.65 0.70 0.75
Efficiency
0.0
0.2
0.4
0.6
0.8
1.0
Scaled
Density
All combinations
Mean
o3 + RAG + SPICE
Figure 14: Efficiency of all possible circuits and of the component combinations selected by the LLM-
based workflow
for example in [10], where an LLM is used for design space reduction of optimization techniques.
5 Discussion
Our evaluation shows that LLM-based workflows have strong potential to assist in the design process of
SMPS circuits. This suggests that the breakthroughs of LLMs in domains such as software development
are, to a certain extend, transferable to the hardware development of circuits for PCB applications.
Two main factors that are crucial for good performance are simulation feedback and LLM-reasoning.
The benchmark experiments demonstrate that the LLMs can effectively integrate SPICE simulation
feedback to improve its responses by correcting false assumptions about the circuit or related equation
parameters. The SPICE feedback is also used to refine results derived from approximate analytical
formulas and to verify ”long reasoning chains”. This enables an iterative improvement, where the
solve rate increases significantly within a small number of iterations. Internal reasoning substantially
enhances the performance of LLM-based workflows because it handles required substeps, divides larger
tasks in subtasks, and helps the LLM to consider a broader range of possible solution paths.
The benchmarking experiment shows that the best LLM workflow - o3 + RAG + SPICE - solves
the majority of given parameter tuning tasks for circuits with idealized lumped-element components.
The main remaining limitation is handling topology adaption for larger, more complex circuits. Our
analysis indicates that one primary reason for this limitation is the syntax of the text-based circuit
representations. The SPICE netlist format is disadvantageous for LLMs because its tabular structure
is not language-like, and it has a weak semantic description of component functionality, for example
the format lacks features such as pin designators. As a result the LLM often fails to transfer a correct
solution, which it is able to describe in natural language, into the netlist format.
Therefore, our results motivate future research to develop improved text-based circuit representations
that are better suited for LLM-based workflows as an alternative to SPICE netlists. First proposals in
this direction have been made by [4, 11, 9]. Future research could also investigate the combination of
LLMs with conventional optimization algorithms to achieve an optimal selection of catalog components.
6 Conclusion
This paper evaluates the capabilities of LLM-based workflows to assist in the development of SMPS cir-
cuits for PCB applications. Multiple workflows are presented that combine reasoning LLMs, retrieval-
augmented generation (RAG), and a custom toolkit enabling an LLM to interact with SPICE sim-
ulations to estimate the impact of circuit modifications. To evaluate the LLM-based workflows, two
experiments are conducted. The first experiment focuses on finding idealized components using a
14

## Page 15

broad benchmark of parameter tuning and topology adaption tasks related to SMPS circuits. The
goal of the second experiment is to select an optimal set of catalogue components for circuit optimiza-
tion. Our analysis shows the following key insights: First, internal reasoning capabilities of LLMs are
essential to handle the multi-step design tasks. Second, SPICE simulation feedback can effectively be
integrated into the workflow and provides a significant performance improvement. The combination
of reasoning LLM, RAG, and SPICE feedback enables the workflow to solve a large majority of the
provided parameter tuning tasks with idealized components. Remaining challenges include topology
adaption in larger circuits and design space exploration for circuit optimization. To address these lim-
itations, future research could focus on developing improved LLM-compatible circuit representations
and integrating conventional optimization methods to enhance design space exploration.
References
[1] Bhandari, J., Bhat, V., He, Y., Rahmani, H., Garg, S., and Karri, R. Masala-
chai: A large-scale spice netlist dataset for analog circuits by harnessing ai. arXiv preprint
arXiv:2411.14299 (2024).
[2] Blocklove, J., Garg, S., Karri, R., and Pearce, H. Chip-chat: Challenges and opportu-
nities in conversational hardware design. 1–6.
[3] Chang, K., Wang, Y., Ren, H., Wang, M., Liang, S., Han, Y., Li, H., and Li, X. Chipgpt:
How far are we from natural language hardware design. arXiv preprint arXiv:2305.14019 (2023).
[4] Chen, Z., Huang, J., Liu, Y., Yang, F., Shang, L., Zhou, D., and Zeng, X. Artisan:
Automated operational amplifier design via domain-specific large language model. 1–6.
[5] Fu, Y., Zhang, Y., Yu, Z., Li, S., Ye, Z., Li, C., Wan, C., and Lin, Y. C. Gpt4aigchip:
Towards next-generation ai accelerator design automation via large language models. 1–9.
[6] Heumann, Schomaker, and Shalabh. Introduction to statistics and data analysis. 206–207.
[7] Instruments, T. Understanding buck power stages in switchmode power supplies. Techreport,
Texas Instruments Incorporated, 1999.
[8] Jin, H., Wang, J., Sheng, J., Wu, Y., Chen, J., Wang, Y., and Liu, J. Wiseeda: Llms in
rf circuit design. Microelectronics Journal (2025), 106607.
[9] Jitx, 24.10.2025.
[10] Kochar, D. V., Wang, H., Chandrakasan, A., and Zhang, X. Ledro: Llm-enhanced design
space reduction and optimization for analog circuits. arXiv preprint arXiv:2411.12930 (2024).
[11] Lai, Y., Lee, S., Chen, G., Poddar, S., Hu, M., Pan, D. Z., and Luo, P. Analogcoder:
Analog circuit design via training-free code generation. arXiv preprint arXiv:2405.14918 (2024).
[12] Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler,
H., Lewis, M., Yih, W.-t., Rocktäschel, T., et al. Retrieval-augmented generation for
knowledge-intensive nlp tasks. Advances in neural information processing systems 33 (2020),
9459–9474.
[13] Liu, C., Chen, W., Peng, A., Du, Y., Du, L., and Yang, J. Ampagent: An llm-based
multi-agent system for multi-stage amplifier schematic design from literature for process and
performance porting. arXiv preprint arXiv:2409.14739 (2024).
[14] Liu, C., Liu, Y., Du, Y., and Du, L. Ladac: Large language model-driven auto-designer for
analog circuits. Authorea Preprints (2024).
[15] Liu, M., Pinckney, N., Khailany, B., and Ren, H. Verilogeval: Evaluating large language
models for verilog code generation. 1–8.
15

## Page 16

[16] Liu, S., Fang, W., Lu, Y., Wang, J., Zhang, Q., Zhang, H., and Xie, Z. Rtlcoder:
Fully open-source and efficient llm-assisted rtl code generation technique. IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems 44, 4 (2025), 1448–1461.
[17] Ltc3419 product information and datasheet, 09.2025.
[18] Ltc7802 product information and datasheet, 09.2025.
[19] Ltspice, 08.2025.
[20] Lu, Y., Liu, S., Zhang, Q., and Xie, Z. Rtllm: An open-source benchmark for design rtl
generation with large language model. 722–727.
[21] Microsoft Corporation. Azure openai in azure ai foundry models rest api reference, last
access on 09.2025.
[22] Shi, Y., Tao, Z., Gao, Y., Zhou, T., Chang, C., Wang, Y., Chen, B., Zhang, G., Liu,
A., Yu, Z., et al. Amsnet-kg: A netlist dataset for llm-based ams circuit auto-design using
knowledge graph rag. arXiv preprint arXiv:2411.13560 (2024).
[23] Shi, Y., Zhang, Z., Wang, H., Tao, Z., Li, Z., Chen, B., Wang, Y., Yu, Z., Lin, T.-J.,
and He, L. Amsbench: A comprehensive benchmark for evaluating mllm capabilities in ams
circuits. arXiv preprint arXiv:2505.24138 (2025).
[24] Simsurfing - murata manufacturing, 11.09.2025.
[25] Text-embedding-3-large, last access on 09.2025.
[26] Thakur, S., Ahmad, B., Pearce, H., Tan, B., Dolan-Gavitt, B., Karri, R., and Garg,
S. Verigen: A large language model for verilog code generation. ACM Transactions on Design
Automation of Electronic Systems 29, 3 (2024), 1–31.
[27] Thakur, S., Blocklove, J., Pearce, H., Tan, B., Garg, S., and Karri, R. Autochip:
Automating hdl generation using llm feedback. arXiv preprint arXiv:2311.04887 (2023).
[28] Tsai, Y., Liu, M., and Ren, H. Rtlfixer: Automatically fixing rtl syntax errors with large
language model. 1–6.
[29] Vungarala, D., Alam, S., Ghosh, A., and Angizi, S. Spicepilot: Navigating spice code
generation and simulation with ai guidance. arXiv preprint arXiv:2410.20553 (2024).
[30] Wang, X., Wei, J., Schuurmans, D., Le, Q., Chi, E., Narang, S., Chowdhery, A.,
and Zhou, D. Self-consistency improves chain of thought reasoning in language models. arXiv
preprint arXiv:2203.11171 (2022).
[31] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Xia, F., Chi, E., Le, Q. V., Zhou,
D., et al. Chain-of-thought prompting elicits reasoning in large language models. Advances in
neural information processing systems 35 (2022), 24824–24837.
[32] Y-sim, 11.09.2025.
[33] Yao, S., Yu, D., Zhao, J., Shafran, I., Griffiths, T., Cao, Y., and Narasimhan, K.
Tree of thoughts: Deliberate problem solving with large language models. Advances in neural
information processing systems 36 (2023), 11809–11822.
16

