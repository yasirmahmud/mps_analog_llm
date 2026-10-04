---
title: "LEDRO: LLM-Enhanced Design Space Reduction and Optimization for Analog Circuits"
authors: "Kochar, Dimple Vijay; Wang, Hanrui; Chandrakasan, Anantha; Zhang, Xin"
date: "2024/11/19"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2411.12930"
pdf: "papers/2024_ledro-llm-enhanced-design-space-reduction-and-optimization-for-analog-circu_2411-12930.pdf"
sha256: "be1ffc5d4322d5a3e27d29e87e2443dd41e96a86f3d29b2a1e30d644431945b8"
pages_extracted: 8
---

# LEDRO: LLM-Enhanced Design Space Reduction and Optimization for Analog Circuits

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

arXiv:2411.12930v2
[cs.LG]
5
Jun
2025
LEDRO: LLM-Enhanced Design Space Reduction
and Optimization for Analog Circuits
Dimple Vijay Kochar1
, Hanrui Wang1
, Anantha P. Chandrakasan1
, Xin Zhang2,3
1
Electrical Engineering and Computer Science, Massachusetts Institute of Technology, Cambridge, MA
2
MIT-IBM Watson AI Lab, Cambridge, MA
3
IBM T. J. Watson Research Center, Yorktown Heights, NY
Emails: dkochar@mit.edu, xzhang@us.ibm.com
Abstract—Traditional approaches for designing analog circuits
are time-consuming and require significant human expertise.
Existing automation efforts using methods like Bayesian Op-
timization (BO) and Reinforcement Learning (RL) are sub-
optimal and costly to generalize across different topologies and
technology nodes. In our work, we introduce a novel approach,
LEDRO, utilizing Large Language Models (LLMs) in conjunction
with optimization techniques to iteratively refine the design
space for analog circuit sizing. LEDRO is highly generalizable
compared to other RL and BO baselines, eliminating the need
for design annotation or model training for different topologies
or technology nodes. We conduct a comprehensive evaluation
of our proposed framework and baseline on 22 different Op-
Amp topologies across four FinFET technology nodes. Results
demonstrate the superior performance of LEDRO as it outper-
forms our best baseline by an average of 13% FoM improvement
with 2.15× speed-up on low-complexity Op-Amps and 48% FoM
improvement with 1.7× speed-up on high-complexity Op-Amps.
This highlights LEDRO’s effective performance, efficiency, and
generalizability.
Index Terms—LLM, optimization, opamp, design, topology,
analog, circuit, sizing, FinFET
I. INTRODUCTION
Designing analog circuits involves various subtasks like
topology selection, transistor sizing, layout, and verification
while optimizing power efficiency, maximizing performance,
and reducing the area to meet the target specifications. These
subtasks are complex, requiring time-consuming simulations
along with human expertise [1]. Minor topological modifi-
cations, technology node transfer, and advanced technologies
such as FinFETs further increase this human effort. Thus, there
is an increasing need to automate the process of analog circuit
design.
Recent advancements have introduced automation in opera-
tional amplifier (Op-Amp) design by reformulating the sizing
and biasing of transistors as an optimization problem [2].
Previous works have explored optimization techniques such as
Bayesian Optimization (BO) [2], [3], Reinforcement Learning
(RL) [4], [5], and genetic algorithms [6], [7] to solve this
problem. However, these approaches treat the circuit design
space as a black box, solving without any domain-specific
analog design knowledge that an expert analog designer might
have, which can lead to sub-optimal solutions. For example,
these approaches often overlook the regions of operation or
The project was funded by the MIT-IBM Watson AI Lab.
key transistor biasing points. Also, these approaches are costly
and not generalizable since they often require retraining for
each circuit topology and technology node [4], [5].
Recent works explore utilizing LLMs for analog circuit
design [8]–[13], but most of them do not focus on transistor
level sizing optimization. Refs. [8], [9] utilize LLMs for
analog layout design, [10] uses agentic LLMs to automate
design process, and [11], [13] prioritize topology and correct
functionality of analog circuits. Ref. [12] focuses on transistor
sizing optimization by expecting LLMs to directly provide
the optimized sizing value, which can be challenging to scale
to larger search spaces. To overcome these shortcomings, we
introduce a novel framework - LLM-Enhanced Design Space
Reduction and Optimization (LEDRO) - that synergizes the
mathematical reasoning from optimization techniques with the
circuit knowledge of LLMs. Fundamentally, LEDRO enhances
design space exploration by iteratively leveraging LLMs to
choose a refined search region (instead of direct points) and
optimization techniques to find high-performing circuits in
the chosen region. Fig. 1 provides a high-level illustration
of LEDRO. To ensure LEDRO’s generalizability across tech-
nology nodes and topologies, we perform calibration point
synthesis using optimization techniques to provide reference
examples to the LLM to generate refined regions without
requiring any circuit-specific training. To further improve LLM
outputs, we provide LLM with its knowledge-based self-
reflection and simulation-based optimization feedback. In our
work, we utilize LLaMa3-70B [14] as the LLM and TuRBO
[15] as the optimization technique for LEDRO.
We compare our framework against a state-of-the-art actor-
critic RL framework [4], [16], [17] and a pure TuRBO
approach [15]. To benchmark the generalizability of our
framework, we evaluate it on one of the most comprehensive
setups of 88 different circuits comprising a broad range of 22
Op-Amp topologies across 4 FinFET technology nodes. We
divide the topologies into low and high-complexity Op-Amps
for finer comparisons. For evaluation, we utilize the Figure
of Merit (FoM), an objective function of the normalized,
weighted sum of the gain, unity gain bandwidth, phase margin,
and supply current. Experiments reveal the broad generaliz-
ability of LEDRO with an average FoM improvement of 13%
with 2.15× speed-up for the low-complexity Op-Amps over
our best baseline and 48% FoM improvement with a 1.7×

## Page 2

speed-up for the high-complexity ones. Thus, we show that
LEDRO leverages LLMs to refine the search space to more
relevant ranges to ensure better and more efficient designs. In
addition, LEDRO offers wide versatility as it can be coupled
with any optimization method as a plug-and-play approach.
Overall, we summarize our key contributions as follows:
• We introduce a novel framework, LEDRO, which con-
ducts refined-search space exploration by combining
LLMs with optimization techniques, becoming one of the
first works to utilize LLMs for analog circuit sizing.
• We demonstrate the generalizability of our framework
on an extensive experimental setup comprising 22 Op-
Amp topologies across four different technology nodes,
particularly in FinFET technology
• We showcase the superior performance and speed-up of
LEDRO on our benchmarks compared to state-of-the-
art RL and standalone TuRBO algorithms, establishing
LEDRO as a highly effective and efficient framework for
analog circuit design.
The rest of this paper is organized as follows. Section II
introduces the proposed LEDRO method with a case study
example. Section III provides experiment results on a set of
diverse circuits and Section IV concludes the paper. The code
is available at https://github.com/dimplekochar/LEDRO.
II. METHODS
Our proposed framework LEDRO utilizes a Large Language
Model (LLM) in conjunction with an optimization module to
more efficiently explore the search space for analog design.
A circuit simulator simulates the created analog designs and
estimates the performance. LEDRO continually explores and
improves the design by running these components in iteration
for a fixed set of rounds. In our work, we utilize LLaMa3-
70B [14] as the LLM and TuRBO [15] as our optimizer. For
LLaMa3-70B, we utilize greedy decoding with temperature of
0.8, and maximum generation length of 1000 tokens. In total,
we run LEDRO for ten iterations. We provide a high-level
block diagram of LEDRO in Fig. 2 and a case study in II-D.
A. Calibration Point Synthesis for LLM
Directly identifying optimal points or regions is a non-trivial
task for LLMs since different topologies across technology
nodes can have highly diverse optimal points in the complex
design space. Instead, to effectively utilize LLMs, we create
a calibrated initial prompt which aids the LLM to understand
and navigate the search space better. Specifically, we include
some reference designs in this prompt that provide this search
space calibration and understanding.
To provide these reference designs, we first sample 200
points by running the optimizer over the entire search space
as allowed by the technology node. Next, we extract “good”
points from these sampled points by filtering the design
configurations for which the amplifier gain exceeds a threshold
value. We set the threshold to a low 0 dB so that these “good”
points are easy to find. In rare cases where no “good” points
are found, the optimization can be run longer, or the threshold
value can be lowered (although we did not face such an
issue with our experiments). Finally, we rank these filtered
“good” points on their Figure of Merit (described in the next
paragraph) and add the top five designs to our calibrated initial
prompt as the reference designs, as shown in Fig 1(A).
Figure of Merit: To quantitatively assess the configurations,
similar to [4], [18], [19], we define our Figure of Merit (FoM)
by an objective function utilizing the specifications S of gain
(G), unity gain bandwidth (f), supply current (I), and phase
margin (PM). First we normalize each specification s ∈ S
using its corresponding user-defined boundary value sbound ∈
Sbound by the normalizing function ϕ as
ϕ(s, sbound) =
s − sbound
s + sbound
Thus, if s = sbound, the normalized value is capped at 0; else
ϕ(s, sbound) < 0. Such capping and normalization ensures
the optimization of all specifications rather than the hyper-
optimization of a single specification.
We formulate the FoM objective function V as the weighted
sum of the normalized specifications as
V(S, Sbound; w) =
X
(s,sbound)∈(S,Sbound)
ws.ϕ(s, sbound)
where ws is the weight assigned for specification s. Owing
to the capping in the normalization, the maximum achievable
FoM value is 0, with higher FoMs being more desirable.
B. LLM: First Round Prompt
The design search space for amplifiers is high dimensional
and complex since each transistor can have a wide array of
possible configurations in terms of length, number of fins,
and biasing conditions [20]. LEDRO leverages the pre-trained
knowledge and reasoning capabilities of LLMs to reduce
and refine this design search space, simplifying exploration
for subsequent optimizations (as shown in Fig 1(B)). More
technically, given the input prompt with the calibration points,
LLMs are required to output this refined search space region.
Circuit Representation: To describe the circuit to LLMs in
natural language, we represent it by its netlist. The netlist pro-
vides a detailed description of the connectivity and component
specifications in the circuit. Our studies show that LLMs can
parse and interpret the circuit intricacies using the netlist.
Prompt Setup: The LLM prompt comprises the system
prompt for high-level instructions and the user prompt for spe-
cific inputs and queries. We provide our problem description,
which includes the circuit’s netlist, amplifier topology, and the
specifications in the system prompt. This ensures that these
details serve as a continuous reference to the LLM.
For the user prompt for the first round, we provide the
problem overview and the design objectives and introduce
the initial set of calibrated points (as described in II-A).
Each calibrated point is characterized by the specifications (G,
PM, etc), the transistor parameters (number of fins, biasing,
etc.), the FoM V, and each transistor’s region of operation.
We also provide the specification boundary values and the

## Page 3

Design
space
First round LLM
constrained
design space
Feedback
refined
design space
reference
point
Optimizer
sampling
Sampling for calibration
point synthesis
(A) (B) (C)
Fig. 1. Illustration of LLM-enhanced design space reduction and optimization (LEDRO).
Technology VDD T °C Topology
Calibration
Point
Synthesis
LLM
Zero-shot
LLM
Feedback
LLM
Reflection
Top 5 points
1) Specs
2) Params
3) Region
4) FoM
Output
Ranges
nA1: ...
nB1: ...
vbiasp1: ...
CIRCUIT SIMULATOR
LLM AGENT
OPTIMIZER
vdd!
gnd!
vinp voutp
vinn voutn
+
-
vdd!
gnd!
vinp
voutp
vinn
+
-
Optimizer
Extractor and
Sampler
Fig. 2. Block-diagram of proposed framework.
verbalized FoM objective function. Finally, we add instructions
prompting the LLM to select and appropriately reduce the
design search space, along with output formatting instruc-
tions. Detailed information about the transistor operation (cut-
off, subthreshold, saturation, or linear region) and its self-
knowledge allows the LLM to make informed decisions and
efficiently provide the refined search space region. We provide
an illustration of this prompt in Fig 3.
LLM to Optimization: The LLM-generated refined search
region is then provided back to the optimizer (in our case,
TuRBO). We sample 100 points in this reduced space using
the optimizer and provide corresponding feedback (discussed
in II-C) to the LLM in an iterative fashion. The refined search
space from LLMs is the defining feature of LEDRO as it
prioritizes regional exploitation by significantly reducing the
optimization complexity over a wide-range exploration on the
entire search space. This subsequently accelerates the identi-
fication of high-performing designs and uncovers innovative
configurations that might have remained undiscovered with
only the optimizer due to higher complexity.
C. LLM: Feedback and Reflection for Further Rounds
For subsequent rounds, we remove the problem description
from the LLM prompt and instead add the optimizer feedback
and the self-reflection for effective and better-informed search
space reduction, as shown in Fig 1(C). The optimizer feedback
can be of two forms based on the optimizer’s performance
LLaMA-3-70b first round prompt and response:
Role: System (starting prompt)
simulator lang=spectre
parameters nA1={{nA1}} nB1={{nB1}} nA2={{nA2}} .... vbiasp1={{vbiasp1}} vbiasn1={{vbiasn1}}...
MM6 Voutp Vbiasp2 net23 vdd! pfet l=nA1 nfin=nB1
MM5 Voutn Vbiasp2 net24 vdd! pfet l=nA1 nfin=nB1
MM2 net23 Vbiasp1 vdd! vdd! pfet l=nA2 nfin=nB2 ......
This is a fully_differential_folded_cascode amplifier, using 7nm PTM finfets. Each transistor has
a length parameter l and number of fins nfin. nAx and nBx are variable numbers for these. Vbias
are biasing volts. As we change them, our performance changes.
We are considering the gain, phase margin (PM), unity gain bandwidth (UGBW) and power.
Try to get all transistors in what you think will be the correct region of operation by getting their
role from the netlist provided earlier and changing parameters for sizing and biasing. First step
is to analyze this netlist. Do you understand the role of each transistor in this netlist?
Role: User (first-round prompt, sent with the previous message)
Problem Description:
This is a fully_differential_folded_cascode amplifier, using 7nm PTM finfets. Each transistor ...
We are considering the gain, phase margin (PM), unity gain bandwidth (UGBW) and power.
Initial Data:
I got these points and their reward: metrics- gain: 5.77 , UGBW: 2.91e+04 , PM: 69.2 , power:
4.32e-12 with nA1: 7.45e-08, nB1: 6, nA2: 1.4e-07, nB2: 2, nA3: 3.75e-08, nB3: 3, nA4: 3.04e-07,
nB4: 3, nA5: 3.7e-08, nB5: 4, nA6: 1.24e-07, nB6: 2, vbiasp1: 0.659, vbiasp2: 0.408, vbiasn0: 0.525,
vbiasn1: 0.16, vbiasn2: 0.352, vcm: 0.4, vdd: 0.8, tempc: 27 and transistor regions MM0 is in
cut-off, MM1 is in cut-off, MM2 is in cut-off...
Final Goal:
Our goal is to get reward = 0. Reward gets more negative when the gain achieved < target gain, ...
Try to get all transistors in what you think will be the correct region of operation by getting their
role from the netlist provided earlier and changing parameters for sizing and biasing.
What you need to do and response format guidelines:
What parameters should I change to obtain gain: 3300, funity: 20.0e+6, pm: 70, power: 10e-6?
Give me ranges like nA = [#number, #number], etc - nA1:[10e-9, 400e-9], nB1:[1, 7], ...
Format your answer as "parameter: [value, value]", for example "nA1: [5, 6]", every parameter in
a new line. Keep in mind the initial data you are provided, choose ranges based on that
information as well as your knowledge of analog circuit design and amplifiers ...
Role: Assistant (Response)
Here are the parameter ranges to achieve the desired performance:
nA1: [1e-8, 5e-8]
nB1: [2, 6] .....
Netlist
Calibrated points
Overview
Objective function description
Boundary values
Output
Formatting
Topology
Specifications
Fig. 3. First round prompt and response for folded cascode amplifier.
within the LLM-provided design space from the last round. If
the optimizer performs well - characterized by achieving more
than five “good” points along with an increased FoM relative
to previous rounds - we provide these new points back to the
LLM, acknowledging its effective performance. Contrarily, if
the optimizer performs poorly, we provide this feedback to the
LLM and ask it to explore a new and different search region.
As part of self-reflection, we prompt the LLM to reflect
on its past ranges and strategize what to do next. This self-
reflection is inspired by concepts outlined in [21], where we
prompt the LLM agent to summarize previous design ranges
and look back on its past strategies. Overall, we find that the
optimization feedback and self-reflection encourage the LLM
to dynamically adjust its exploration strategy and efficiently
enhance the refined region. We provide an illustration of the
feedback and reflection prompts in Fig 4.

## Page 4

LLaMA-3-70b feedback and reflection prompts:
Role: User (Feedback prompt)
Results:
I did a sampling of 100 points in the ranges you provided before your reflection and
summarization. I got good points but they still don't meet all the specs, here are the top 5 points
where gain > 1 with their reward. metrics- gain: 3.13e+02, ....
Final Goal:
Our goal is to get reward = 0. Reward gets more negative when the gain achieved < target gain, ...
The top performing point I have until now is: metrics- ... Try to get all transistors in what you think
will be the correct region of operation by getting their role from the netlist provided earlier and
changing parameters.
What you need to do and response format guidelines:
What parameters should I change to obtain... Give me exact ranges (you should give me a new set
of ranges more different from your previous set of ranges if you think you are hitting a local
maximum and the reward isn't improving over the past few messages, feel free to be aggressive)...
Remember all the reflection you did in your past message and what conclusions you drew when
making your predictions.
Role: User (Reflection prompt)
Reflection on the past ranges:
Summarize the history of ranges you have given me so far as to not be stuck in a loop or a local
maxima. Consider the big picture. Let's reflect on what we can do better. Think how you need to
set ranges to meet the specs, how to change them to move the transistors to the right region ...
Role: Assistant (Response)
Here are the new parameter ranges to achieve the desired performance:
nA1: [2.5e-7, 4.5e-7]
nB1: [3, 5] .....
Role: Assistant (Reflection response)
Upon reflection, I realize that my previous ranges were not aggressive enough to explore the
entire design space. I was stuck in a local maxima, and the reward wasn't improving significantly.
To break this cycle, I need to take a step back and reassess my approach. Initially, I provided
ranges that were relatively narrow and centered around the initial data points. While this approach
helped to identify some promising regions, it limited the exploration of the design space. The
subsequent ranges I provided were not significantly different from the initial ones, which led to ...
Summarize past, strategize new
Optimizer feedback
Next round instructions
Fig. 4. Further feedback and reflection prompts and responses for further
rounds for a folded cascode amplifier.
Vdd
gnd
Vin,p
Vout,p Vout,n
Vb,n,0
Vb,n,2
Vb,n,1 Vb,n,1
Vb,n,2
Vb,p,1
Vb,p,2 Vb,p,2
Vin,n
200 400 600 800 1000 1200
Number of steps
0
20
40
60
Gain
(in
dB)
200 400 600 800 1000 1200
Number of steps
106
108
1010
UGBW
(in
Hz)
200 400 600 800 1000 1200
Number of steps
4
3
2
1
FoM
BO-1200
LEDRO (LLaMa-3)
LEDRO (GPT-3.5)
Fig. 5. Differential folded cascode circuit and gain, UGBW, and FoM for
LEDRO with LLaMa3-70b, GPT-3.5-turbo models and BO-1200 vs steps.
D. Case Study: End-to-end example of 7nm differential folded
cascode amplifier
We present a case study for the design of a differential-
ended folded cascode circuit (as shown in Fig. 5) in 7nm
PTM-MG technology [22] using LEDRO. This circuit, chosen
for its relevance in past literature [23]–[25], serves as a
good benchmark for showcasing our model performance. We
conduct calibration point synthesis with TuRBO and add them
to the first round prompt for the LLM as shown in Fig 3. For
subsequent rounds, we follow up with optimizer feedback and
LLM’s self-reflection, as shown in Fig 4. We compare the
optimization performance of two different runs of LEDRO
utilizing two different LLMs with a pure TuRBO optimizer
in terms of Gain, Unity Gain Bandwidth (UGBW), and FoM
vs. optimizer steps in Fig 5. As observed, pure TuRBO over-
optimizes UGBW and is unable to improve gain over the fixed
1200 iterations. Although TuRBO might be able to reach a
similar optimal point, it would take many more optimization
S_LV_C_P
S_LV_C
S_T_P
S_T
D_C_P
D_C
D_T_P
D_T
S_CM_P
S_CM
S_MC
S_MC_P
D_MC
D_MC_P
S_TS
S_C_P
S_C
D_FC
D_FC_P
S_C_CM
S_C_CM_P
D_GB
Amplifier Topologies
10
15
20
Number
of
parameters
High complexity
Low complexity
S - Single ended, D - Differential ended, P - PMOS configuration,
LV - Low Voltage, C - Cascode, CM - Current Mirror, MC - Miller Compensated,
FC - Folded Cascode, T - Telescopic, TS - Two Stage, GB - Gain Boosting
gainmax=50dB,
UGBWmax=5MHz
PMmax=70
IDD,max=5 A
gainmax=70dB,
UGBWmax=20MHz
PMmax=70
IDD,max=10 A
Fig. 6. Parameters per Op-Amp and division into high and low-complexity
Op-Amps and their boundary values.
steps. Both runs of LEDRO achieve stronger gain, UGBW,
and, in turn, high FoM in fewer optimizer steps. We conduct
future comprehensive experiments using only LLaMa3-70B
[14] since we validated the utility of GPT-3.5-turbo through
this case study and it is more expensive to run the GPT model.
III. EXPERIMENTS AND RESULTS
We describe our experimental setup in III-A, the main
results in III-B and supporting analysis in III-C.
A. Experimental Setup
Benchmarks: To demonstrate the broad generalizability of
our framework, we conduct experiments using a diverse set
of 22 amplifier topologies, including both PMOS and NMOS
input configurations with single-ended and differential-ended
output designs, ranging from 7 to 19 transistors. To create this
benchmark, we modify and repurpose the subcircuit annotation
task in [26] to design the parametrized FinFET netlists. For
richer benchmarking, we split the amplifiers into 10 low-
complexity and 12 high-complexity Op-Amps based on the
number of optimization parameters. To ensure robustness, we
utilize predictive technology models for multi-gate transistors
(PTM-MG) [22] across multiple technology nodes, specifically
7nm, 10nm, 14nm, and 16nm. We show the different Op-Amps
with their categorization and number of parameters in Fig 6.
Evaluation Metric: We consider performance and efficiency
as the major evaluation metrics. Following previous works
like [12], [19], we utilize the number of optimization steps
to measure the efficiency. For performance, the evaluation
metric is the Figure of Merit (FoM), as we defined in II-A,
using user-defined specification boundary values. Different
amplifiers have different characteristics - some achieve higher
gain, some are optimized for low power, etc - and thus can
have varied specification boundaries. These characteristics can
vary significantly when scaling the same topology across
technology nodes. Most previous works [17], [25] experiment
with just 1-3 Op-Amps, which allows them to set customized
FoM boundary values for each circuit easily. Since our work
focuses on a comprehensive benchmark of 22 Op-Amps across

## Page 5

2.0
1.5
1.0
0.5
FoM
S_LV_C_P
S_LV_C
S_T_P
S_T
D_C_P
D_C
D_T_P
D_T
S_CM_P
S_CM
0
500
1000
1500
2000
Number
of
Steps
7nm
S_LV_C_P
S_LV_C
S_T_P
S_T
D_C_P
D_C
D_T_P
D_T
S_CM_P
S_CM
10nm
S_LV_C_P
S_LV_C
S_T_P
S_T
D_C_P
D_C
D_T_P
D_T
S_CM_P
S_CM
14nm
S_LV_C_P
S_LV_C
S_T_P
S_T
D_C_P
D_C
D_T_P
D_T
S_CM_P
S_CM
16nm
S - Single ended, D - Differential ended, P - PMOS configuration,
LV - Low Voltage, C - Cascode, CM - Current Mirror, T - Telescopic
Amplifier Topologies
RL-2000
Average BO-2000
BO-2000
Average LEDRO-1200
LEDRO-1200
Fig. 7. Performance (FoM) and Efficiency (number of steps) comparison of LEDRO with other baselines for different low-complexity amplifier topologies
across four different technology nodes. As shown, LEDRO not only achieves better FOM, but also takes fewer steps to reach the best BO-2000 FoM.
four technology nodes, it is challenging and infeasible to set
customized boundary values for each circuit.
Instead, we specify two sets of boundary values - one
each for the low-complexity and the high-complexity Op-Amp
groups. Specifically, for our low-complexity group, we set
Gbound = 50 dB, Ibound = 5 µA, fbound = 5 MHz, PMbound =
70°. For our high-complexity group, we set Gbound = 70 dB,
Ibound = 10 µA, fbound = 20 MHz, PMbound = 70°. We set
wG = 3, wf = wP M = 1, and wI = −1 for the FoM objective
function. These parameters are specifically chosen by circuit
experts to be challenging for most amplifiers to rigorously
evaluate the capabilities of the optimization algorithms.
Baselines and Implementation: We utilize Cadence Spec-
tre to run our simulations. Primarily, we compare our approach
against the pure TuRBO optimizer [15] which is proposed for
automated analog sizing in [3], and also used for subspace
sampling in [19]. TuRBO is run for 1200 steps (BO-1200),
matching our method’s maximum allowable evaluation budget
(200 steps for calibration point synthesis and 100 optimizer
steps × 10 LEDRO iterations). As a stronger baseline, we
also compare with an over-optimized TuRBO run with an
evaluation budget of 2000 steps (namely BO-2000). For a fair
comparison, we ensure that LEDRO and the TuRBO baseline
use the same starting points by continuing to train TuRBO
using the initial 200 steps used for calibration point synthesis
in LEDRO, and report the best of five runs. We also include
a Reinforcement Learning (RL) baseline, which was also run
for 1200 and 2000 steps (namely RL-1200 and RL-2000). For
this baseline, we modify the base model from [4], with the
recent improvements of actor-critic networks of [16], [17].
We provide the actor with the region of operation knowledge
and the critic with additional vgs, vds, gm, and ids informa-
tion for better optimization. Simulation-based approaches like
genetic algorithms [7], [27] are sample-inefficient and poor
in performance compared to TuRBO as shown in [3], [4].
AmpAgent’s [10] device sizing agent primarily uses TuRBO
algorithm. Since its code is not public, initial-value estimates
TABLE I
AVERAGE FIGURE OF MERIT AND OPTIMIZATION STEPS FOR
LOW-COMPLEXITY AND HIGH-COMPLEXITY OP-AMPS.
Low-Complexity High-Complexity
Algorithm 7nm 10nm 14nm 16nm 7nm 10nm 14nm 16nm
AVG FIGURE OF MERIT (FOM) ↑
RL-1200 -1.79 -1.73 -1.76 -1.82 -2.69 -2.64 -2.27 -2.24
RL-2000 -1.72 -1.68 -1.74 -1.80 -2.62 -2.55 -2.20 -2.13
BO-1200 -1.57 -1.48 -1.57 -1.61 -1.46 -1.26 -1.18 -1.16
BO-2000 -1.57 -1.47 -1.47 -1.55 -1.19 -1.21 -1.12 -1.08
LEDRO -1.45 -1.32 -1.28 -1.25 -0.53 -0.62 -0.61 -0.64
% boost
8% 10% 13% 20% 55% 48% 45% 41%
over BO-2000
AVG NUMBER OF STEPS TO BEST BO-2000 FOM ↓
BO-2000 680 1030 970 1040 992 1217 1042 875
LEDRO 440 440 390 460 417 675 575 750
were not available. Our method will show clear improvements
over TuRBO, benchmarking against AmpAgent’s approach to
our best capability. Despite efforts to replicate ADO-LLM’s
[12] process on our case study, it yielded suboptimal results.
Due to unavailable implementation details, we do not include
this baseline for our comparisons.
B. Results
We show a detailed breakdown of the performance and
efficiency for each amplifier topology and technology node in
Fig 7 and Fig 8 for the low-complexity and high-complexity
groups, respectively. We notice a clear trend wherein our
model LEDRO not only achieves the best FoM performance
but also reaches optimal points in lesser number of steps.
Through Table I, we present a summary of the average
maximum FoM achieved across the low and high-complexity
groups of amplifier designs across the different technology
nodes. Our results show the 8-20% and 16-21% improvement
in amplifier performance for low-complexity Op-Amps com-
pared to the 2000-step TuRBO and 2000-step RL optimization,

## Page 6

4
3
2
1
0
FoM
S_MC
S_MC_P
D_MC
D_MC_P
S_TS
S_C_P
S_C
D_FC
D_FC_P
S_C_CM
S_C_CM_P
D_GB
0
500
1000
1500
2000
Number
of
Steps
7nm
S_MC
S_MC_P
D_MC
D_MC_P
S_TS
S_C_P
S_C
D_FC
D_FC_P
S_C_CM
S_C_CM_P
D_GB
10nm
S_MC
S_MC_P
D_MC
D_MC_P
S_TS
S_C_P
S_C
D_FC
D_FC_P
S_C_CM
S_C_CM_P
D_GB
14nm
S_MC
S_MC_P
D_MC
D_MC_P
S_TS
S_C_P
S_C
D_FC
D_FC_P
S_C_CM
S_C_CM_P
D_GB
16nm
S - Single ended, D - Differential ended, P - PMOS configuration,
C - Cascode, CM - Current Mirror, FC - Folded Cascode,
MC - Miller Compensated, TS - Two Stage, GB - Gain Boosting
Amplifier Topologies
RL-2000
Average BO-2000
BO-2000
Average LEDRO-1200
LEDRO-1200
Fig. 8. Performance (FoM) and Efficiency (number of steps) comparison of LEDRO with other baselines for different high-complexity amplifier topologies
across four different technology nodes. As shown, LEDRO not only achieves better FOM, but also takes fewer steps to reach the best BO-2000 FoM.
7nm 10nm 14nm 16nm
Technology Node
0
5
10
15
20
Avg
Gain
%
Improvement
Low Complexity
High Complexity
Fig. 9. Average percentage improvement in gain for both sets of amplifiers
across technology nodes.
respectively, across technology nodes. These improvements
are further amplified in the high-complexity Op-Amps with
41-55% improvement over TuRBO-2000 and 70-80% over
RL-2000. Notably, this increased improvement from low to
high-complexity is consistently observed across all technology
nodes. Such consistency suggests that while low-complexity
and low-dimensional circuits are easy to optimize even with
more simplistic methods, designing higher-complexity circuits
is still challenging for conventional optimization algorithms.
On the other hand, our method LEDRO leverages LLMs
to reduce and refine the high-complexity search space and
efficiently use optimization on this lower-complexity refined
space to provide strong improvements for the high-complexity
circuits as well. Such results highlight the robustness, gener-
alizability, and scalability of LEDRO.
We also compare the optimization speed in terms of number
of steps required by LEDRO and the best baseline BO-
2000 to achieve the best FoM achieved by the BO-2000
algorithm, as shown in Table I. Results show that LEDRO
achieves an average speed-up of 1.54 ∼ 2.48× across low-
complexity Op-Amps and 1.17 ∼ 2.38× for high-complexity
Op-Amps across the technology nodes relative to the BO-
2000 baseline. To compare computational latencies of BO-
2000 and LEDRO, we compare the average time to achieve
the best FoM achieved by the BO-2000 algorithm, while also
accounting for the LLM prompt-response time. Relative to
BO-2000, LEDRO achieves this optimum in: 0.49 ∼ 0.72×
time for low-complexity Op-Amps 0.47 ∼ 0.98× time for
high-complexity Op-Amps, across technology nodes. This
demonstrates the superior optimization efficiency of LEDRO
compared to traditional optimization techniques.
C. Analysis: Op-Amp gain (G) improvement
When we examine the detailed improvements introduced
by LEDRO, it is remarkable that LEDRO not only enhances
the overall FoM, but also demonstrates exceptional efficiency
in improving the Op-Amp gain. The average improvement in
terms of gain achieved by LEDRO relative to BO-2000 is
plotted in Fig. 9. It is noteworthy that the gain improvement
is particularly significant for high-complexity circuits across
different technology nodes, highlighting the advantage of using
LEDRO in design automation. The gain values were capped
at Gbound if either of the methods met the boundary gain
value, ensuring a fair comparison. Overall, LEDRO provides
an average improvement of about 15-23% on high-complexity
Op-Amps and about 3-6% on low-complexity Op-Amps.
IV. CONCLUSION
In this paper, we introduce LEDRO, which leverages LLMs
in synergy with optimization techniques to efficiently explore
design space of Op-Amps. Our method advances analog circuit
design automation by iteratively refining the design search
space through the LLM’s prior knowledge and reasoning, thus
facilitating strong optimization performance and better designs
without human expertise. Experiments across a wide range
of topologies and technology nodes demonstrate LEDRO’s
generalizability with 13-48% FoM improvements and effi-
ciency with 1.7 ∼ 2.15× speed-ups compared to conventional
optimization techniques. Furthermore, LEDRO is significantly
stronger in designing high-complexity and high-dimensional
circuits, which provides promise for more practical use cases.
In conclusion, the generalizability and adaptability with other
optimization techniques marks an advancement in the field, of-
fering a solution for complex analog circuit design challenges.

## Page 7

ACKNOWLEDGMENT
We thank MIT Microsystems Technology Laboratory
(MTL) for compute and simulation environment access, and
IBM BAM for LLM API access.
REFERENCES
[1] Harjani, Ramesh, Rob A. Rutenbar, and L. Richard Carley. ”OASYS:
A framework for analog circuit synthesis.” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems 8.12 (1989):
1247-1266.
[2] Lyu, Wenlong, et al. ”An efficient bayesian optimization approach
for automated optimization of analog circuits.” IEEE Transactions on
Circuits and Systems I: Regular Papers 65.6 (2017): 1954-1967.
[3] Touloupas, Konstantinos, Nikos Chouridis, and Paul P. Sotiriadis. ”Local
Bayesian optimization for analog circuit sizing.” 2021 58th ACM/IEEE
Design Automation Conference (DAC). IEEE, 2021.
[4] Settaluri, Keertana, et al. ”Autockt: Deep reinforcement learning of
analog circuit designs.” 2020 Design, Automation & Test in Europe
Conference & Exhibition (DATE). IEEE, 2020.
[5] Wang, Hanrui, et al. ”GCN-RL circuit designer: Transferable transistor
sizing with graph neural networks and reinforcement learning.” 2020
57th ACM/IEEE Design Automation Conference (DAC). IEEE, 2020.
[6] Koza, John R., et al. ”Automated synthesis of analog electrical circuits
by means of genetic programming.” IEEE Transactions on evolutionary
computation 1.2 (1997): 109-128.
[7] Cohen, M. Genetic Algorithm Software System for Analog Circuit
Design. Procedia CIRP 36 (2015) 17-22
[8] Liu, Bingyang, et al. ”LayoutCopilot: An LLM-powered Multi-agent
Collaborative Framework for Interactive Analog Layout Design.” arXiv
preprint arXiv:2406.18873 (2024).
[9] Chen, Guojin, et al. ”LLM-Enhanced Bayesian Optimization for
Efficient Analog Layout Constraint Generation.” arXiv preprint
arXiv:2406.05250 (2024).
[10] Liu, Chengjie, et al. ”Ampagent: An llm-based multi-agent system for
multi-stage amplifier schematic design from literature for process and
performance porting.” arXiv preprint arXiv:2409.14739 (2024).
[11] Lai, Yao, et al. ”AnalogCoder: Analog Circuit Design via Training-Free
Code Generation.” arXiv preprint arXiv:2405.14918 (2024).
[12] Yin, Yuxuan, et al. ”ADO-LLM: Analog Design Bayesian Optimization
with In-Context Learning of Large Language Models.” arXiv preprint
arXiv:2406.18770 (2024).
[13] Chen, Zihao, et al. ”Artisan: Automated Operational Amplifier Design
via Domain-specific Large Language Model.” Proceedings of the 61st
ACM/IEEE Design Automation Conference. 2024.
[14] Dubey, Abhimanyu, et al. ”The llama 3 herd of models.” arXiv preprint
arXiv:2407.21783 (2024).
[15] Eriksson, David, et al. ”Scalable global optimization via local Bayesian
optimization.” Advances in neural information processing systems 32
(2019).
[16] Choi, Minjeong, et al. ”Reinforcement Learning-based Analog Circuit
Optimizer using g m/I D for Sizing.” 2023 60th ACM/IEEE Design
Automation Conference (DAC). IEEE, 2023.
[17] Somayaji, NS Karthik, Hanbin Hu, and Peng Li. ”Prioritized reinforce-
ment learning for analog circuit optimization with design knowledge.”
2021 58th ACM/IEEE Design Automation Conference (DAC). IEEE,
2021.
[18] Zhang, Jinxin, et al. ”Automated Design of Complex Analog Circuits
with Multiagent based Reinforcement Learning.” 2023 60th ACM/IEEE
Design Automation Conference (DAC). IEEE, 2023.
[19] Gu, Tianchen, et al. ”tSS-BO: Scalable Bayesian Optimization for
Analog Circuit Sizing via Truncated Subspace Sampling.” 2024 Design,
Automation & Test in Europe Conference & Exhibition (DATE). IEEE,
2024.
[20] Li, Yaping, et al. ”An artificial neural network assisted optimization
system for analog design space exploration.” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems 39.10
(2019): 2640-2653.
[21] Shinn, Noah, et al. ”Reflexion: Language agents with verbal reinforce-
ment learning.” Advances in Neural Information Processing Systems 36
(2024).
[22] Sinha, Saurabh, et al. ”Exploring sub-20nm FinFET design with pre-
dictive technology models.” Proceedings of the 49th Annual Design
Automation Conference. 2012.
[23] Budak, Ahmet Faruk, et al. ”An efficient analog circuit sizing method
based on machine learning assisted global optimization.” IEEE Trans-
actions on Computer-Aided Design of Integrated Circuits and Systems
41.5 (2021): 1209-1221.
[24] Budak, Ahmet F., et al. ”Dnn-opt: An rl inspired optimization for analog
circuit sizing using deep neural networks.” 2021 58th ACM/IEEE Design
Automation Conference (DAC). IEEE, 2021.
[25] Zhao, Zhenxin, and Lihong Zhang. ”Deep reinforcement learning for
analog circuit sizing.” 2020 IEEE International Symposium on Circuits
and Systems (ISCAS). IEEE, 2020.
[26] Kunal, Kishor, et al. ”GANA: Graph convolutional network based auto-
mated netlist annotation for analog circuits.” 2020 Design, Automation
& Test in Europe Conference & Exhibition (DATE). IEEE, 2020.
[27] B. Liu, F. V. Fernandez, and G. G. E. Gielen, “Efficient and accurate
statistical analog yield optimization and variation-aware circuit sizing
based on computational intelligence techniques,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 30, no.
6, pp. 793–805, 2011.

## Page 8

V. APPENDIX
A. Additional experiments and ablations
Temperature robustness: We evaluated LEDRO’s robustness
across different temperature values, specifically for the case
study in Sec. II-D. The best performance for the case study
was achieved at temperature = 0.8, which was used for
all experiments for consistency. Other temperatures (0.5–0.9)
yielded performances within 0.4 FoM of the best setting
and still outperformed the baseline by >1.7 FoM. Higher
temperatures produced more diverse search spaces, while
lower temperatures led to more deterministic explorations. The
best temperature=0.8 is high, and it suggests that the added
diversity due to sampling aids the LLM for better exploration
for the optimization problem.
LLM Generalizability: While LLaMA3-70B [14] was pri-
marily used, LEDRO is not restricted to a single LLM. Results
in Fig 5. demonstrate its effectiveness with GPT-3.5, confirm-
ing its versatility. Due to resource limitations, we conduct
future comprehensive experiments only using LLaMa3-70B
since we validated the utility of GPT-3.5-turbo through the
case study and also since it is more expensive to run the GPT
model. To establish this generalizability across LLMs better,
we conduct additional experiments with the newer LLaMA3.3-
70B. Our experiments revealed that LLaMA3.3-70B runs also
show comparable performance.
Ablating the feedback loop and reflection mechanism:
To emphasize the importance of two components we use in
our LLM pipeline, we conduct a small ablation study on
the case study by removing these components individually
and comparing the model performance. First, removing the
feedback loop reduces LEDRO to a single iteration, causing a
100–300% FoM reduction. This emphasizes its crucial role in
iterative improvement, which lead to continual improvements.
Secondly, removing the self-reflection prompts results in a
120% FoM reduction. This highlights the significance of self-
guided reflections in calibrating the model’s outputs and in
turn, refining the design space.
Latency and complexity analysis: Since LLM-based and
non-LLM approaches are vastly different, comparing their
computational complexity is not trivial. Nonetheless, we at-
tempt to compare their complexities in terms of runtime com-
putes. Specifically, we consider the two best models LEDRO
and BO-2000 and compare the average runtimes to reach the
best optimum of BO-2000. For BO-2000, we consider the
time taken for the TuRBO algorithm to reach the optimum.
For LEDRO, we take the summation of the time for the
LLM API calls (accounting for the LLM prompt response
time) and the time to run the intermediate TuRBO iterations.
Overall, relative to BO-2000, LEDRO achieves this optimum
0.55× faster for low-complexity and 0.68× faster for high-
complexity Op-Amps. These gains stem from iterative design
space narrowing, minimizing the number of optimization steps.
B. Discussion and Comparison about recent LLM works in
analog circuit sizing
Recent times have seen a surge in the utilization of LLMs
for analog circuit design. While we briefly cover relevant
literature in the main text, this appendix provides additional
clarification regarding the scope and applicability of compar-
ison with selected works.
AnalogCoder [11] primarily addresses functional verifica-
tion rather than transistor-level optimization, as noted in their
paper. Their objectives differ from ours, making a direct
performance comparison not meaningful in this context.
AmpAgent [10] employs the TuRBO [15] algorithm as the
core of its device sizing agent. While we aimed to evaluate
our approach relative to AmpAgent, the absence of public
code and initial-value estimates limited the reproducibility of
their exact setup. Nonetheless, we benchmarked against the
TuRBO algorithm, to the best of our ability, and observed
clear improvements.
For ADO-LLM [12], we attempted to adapt their method-
ology to our case study described in Sec. II-D. However, the
results were suboptimal, likely due to limited implementation
details. To ensure accuracy and fairness, we chose not to
include these findings. It is also worth noting that, in contrast
to ADO-LLM, our work provides extensive benchmarking
against a wide range of recent RL and BO approaches.
C. Further information
We hypothesize that LLMs bring design expertise to op-
timization by identifying relationships in circuit netlists, like
relative transistor-sizing. These insights generalize across tech-
nology nodes, avoiding computationally prohibitive optimiza-
tion over the entire parameter space defined by technology
model files. By narrowing the design space to high-potential
regions, LLMs enable faster, more efficient searches, boosting
performance. We would also like to emphasize that our method
requires no LLM training or fine-tuning and just works with
LLM prompting.
While LLMs introduce decision-making complexity, this
challenge also applies to RL/BO methods, which rely on
opaque neural network models. Our framework improves
design efficiency compared to traditional human-driven pro-
cesses, which aligns with the increasing adoption of AI/LLMs
across circuit design tasks. Our methodology allows for refine-
ment based on circuit knowledge, mitigating concerns around
interpretability while enhancing design convergence.
As for the scope of our framework, while our evaluations
focus on Op-Amp designs, the proposed approach is general-
izable to broader analog circuit design tasks. The framework’s
heuristic-driven design space reduction can be applied across
multiple domains, including RF circuits, data converters, and
other automation tasks where informed design space explo-
ration accelerates convergence and efficiency. Future work
aims to explore these extensions further.

