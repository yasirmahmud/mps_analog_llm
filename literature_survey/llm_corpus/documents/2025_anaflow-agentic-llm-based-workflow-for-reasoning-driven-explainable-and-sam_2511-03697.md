---
title: "AnaFlow: Agentic LLM-based Workflow for Reasoning-Driven Explainable and Sample-Efficient Analog Circuit Sizing"
authors: "Ahmadzadeh, Mohsen; Chen, Kaichang; Gielen, Georges"
date: "2025/11/05"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2511.03697"
pdf: "papers/2025_anaflow-agentic-llm-based-workflow-for-reasoning-driven-explainable-and-sam_2511-03697.pdf"
sha256: "c29e52ca68076ab2b52dffdaa467c9033e2cb47a315ba11333f98bd09e049736"
pages_extracted: 7
---

# AnaFlow: Agentic LLM-based Workflow for Reasoning-Driven Explainable and Sample-Efficient Analog Circuit Sizing

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AnaFlow: Agentic LLM-based Workflow for
Reasoning-Driven Explainable and Sample-Efficient
Analog Circuit Sizing
Mohsen Ahmadzadeh, Kaichang Chen, Georges Gielen
ESAT-MICAS, KU Leuven, 3001 Leuven, Belgium
{mohsen.ahmadzadeh, kaichang.chen, georges.gielen}@kuleuven.be
Abstract—Analog/mixed-signal circuits are key for interfacing
electronics with the physical world. Their design, however,
remains a largely handcrafted process, resulting in long and
error-prone design cycles. While the recent rise of AI-based
reinforcement learning and generative AI has created new
techniques to automate this task, the need for many time-
consuming simulations is a critical bottleneck hindering the
overall efficiency. Furthermore, the lack of explainability of the
resulting design solutions hampers widespread adoption of the
tools. To address these issues, a novel agentic AI framework for
sample-efficient and explainable analog circuit sizing is presented.
It employs a multi-agent workflow where specialized Large
Language Model (LLM)-based agents collaborate to interpret
the circuit topology, to understand the design goals, and to
iteratively refine the circuit’s design parameters towards the
target goals with human-interpretable reasoning. The adaptive
simulation strategy creates an intelligent control that yields a
high sample efficiency. The AnaFlow framework is demonstrated
for two circuits of varying complexity and is able to complete
the sizing task fully automatically, differently from pure Bayesian
optimization and reinforcement learning approaches. The system
learns from its optimization history to avoid past mistakes and
to accelerate convergence. The inherent explainability makes this
a powerful tool for analog design space exploration and a new
paradigm in analog EDA, where AI agents serve as transparent
design assistants.
Index Terms—Agentic AI, Large Language Models, Analog
Design Automation, Analog Circuit Sizing, Explainability.
I. INTRODUCTION
Analog and mixed-signal circuits form the essential in-
terface between the digital world of computation and the
continuous signals of the physical world. Their role is crit-
ical in a vast array of applications. Despite their ubiquity,
the design of these circuits, however, remains a formidable
challenge. Unlike digital design, which benefits from extensive
automation, analog circuit sizing is still largely a handcrafted
process reliant on the intuition and experience of human
designers. This manual approach results in long, costly, and
error-prone design cycles, creating a significant bottleneck in
the development of modern electronic systems [1]–[3].
To boost the automation process, the field of Electronic
Design Automation (EDA) is increasingly turning to Artificial
Intelligence (AI). Figure 1 shows the history of approaches
explored in research. It will be described in more detail
Fig. 1: Historic trend of different approaches used for the
automation of analog circuit sizing.
in Section II. Early efforts employing expert systems, op-
timization methods such as Genetic Algorithms (GA) [4],
[5] Bayesian optimization (BO) [6], [7], and more recently
Reinforcement Learning (RL) [2], [3], [8]–[15] have demon-
strated the potential to automate the search for optimal circuit
sizes. However, these pioneering techniques suffer from two
fundamental limitations that hinder their widespread adoption.
• First, they are notoriously sample-inefficient, often requir-
ing many thousands of computationally expensive circuit
simulations to converge to a solution [2], [3], [10]. This
CPU time explosion problem makes them impractical for
complex designs.
• Secondly, they function as "black boxes," providing solu-
tions without offering any insight into them. This lack of
explainability prevents designers from trusting the results,
nor understanding the trade-offs used, leading to a critical
lack of confidence in the automated solutions and the
analog design automation tools that generate them.
To address the above challenges, we resort to the emerging
Large Language Model (LLM)-based agentic AI paradigm
[16], [17]. This paper introduces a novel framework, called
AnaFlow, for analog circuit sizing that is both reasoning-
driven, sample-efficient, and inherently explainable. Our
© 2025 IEEE. This article has been accepted for publication by the 2025 International Conference on Computer-Aided Design (ICCAD 2025) and was
presented in Munich, October 2025. All copyrights are reserved for IEEE.
arXiv:2511.03697v1
[cs.LG]
5
Nov
2025

## Page 2

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
AnaFlow approach moves beyond pure numerical optimiza-
tion and mimics the cognitive workflow of an expert ana-
log designer. We employ an agentic system where multiple
specialized LLM agents collaborate to interpret connections,
find matching and other constraints, formulate design steps
based on their acquired analog circuit theory knowledge,
execute targeted simulations, and iteratively refine the design
towards the target design goals through reasoning feedback
and through the calling of optimizers (RL, BO, etc.). By
grounding the optimization process in an explicit, human-
interpretable reasoning framework, AnaFlow has the potential
to reduce the number of required simulations, to fully automate
the sizing task, and to produce a fully justified design rationale
besides finding optimized design parameters.
This paper makes the following key contributions:
• We present a novel, multi-agent LLM workflow that mim-
ics the workflow of expert designers and brings structured
reasoning and explainability to the analog circuit sizing
task.
• We demonstrate that the reasoning-driven approach
achieves better sample efficiency, can call optimizers and
can achieve target specifications using a fully automated
process that requires minimal input from the users, as
opposed to other established black-box optimization ap-
proaches.
• We demonstrate our framework’s explainability by show-
ing its capability in producing human-interpretable rea-
soning for the steps taken during the circuit sizing.
• We show our framework’s capability by sizing two oper-
ation amplifiers of different complexities; all through rea-
soning, planning, tool use and decision feedback within
AnaFlow.
This paper is organized as follows. Section II will briefly
review the evolution of automated analog circuit sizing ap-
proaches, from traditional optimization algorithms to re-
cent LLM-based approaches, highlighting the remaining chal-
lenges. Section III will detail the architecture of our proposed
AnaFlow agentic LLM workflow, describing the roles and
interactions of the specialized agents incorporated. Section IV
will present our experimental results, providing a quantitative
analysis of our method’s sample efficiency and a qualitative
demonstration of its explainability. Finally, Section V will
conclude the paper with a summary of our contributions.
II. BACKGROUND AND RELATED WORK
Automating analog circuit sizing has been a long-standing
pursuit in the Electronic Design Automation (EDA) commu-
nity. The approaches have evolved significantly over the years,
moving from expert systems over optimization methods to
more machine learning-based solutions. These were depicted
in Figure 1.
A. Expert Systems and Optimization Methods
Early attempts to automate analog sizing were based on
expert system techniques [18], [19]. They failed as it was
difficult to make all designer expertise explicit.
Then came optimization methods that explore the vast
design space by successively evolving candidate solutions.
Example algorithms are simulated annealing and population-
based genetic algorithms (GAs) [4], [5]. While capable of
finding pretty optimal solutions, these methods are notoriously
inefficient, often requiring many tens of thousands of circuit
simulations to converge. Furthermore, their inherent random-
ness and lack of learning mechanisms mean that knowledge
from one optimization run cannot be transferred to another one
[10]. They also provide no insight into the design solutions
obtained and the design choices made.
Because of the computational explosion, Bayesian Opti-
mization (BO) was adopted by researchers for its greater
sample efficiency [6], [7]. The BO methods rely on Gaussian
Processes (GPs) to build surrogate models of the objective
function. This makes them more suited for expensive black-
box problems. BO’s applicability, however, is limited by its
computational complexity, which scales cubically with the
number of observations [20]. This excessive runtime makes
it impractical for problems with more than 20 variables [21],
rendering it unsuitable for complex circuits of truly high-
dimensional design space [2], [3].
B. Reinforcement Learning Approaches
In recent years, Reinforcement Learning (RL) has emerged
as a powerful paradigm for analog circuit sizing. Actor-Critic
RL agents learn a sizing policy (the Actor neural network) by
interacting with a circuit simulator (the environment), receiv-
ing rewards based on the performance metrics and maximizing
the sum of the expected rewards (the Critic neural network)
[8]–[14]. Recent studies have shown that RL can outperform
both GA and BO in terms of sample efficiency and scalability.
The learning-based nature of RL allows for transfer learning
[9], where a policy trained on one circuit can be fine-tuned
for another one. Also, multi-agent schemes [2], [3], [15] have
demonstrated the potential to scale to complex, industrial-scale
designs. Despite these advances, RL-based methods still face
significant hurdles. They still require hundreds to thousands
of simulations to train an effective policy. More critically,
the learned policy is still a black box, typically encoded in
the weights of the trained Actor-Critic networks. This opacity
makes their decision-making process inscrutable, offering no
explanation to the designer for why certain design parameter
values were chosen. This lack of explainability is a major
barrier to the approach’s adoption in the industry, as human
designers are hesitant to trust solutions they cannot interpret
nor understand.
C. The Rise of LLM-based Agentic AI
Since the advent of Large Language Models (LLMs), a
new paradigm for design automation is emerging, centered
on LLM-based agentic workflows. LLM agents can leverage
their vast pre-trained knowledge to reason, plan, and use tools
to accomplish complex tasks, such as customer responding,
automated software development, complex web navigation,
social behavior simulation [16], [17], etc. The power of these
2

## Page 3

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
Fig. 2: Overview of the four phases in AnaFlow.
workflows stems not only from simple prompt engineering,
but also from sophisticated context engineering [22], where
the agents are dynamically provided with the necessary in-
formation—such as documents, data, and access to external
tools—to execute a task. This ability to dynamically provide
the right information and tools, in the right format, and at
the right time in the flow is the core of context engineering,
enabling dynamic and flexible problem solving [22].
The agentic paradigm is now also finding ground in the
complex domain of EDA, where researchers are applying
LLMs across the entire chip design stack. Also in analog
design, generative AI and agentic workflows are emerging to
tackle the remaining challenges. AnalogGenie [23] focuses
on the creative task of topology generation by training a
GPT neural network on connecting devices and sub-blocks to
produce analog circuit structures. AnalogCoder generates the
circuit implementation code in PySpice through a feedback-
enhanced, training-free flow [24]. Beyond this topological
design, agents are also being integrated into the optimization
loops: ADO-LLM [25] combines an LLM with Bayesian
Optimization for efficient analog circuit sizing, leveraging the
LLM’s knowledge to guide the search for optimal parameters.
The LLM acts as a knowledge-infused proposer in parallel
with a Bayesian optimizer. However, it decides its own pro-
posals rather than directing the overall workflow or providing
a retrospective analysis of the full optimization trajectory and
trade-offs, which is the focus of our AnaFlow framework in
this paper.
III. METHODOLOGY
A. Foundations of the Agentic Design Workflow
The efficacy of our agentic system is rooted in core LLM
practices and capabilities: context engineering, tool integra-
tion, and structured output generation. Modern LLM appli-
cations depend critically on context engineering, where the
input prompt is carefully crafted with instruction prompts and
relevant data to guide the model toward a desired outcome.
This goes beyond simple queries and represents the primary
mechanism for specializing a general-purpose model towards
a specific task. A pivotal breakthrough in this direction was to
enable LLMs to use external tools, as demonstrated by seminal
works like Toolformer [26], LaMDA [27] and ReAct [28]. This
allows an LLM to overcome its inherent knowledge limitations
by interacting with external software tools, such as invoking
a simulator, retrieving data from a database, or performing a
web search. By integrating an LLM with tools, it can ground
its reasoning in real-world data and act upon its environment,
forming the basis of an autonomous agent. Our work leverages
these foundational principles to construct a specialized agentic
workflow tailored specifically for the task of analog circuit
sizing.
B. The AnaFlow Agentic Workflow
In the AnaFlow agentic framework, specialized LLM agents
collaborate to mimic the structured reasoning process of an
expert analog designer. The complex sizing task is broken up
in a carefully orchestrated sequence of four phases (see Figure
2): an initial analysis phase to understand the circuit, fol-
lowed by three distinct, iterative loops for DC-operating-point-
(DC-OP)-focused sizing, reasoning-only sizing, and optimizer-
equipped sizing. The latter can call external optimizers like
BO or RL. This modular architecture ensures a reliable and
explainable automated path to optimized solutions. The details
of the overall workflow of Figure 2 will be discussed below
(see Figure 3).
B.1. Understanding the Circuit and Requirements
The workflow begins with a deep analysis of the sizing
problem, handled by a sequence of specialist agents that
deconstruct the task into manageable parts. Figure 3.1 shows
the context provided to each agent in this first phase:
• Circuit Explainer: this agent analyzes the circuit topol-
ogy given in the input SPICE netlist. It reasons about
all the connections of devices in the netlist, tries to find
different structures in the circuit and provide explanations
about the role of each device or structure in the topology.
• Matching Finder: this agent identifies devices in the
circuit that require precise matching or symmetry for
optimal performance, such as differential pairs or current
mirrors. It annotates these components, allowing the siz-
ing agents to apply constraints that preserve these critical
requirements during optimization.
• DC Goal Setter: this agent reads the information gath-
ered from the previous agents, reasons about them, and
finds the DC and operating-region requirements that each
transistor of the circuit needs to work in.
• Initial Designer: this agent represents the initial sizing
step. It synthesizes the information from the previous
agents and, based on the inherent knowledge of the LLM
about analog circuits, generates a plausible initial set of
device sizes that it think leads to meeting the required
specifications.
B.2. DC-OP-focused Sizing
Before committing to expensive performance simulations,
the workflow enters a preparatory second phase (see Figure 2)
to establish a reasonable DC biased solution. This is handled
by a specialized iterative loop, intentionally constrained to
a few cycles (∼5-10) to limit LLM calls and runtime. The
objective here is not to reach perfect DC convergence, but
rather to rapidly gather insights into the circuit’s behavior in
3

## Page 4

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
Fig. 3: Detailed diagram of all agents in the AnaFlow workflow with the four phases of Figure 2.
the target technology; much like a human expert’s initial steps.
Figure 3.2 shows the context of the agents in this phase:
• DC Reviewer: This agent takes care of the DC validation
process. It takes the current set of design parameters,
performs a fast DC operating point (.op) SPICE sim-
ulation, and parses the simulation output to understand
the operating region of each transistor. The agent then
compares these simulated results against the target DC
goals set before, identifying discrepancies and suggesting
reasoning-based overall modifications for resolving them.
• DC Sizer: this agent receives the discrepancy report
and the modification feedback from the DC Reviewer
and, based on the inherent knowledge of the LLM about
analog circuit behavior, determines the specific parameter
modifications to resolve the identified issues. Its output
is a structured output of parameter names and their new
refined values, which are fed back to the DC Reviewer
for the next validation cycle.
B.3. Reasoning-Only Sizing
After a few iterations of DC refinement, the workflow tran-
sitions to its first full performance optimization loop: checking
all required specifications through full simulation. This third
phase relies purely on the LLM’s inherent, learned knowledge
of analog circuit theory combined with the context gathered by
the previous agents to refine the design—no external numerical
optimizers are used yet in this phase. This stage emulates an
experienced designer applying his/her intuition and making
heuristic-driven adjustments. It is a sample-efficient loop,
designed to quickly find the best possible design that can be
found using solely the agent’s ability to reason about complex
circuit trade-offs before resorting to simulation-expensive op-
timizers (BO, RL). We limit this loop to a maximum of 20
full simulations. Figure 3.3 shows the context provided to the
agents in this phase:
• Specs Reviewer: This agent’s job is to carry out a
full performance validation and provide reasoning-based
feedback for modifications to improve the performance
metrics. It takes the latest sizing parameters and employs
a comprehensive simulation tool that executes more com-
plex analyses (e.g., .ac, .tran) to measure key perfor-
mance metrics like voltage gain, phase margin, unity-gain
bandwidth, and power consumption. The agent’s output
is a combination of critique and feedback, identifying the
most significant performance deviations to be addressed,
and providing insights and reasoning on which changes to
design parameters need to be made for improving those
performance metrics. This agent also has access to DC-
only simulation tool; it can call this when it feels that the
sizing is in a very bad condition or when it is confused
and needs cheap DC simulation insights. The choice of
which tool to use when is based on the agent’s own
judgment and reasoning.
• Reasoning Sizer: Acting on the feedback from the Specs
Reviewer and based on the inherent analog circuit knowl-
edge of the LLM as well as the optimization history, this
agent provides specific parameter modifications, delivered
as a structured output of new design parameter values for
the next simulation cycle.
B.4. Optimizer-equipped Sizing
When the pure reasoning-based loop in the third phase
reaches its iteration limit, the workflow escalates to the fourth
4

## Page 5

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
Fig. 4: Schematic of the fully differential folded-cascode
opamp (its CMFB is not shown), and user input for sizing
the circuit fully automatically in AnaFlow.
and final optimization stage. Here, the agents can call external
numerical optimizers (e.g., BO or RL) to handle particularly
challenging trade-offs or to overcome local minima that defy
heuristic adjustments. Figure 3.4 shows the context of the two
agents used here:
• Advisor Reviewer: This agent performs the same core
functions as in the reasoning-only phase, including full
performance validation and providing detailed critiques of
metric deviations and feedback for sizing adjustment. It
additionally monitors the optimization progress for signs
of stagnation, such as repeated patterns of diminishing
returns or failure to improve key metrics over several
cycles. When such conditions are detected, the agent ex-
plicitly advises to invoke an external optimizer, reasoning
about why it is needed. It retains access to the DC-only
simulation tool for quick checks.
• Equipped Sizer: Building directly on the feedback and
any optimizer advice from the Advisor Reviewer, this
agent evaluates whether to rely solely on its own LLM-
based reasoning for parameter refinements—much like
the Reasoning-Only Sizer—or to activate an external
optimizer tool for a more systematic search. When opting
for the optimizer, it configures the tool with initial points
(from the previous simulations and the optimization his-
tory) and decides the simulation budget for that run,
and finally integrates the optimizer’s suggestion into a
structured output of refined parameters.
IV. EXPERIMENTAL RESULTS
The AnaFlow agentic workflow is now demonstrated to size
two circuits with increasing levels of complexity: a two-stage
Miller opamp and a differential folded-cascode opamp with
common-mode feedback (CMFB).
Since we are using LLMs that run their inference in the
cloud, we use the open-source predictive BSIM 45nm PDK
[29] with NGSPICE [30] as simulator. Figure 4 shows the
schematic of the differential folded-cascode opamp and all
the entire information about this circuit given as input to the
framework, namely: the netlist, the sizing parameters and their
ranges, the required performance metrics, and the optimization
targets. As can be seen, minimal input needs to be given to
the workflow.
Our AnaFlow agentic framework is implemented in
Google’s ADK [31] which provides sequential agents, loop
agents, the use of tools, and state (memory) management
capabilities. It also provides the ability to use LLMs other
than Gemini, such as GPT, Claude, Grok, etc.
Below, we first provide the sample-efficiency results by
comparing our method to the previously published RL-based
analog sizing optimizer AnaCraft [2], [3]. Then, we show
examples of explanations and decisions made by the agents
throughout the steps of the sizing workflow, to demonstrate
the explainability capabilities of our framework.
A. Sample-Efficiency and Runtime Comparison
As the LLM agents in our framework guide the sizing
process through reasoning, it considerably requires less simu-
lations; simulations are either called by the LLM or through
the use of the external optimizer. The better the LLM agent
(the larger and smarter the model), the better it is in reasoning
and the more circuit knowledge it inherently has, leading to
even fewer external optimizer calls and less full simulations
in total.
On the other hand, smarter and larger LLM models take
more time to respond to each call. Therefore, although con-
siderable sample efficiency is gained by larger LLMs, they
reversely may lead to longer runtimes. This drawback is
less prominent for circuits with much longer full simulation
runtimes.
The results are shown in Table I. As can be seen, for
the two-stage opamp, Gemini 2.5 Pro can find a sizing
solution that satisfies all hard performance constraints in the
specifications within 10 iterations of full simulations and
reasoning, without having to call the external optimizer. For
the more complex differential folded-cascode opamp, it can
find a solution after only 1 call to the external optimizer —the
single-agent version of the RL-based AnaCraft algorithm [2],
[3]— and still requires to a total of less than only 100
simulations. However, these results do come at the price of a
longer runtime and a higher cost per token. Faster LLM models
like Gemini 2.5 Flash, Claude Sonnet 4, and Grok 3 require
more reasoning steps, more calls to the external optimizer, and
therefore, a higher number of simulations, but their overall
runtime is better, as evidenced in Table I.
We experienced two main problems with some of the LLM
models used in our experiments; 1) complying to a long
structured output for a simulation call, and 2) very long
runtimes. For example, the first problem caused some models
to fail for the more complex folded-cascode opamp as the
circuit has many more parameters and therefore a longer and
more sophisticated structured output which those models failed
to provide at each call to the simulator. On the other hand,
some larger models needed significantly longer runtimes while
not bringing any sample-efficiency benefits, whlie others had
such a long runtime that they could not reach any significant
step within 2 hours while having a very high cost per token.
Figure 5 shows a runtime comparison of the LLM-based
AnaFlow framework (with Gemini 2.5 Pro as the LLM) and
5

## Page 6

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
TABLE I: Comparing the agentic workflow (using different
LLMs) to RL based on finding the first sizing solution.
Circuit Approach
LLM
Calls
Opt.
Calls
DC
Sims.
Full Simulations Time
(min)
LLM Opt. Total
Two-
Stage
Amp.
Gemini 2.5 Flash 81 5 5 32 250 282 16
Gemini 2.5 Pro 34 0 6 9 0 9 24
GPT o3 68 4 7 25 180 205 46
Claude Sonnet 4 52 6 24 19 300 319 17
Claude Opus 4 36 1 16 11 50 61 27
Grok 3 70 7 8 28 270 298 18
AnaCraft [2], [3] N/A 1 N/A N/A 1035 1035 14
Diff.
Folded
Cascode
Amp.
Gemini 2.5 Flash 50 8 9 18 300 318 17
Gemini 2.5 Pro 42 1 6 14 50 64 29
GPT o3 110 8 23 48 312 360 82
Grok 3 116 7 15 46 280 316 22
AnaCraft [2], [3] N/A 1 N/A N/A 1808 1808 31
the single-agent AnaCraft RL algorithm of [2], [3] in reaching
an optimized solution with a Figure of Merit (FoM) of above
0.5 for the differential folded-cascode opamp with CMFB (the
FoM is calculated according to [2], [3]). It can be seen that
LLM-based workflows like AnaFlow today still need similar
runtimes as the state-of-the-art RL-based algorithms.
B. Explainability of the Workflow
In order to demonstrate the explainability of the decisions
made by our agentic workflow, we show some examples of
reasoning and explanation provided by the LLM agents in
Figure 6. As shown, the LLM (Gemini 2.5 Pro in these figures)
can explain for example: what causes a DC failure and which
design parameter to change for fixing it; how do changes to
channel lengths cause changes in the Gain and Slew Rate,
and which sizing parameters need to change to achieve the
target goals for those performance metrics. It can also decide
that its own reasoning (which it calls manual adjustment)
is stuck and that it advises to run the external optimizer.
It can even learn lessons from the output of the optimizer
and try to make sense of it after observing its actions and
the corresponding outcomes (through reasoning based on the
LLM’s inherent knowledge of analog circuits). The workflow’s
answers provide insightful trade-off analysis and can explain
the balance struck between conflicting performance metrics
like gain, stability, and speed.
This capability is unprecedented and could not be seen in
any of the analog sizing optimization tools published before.
Hence, using the AnaFlow framework, designers can better
understand the sizing choices and get more insight from the
automated sizing process. Designers have access to traceable
design decisions, can learn from them, or criticize them.
V. CONCLUSIONS
This paper has introduced the AnaFlow agentic LLM work-
flow for reasoning-driven, sample-efficient, and explainable
analog circuit sizing. The framework employs a collaborative
multi-agent LLM-based workflow that mimics the cognitive
process of expert human designers. The structured approach,
which decomposes the sizing task into analysis, DC refine-
ment, and heuristic and optimization-based refinement steps,
Fig. 5: Runtime comparison of AnaFlow and AnaCraft [2], [3]
in reaching optimized design solutions.
brings human-interpretable reasoning into the design process.
The experimental results have shown a large reduction in the
Fig. 6: Illustrative examples of the LLM agents’ explanations
in direct reasoning and problem diagnosis.
6

## Page 7

Accepted for 2025 IEEE/ACM International Conference On Computer-Aided Design (ICCAD)
number of simulations required compared to straightforward
optimization. The framework’s key contribution is its inherent
explainability, providing transparent justifications for the de-
sign choices and fostering trust in the generated solutions. This
work represents a significant step towards a new paradigm in
analog EDA, where AI agents act not as opaque optimizers,
but as transparent design assistants that accelerate the design
cycle while augmenting the designer’s own understanding.
ACKNOWLEDGMENT
This work has been funded by the ERC Advanced Grant
AnalogCreate (n° 101019982).
REFERENCES
[1] M. F. M. Barros, J. M. C. Guilherme, and N. C. G. Horta, Analog
Circuits and Systems Optimization Based on Evolutionary Computation
Techniques. Springer, vol. 9, 2010.
[2] M. Ahmadzadeh and G. Gielen, “Using Probabilistic Model Rollouts to
Boost the Sample Efficiency of Reinforcement Learning for Automated
Analog Circuit Sizing,” in Proceedings of the 61st ACM/IEEE Design
Automation Conference (DAC), 2024, pp. 1–6.
[3] M. Ahmadzadeh, J. Lappas, N. Wehn and G. Gielen, "AnaCraft: Duel-
Play Probabilistic-Model-based Reinforcement Learning for Sample-
Efficient PVT-Robust Analog Circuit Sizing Optimization," in IEEE
Transactions on Computer-Aided Design of Integrated Circuits and Sys-
tems, Early Access 2025, doi: 10.1109/TCAD.2025.3582175.
[4] G. Wolfe and R. Vemuri, “Extraction and use of neural network models
in automated synthesis of operational amplifiers,” in IEEE Transactions
on Computer-Aided Design of Integrated Circuits and Systems, vol. 22,
no. 2, pp. 198–212, 2003.
[5] T. Pessoa, N. Lourenço, R. Martins, R. Póvoa and N. Horta, "En-
hanced analog and RF IC sizing methodology using PCA and NSGA-
II optimization kernel," in 2018 Design, Automation & Test in Europe
Conference & Exhibition (DATE), Dresden, Germany, 2018, pp. 660-665,
doi: 10.23919/DATE.2018.8342092.
[6] W. Lyu, F. Yang, C. Yan, D. Zhou, and X. Zeng, “Batch Bayesian opti-
mization via multi-objective acquisition ensemble for automated analog
circuit design,” in Proceedings of the 35th International Conference on
Machine Learning, pp. 3306–3314, 2018.
[7] B. Liu, D. Zhao, P. Reynaert, and G. G. E. Gielen, “GASPAD: A
general and efficient mm-wave integrated circuit synthesis method based
on surrogate model assisted evolutionary algorithm,” IEEE Transactions
on Computer-Aided Design of Integrated Circuits and Systems, vol. 33,
no. 2, pp. 169–182, 2014.
[8] H. Wang, J. Yang, H.-S. Lee, and S. Han, “Learning to design circuits,”
arXiv preprint arXiv:1812.02734, 2018.
[9] H. Wang, K. Wang, J. Yang, L. Shen, N. Sun, H.-S. Lee, and S. Han,
“GCN-RL circuit designer: Transferable transistor sizing with graph
neural networks and reinforcement learning,” in 2020 57th ACM/IEEE
Design Automation Conference (DAC), pp. 1–6, 2020.
[10] K. Settaluri, A. Haj-Ali, Q. Huang, K. Hakhamaneshi, and B. Nikolic,
“AutoCkt: Deep reinforcement learning of analog circuit designs,” in 2020
Design, Automation & Test in Europe Conference & Exhibition (DATE),
pp. 490–495, 2020.
[11] N. S. K. Somayaji, H. Hu, and P. Li, “Prioritized reinforcement learning
for analog circuit optimization with design knowledge,” in 2021 58th
ACM/IEEE Design Automation Conference (DAC), pp. 1231–1236, 2021.
[12] W. Shi, H. Wang, J. Gu, M. Liu, D. Z. Pan, S. Han, and N. Sun,
“RobustAnalog: Fast variation-aware analog circuit design via multi-
task rl,” in Proceedings of the 2022 ACM/IEEE Workshop on Machine
Learning for CAD, pp. 35–41, 2022.
[13] M. Choi, Y. Choi, K. Lee, and S. Kang, “Reinforcement Learning-based
Analog Circuit Optimizer using g m/I D for Sizing,” in Proceedings of the
60th ACM/IEEE Design Automation Conference (DAC), pp. 1–6, 2023.
[14] K.-E. Yang, C.-Y. Tsai, H.-H. Shen, C.-F. Chiang, F.-M. Tsai, C.-
A. Wang, Y. Ting, C.-S. Yeh, and C.-T. Lai, “Trust-region method
with deep reinforcement learning in analog design space exploration,”
in Proceedings of the 58th ACM/IEEE Design Automation Conference
(DAC), pp. 1225–1230, 2021.
[15] J. Zhang, J. Bao, Z. Huang, X. Zeng, and Y. Lu, “Automated De-
sign of Complex Analog Circuits with Multiagent based Reinforcement
Learning,” in Proceedings of the 60th ACM/IEEE Design Automation
Conference (DAC), pp. 1–6, 2023.
[16] Xi, Z., Chen, W., Guo, X. et al., “The rise and potential of large language
model based agents: a survey,” Sci. China Inf. Sci., 68, 121101 (2025).
https://doi.org/10.1007/s11432-024-4222-0
[17] Wang, L., Ma, C., Feng, X. et al., “A survey on large language model
based autonomous agents,” Front. Comput. Sci., 18, 186345 (2024).
https://doi.org/10.1007/s11704-024-40231-1
[18] G. G. E. Gielen and R. A. Rutenbar, "Computer-aided design of analog
and mixed-signal integrated circuits," in Proceedings of the IEEE, vol.
88, no. 12, pp. 1825-1854, Dec. 2000.
[19] F. El-Turky and E. E. Perry, "BLADES: an artificial intelligence ap-
proach to analog circuit design," in IEEE Transactions on Computer-
Aided Design of Integrated Circuits and Systems, vol. 8, no. 6, pp. 680-
692, June 1989.
[20] B. Shahriari, K. Swersky, Z. Wang, R. P. Adams, and N. De Freitas,
“Taking the human out of the loop: A review of Bayesian optimization,”
Proceedings of the IEEE, vol. 104, no. 1, pp. 148–175, 2015.
[21] P. I. Frazier, “A tutorial on Bayesian optimization,” arXiv preprint
arXiv:1807.02811, 2018.
[22] P. Schmid, "The New Skill in AI is Not Prompting, It’s Context
Engineering," philschmid.de, Jun. 30, 2025.
[23] Gao, Jian, Weidong Cao, Junyi Yang, and Xuan Zhang. "AnalogGenie:
A generative engine for automatic discovery of analog circuit topologies."
arXiv preprint arXiv:2503.00205, 2025.
[24] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, & P. Luo,
"AnalogCoder: Analog Circuit Design via Training-Free Code Genera-
tion," in Proceedings of the AAAI Conference on Artificial Intelligence,
39(1), 379-387. https://doi.org/10.1609/aaai.v39i1.32016, 2025.
[25] Y. Yin, Y. Wang, B. Xu, and P. Li, "ADO-LLM: Analog design Bayesian
optimization with in-context learning of large language models," arXiv
preprint arXiv:2406.18770, 2024.
[26] Schick, T., Dwivedi-Yu, J., Dessì, R., Raileanu, R., Lomeli, M., Ham-
bro, E., Zettlemoyer, L., Cancedda, N. and Scialom, T., "Toolformer:
Language models can teach themselves to use tools," in Advances in
Neural Information Processing Systems, 36, pp.68539-68551, 2023.
[27] Thoppilan, Romal, Daniel De Freitas, Jamie Hall, Noam Shazeer,
Apoorv Kulshreshtha, Heng-Tze Cheng, Alicia Jin et al. "Lamda: Lan-
guage models for dialog applications." arXiv preprint arXiv:2201.08239,
Jan, 2022.
[28] Yao, Shunyu, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik
Narasimhan, and Yuan Cao. "ReAct: Synergizing reasoning and acting in
language models." In International Conference on Learning Representa-
tions (ICLR). 2023.
[29] W. Zhao and Y. Cao, “New generation of predictive technology model
for sub-45 nm early design exploration,” IEEE Transactions on Electron
Devices, vol. 53, no. 11, pp. 2816–2823, 2006.
[30] H. Vogt, G. Atkinson, P. Nenzi, and D. Warning, Ngspice User’s Manual
Version 43 (Ngspice Release Version), [Online]. Available: https://ngspice.
sourceforge.io/docs/ngspice-43-manual.pdf, 2024.
[31] E. Huizenga and B. Yang, "Agent Development Kit: Making it easy
to build multi-agent applications," Google for Developers Blog, Apr. 09,
2025.
7

