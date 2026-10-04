---
title: "An LLM-assisted Analog IC Design Tool with Automatic Topology Selection and Circuit Sizing"
authors: "Anh Nguyen; Chien-Nan Jimmy Liu"
date: "2025-07-07"
category: "end_to_end"
source_url: "https://doi.org/10.1109/SMACD65553.2025.11091899"
pdf: "papers/2025_llm-assisted-analog-ic-design-tool_10-1109-smacd65553-2025-11091899.pdf"
sha256: "47ef012fa84825ac96efc5ff37bf45ed2bd8709f969c4b7ab24f26470fe511e4"
pages_extracted: 4
---

# An LLM-assisted Analog IC Design Tool with Automatic Topology Selection and Circuit Sizing

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

An LLM-assisted Analog IC Design Tool with
Automatic Topology Selection and Circuit Sizing
Anh Nguyen, Chien-Nan Jimmy Liu
{EECS International Graduate Program, Institute of Electronics}
National Yang-Ming Chiao Tung University
Hsinchu City, Taiwan, R.O.C.
{nta.ee12, jimmyliu}@nycu.edu.tw
Abstract—Recently, Large Language Models (LLMs) have
gained a rising interest in different domains, including code
generation for digital IC design. However, building an effective
LLM for analog IC design is challenging because a valid analog
circuit cannot be constructed by just randomly combining some
codes. In this paper, an LLM-assisted analog IC design tool is
presented to support from automatic topology selection to
circuit sizing in a conversational manner. This tool allows
designers to express their requirements through natural
language rather than being restricted by a fixed template, which
decreases the learning curve of using such an automation tool.
In order to meet users’ intentions, a subsequent ANN-based
sizing process is invoked to further refine the “nearest” circuit
obtained from the LLM without tedious iterations. Compared
with previous works, the proposed LLM-assisted tool is more
efficient and convenient while generating reliable analog designs
to meet users’ requirements. Our experiments show that the tool
can generate workable circuits with short execution time.
Keywords—LLM, Zero-shot-CoT prompting, kNN search,
automatic circuit sizing, ANN, circuit topology selection
I. INTRODUCTION
Because traditional analog IC design flow heavily relies
on past knowledge and intuition, analog design automation
has been studied for years to alleviate the long design cycles
[1]. Most of existing works require a given circuit topology
from designers, which still needs some prior knowledge to
make a proper selection. A topology searching method in
collaboration with a subsequent sizing process performed on
a pre-built circuit library was proposed [4]. However, using
decision tree traversal for topology selection often results in a
long execution time when the number of candidates increases
[4][14]. Further, [4] cannot take advantage of massive
parallelism of graphics processing units (GPUs). Another
approach uses artificial neural networks (ANNs) to merge
topology selection and circuit sizing into one step, thus
reducing the overall execution time [15]. Nevertheless, due to
the uncertainty of ANN classification, inconsistent topology
selection can occur under the same specification when users
query at different time. This can be further exacerbated as the
number of classification nodes increases. Therefore, it is
important to find a good tradeoff between execution time and
accuracy to be adopted by industrial designers.
Large Language Models (LLMs) have recently gained
attention in Design Automation field because they allow IC
designers to “discuss with an assistant” to construct a circuit.
Thus, they can enable IC design tasks to become more easy
and convenient without too much prior training. Despite a
significant progress made in digital domain [2][13], the
potential of LLMs in analog domain remains underdeveloped.
Digital IC design automation can be thought of as a process of
code generation and evaluation, safely ignoring effect of
transistor sizes. However, due to the multifaceted constraints
of analog ICs, building an effective LLM for analog IC design
is more challenging because a valid analog circuit cannot be
constructed by just randomly combining some codes. A recent
work trains an LLM from both valid and invalid circuits to
generate different topologies for analog designs with random
performance [3]. In this approach, the randomly generated
circuits are not guaranteed to be workable under the given
specification, which is not acceptable for industrial designers.
In practical applications, a reliable design tool is often desired.
To utilize previous design experience, combining LLM with a
trustworthy circuit library might be a good choice to help
analog circuit design automation.
In this paper, we present an LLM-assisted analog IC (AIC)
design tool that supports from automatic topology selection to
circuit sizing in a conversational manner. Designers can
express their requirements through natural language rather
than being restricted by a fixed template, which decreases the
learning curve of using such an automation tool. Based on an
extendable circuit library, the proposed LLM-based topology
selection is guided by Zero-shot-Chain-of-Thought (Zero-
shot-CoT) prompting and k-nearest neighbors (kNN) search
to achieve an extremely short execution time on getting a
feasible solution from the trustworthy candidates. This
approach is able to maintain reliable solutions at each usage
iteration, which is more acceptable by analog designers.
However, it is impossible to prepare all solutions in advance
to meet any user’s request. Therefore, to meet the user’s
intentions, a subsequent ANN-based sizing process is invoked
to further refine the “nearest” circuit obtained from the library
without tedious iterations. Compared with previous analog
design automation approaches, the proposed LLM-assisted
tool is more efficient and convenient while generating reliable
analog designs to meet users’ requirements.
II. LLM-ASSISTED AIC DESIGN METHODOLOGY
A. Workflow of the LLM-assisted AIC Design Tool
Fig. 1 illustrates the overall workflow of this tool. Starting
from an input query, an LLM-based topology selection
module finds a best-matched design from the given circuit
library in terms of circuit performance. The topology and
corresponding device sizes of this design will form the design
context to answer the user’s query. If the question requires
some details of past conversations, the historical context is
formed by retrieving relevant information from the chat log,
which is then concatenated with the design context to form the
total context for the final answer. This final answer will also
be saved to the chat log for future use. Finally, if the best-
matched design does not completely meet users’ intentions, an
ANN-based sizing help to refine the answer without tedious
iterations. The following sections explain each component.
B. Graphic User Interface (GUI)
Fig. 2 shows the GUI of the tool. There are 5 boxes for
different purposes: inputting a user’s query, receiving an
answer, printing a best-matched design, displaying a netlist,
and displaying a schematic. To perform different functions,
2025
21st
International
Conference
on
Synthesis,
Modeling,
Analysis
and
Simulation
Methods,
and
Applications
to
Circuits
Design
(SMACD)
|
979-8-3315-2395-4/25/$31.00
©2025
IEEE
|
DOI:
10.1109/SMACD65553.2025.11091899
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:33 UTC from IEEE Xplore. Restrictions apply.

## Page 2

there are 5 buttons in total: submitting a user’s query,
generating a best-matched design, activating the netlist and
schematic display, performing a subsequent sizing, creating a
simulation folder.
C. Trustworthy Circuit Library
There are 12 circuit “feature names” in our trustworthy
circuit library: gain, phase margin, unity frequency,
bandwidth, dynamic power (with a 100kHz square wave
input), static power, slew rate, estimated area, common mode
rejection ratio (CMRR), power supply rejection ratio (PSRR),
bias current, capacitance load. The library consists of 25
commonly used operational transconductance amplifiers
(OTAs) and operational amplifiers (OPAMPs). Because each
row in the library pertains to a vector including specifications
of all feature names with corresponding device sizes and
device values of a particular topology, the term “Specification
Table” is used in Fig. 1.
Fig. 1. The tool’s workflow.
Fig. 2. The tool’s GUI.
D. Three-Stage Conversational Topology Selection
The topology selection module is conceptually divided
into 3 stages. The module utilizes a general-purpose LLM,
namely Llama 3.1 8B Instruct [5], in collaboration with our
searching and prompting techniques, to quickly opt for a
circuit topology in a conversational manner. Three stages are
sequentially called “Summarization”, “Retrieving k Circuits”
and “Choosing Best-Matched”, as shown in Fig. 3.
Fig. 3. Flowchart of the topology selection module.
In the “Summarization” stage, if T numbers of feature
names are queried, a “Summarization” prompt automatically
constructs T questions, each starts with a sentence “what is the
required value of”, followed by a feature name fc. The
sentence “You have to answer in bullet points” also needs to
be present. The LLM takes the user’s query and passes it
through the “Summarization” prompt to generate a
“Summary” in which the queried parameters are always
arranged in the same order. This allows users to query about
any circuit parameter in any order based on their thought
processes. Furthermore, due to the LLM’s capability to handle
spelling mistakes, incorrect feature names can be mended so
that they are consistent with the circuit library. Fig .4
illustrates the process of making a “Summary” from a query.
Fig. 4. The process of making a “Summary” from a query.
The “Summary” is fed to the “Retrieving k Circuits” stage.
Firstly, a regular expression parses the “Summary” to obtain a
“Performance Mapping Sequence” denoted by:
Sperf = {v1(f1), v2(f2), …, vT(fT)} (1)
where v1(f1) is the first feature’s value, v2(f2) is the second
feature’s value, etc. The final feature’s value is vT(fT) if there
are T queried feature names. Secondly, a kNN search [7] is
performed on the library to retrieve k nearest data vectors of
the Sperf. Empirically, we set hyperparameter k=4 and utilize
Euclidean Distance Metric in our tool. Thirdly, an “Unrolling”
procedure transforms the k vectors into k paragraphs by
starting each paragraph with our customized phrase “design
log”, followed by a row index i, a “:” symbol, a feature name
fc, a “=” symbol, and a feature value vc(fc). It makes a series
of T descriptive pairs for each of the k resultant paragraphs.
The kNN search is illustrated in Fig. 5. An example of the
“Unrolling” procedure is illustrated in Fig. 6.
Fig. 5. kNN search illustration.
Fig. 6. An example of the “Unrolling” procedure.
In the “Choosing Best-Matched” stage, we employ Zero-
shot-CoT prompting [6] by adding “Let’s think step by step”
in our “Best-Matched” prompt. In [6], experiments show that
by adding the sentence, LLMs are forced to divide individual
problems into several steps to provide individual answers. In
our work, the specific problem is to compare each feature
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:33 UTC from IEEE Xplore. Restrictions apply.

## Page 3

value vc(fc) in each of the k paragraphs with each feature value
in the Sperf to opt for a best-matched design. As can be seen
from Fig.7, the “Best-Matched” prompt is constructed by a
concatenation of a fixed paragraph called “Request 1”, the k
circuit paragraphs, another fixed paragraph called “Request
2”, and the original user’s query. As illustrated in Fig. 7, The
LLM in our tool takes the “Best-Matched” prompt as an input
to produce the best-matched design, which is a circuit with
device sizes that satisfies a query. This design is also the
output of the topology selection module.
Fig. 7. Struture and functionality of the “Best-Matched” prompt.
The purpose of the kNN search is to return several designs
with shortest distances from a Sperf. It is more practical than
returning only one vector because there would be cases where
the design that aligns with a user’s intentions lies in a vector
that is further away from a specific Sperf than the closest vector.
For example, a vector V1={40dB,15kHz} is further away from
a Sperf ={38dB,17kHz} than a vector V2={37dB,18kHz} but
the query contains verbal conditions such as “gain>38dB”.
Thus, Zero-shot-CoT prompting can collaborate with kNN
search to obtain a more aligned result in such case.
E. Automatic Circuit Sizing
When a retrieved circuit does not meet the requirement, a
subsequent sizing can be performed. ANN-based sizing is
suitable in our case because of 2 reasons. Firstly, data from the
library is reused in our training process. Secondly, because no
new topology is added in the current version of the tool, sizing
an existing circuit with a predetermined target from a query
can take advantage of the extremely fast inference time of
ANNs. We adopt the sizing method featuring ANNs from [8]
with 2 major changes. In [8], error metrics are mean absolute
error (MAE) and mean squared error (MSE). Both MSE and
MAE decrease in the training process to approach zero
without definitive ratios. Thus, R-Square Score (R2-score) [9]
is an addition in our work to increase the interpretability of the
results. As MSE decreases, R2-score approaches 1.0 (a ratio),
meaning that the error between actual and predicted data is
gradually minimized in training and evaluation. Another
departure from [8] is that polynomial features are not utilized
because we have not observed any improvement as compared
to the case where only original data are used.
F. Information Retrieval from Chat Log
A chat log is integrated so that users can query about the
conversation history. To achieve the correct contexts from the
chat log, instead of relying on dense retrieval [10], we employ
a sparse retriever based on BM25 algorithm [11] for 2 reasons.
Firstly, as the log becomes lengthier, creating a dense
embedding vector of the whole log is significantly slower than
calculating BM25 index. Secondly, sparse retrieval permits
exact term matches which is important in searching for
technical terms. This is pointed out in [12]. A hybrid retriever,
a combination of a sparse retriever and a dense retriever, can
also be discarded due to its slow runtime.
III. EXPERIMENTAL RESULTS
In Section III, the topology selection module takes TkNN to
retrieve k circuits and takes TBEST to choose the best-matched
design. Circuit areas are estimated without routing. In Section
III, “N.T.” stands for “Number of Topologies" and “I.S.M.”
stands for “Input Spelling Mistakes”. Circuit simulations are
performed by HSPICE with TSMC 0.18um process.
A. Case study 1: Circuit Retrieval with Performance Match
In case study 1, the required OTA with existing
performance and device sizes in the library is a simple 1-stage
OTA (single-stage differential amplifier). Therefore, extra
sizing is not required in this case, whose execution time can
be denoted as TEXE-CASE1 = TkNN + TBEST. We repeat the
experiment 5 times and record the longest TEXE-CASE1 to
compare with execution time in [4]. While [4] takes 11 min 27
sec to execute, our tool only requires 1 min 12 sec on a
NVIDIA L4 GPU, as shown in Table II. The question-
answering process of case study 1 is illustrated in Fig. 8 and
Fig. 9. The word “phs margin” is corrected as “phase margin”.
The result of the retrieved circuit is summarized in Table I.
Fig. 8. Inputting a query into our tool in case study 1.
Fig. 9. Main parts of a Best-Matched Design from our tool in case study 1.
B. Case study 2: Circuit Retrieval and Subsequent Sizing
In case study 2, the required circuit performance has no
matching data in the library. The nearest sample is chosen by
the kNN algorithm, whose performance does not meet the
expected values. Therefore, a subsequent sizing is invoked
that lasts for TSIZE. The confirming simulation lasts for TSIM.
As a result, the execution time can be denoted as TEXE-CASE2 =
TkNN + TBEST + TSIZE + TSIM.
TABLE I. PERFORMANCE RESULT OF THE RETRIEVED 1-STAGE OTA IN CASE STUDY 1
Para-
meters
Gain
Estimat-
ed Area
Phase
Margin
(degree)
Unity
Frequency
Band-
width
Dynamic
Power
Static
Power
Slew
Rate
/
CMRR PSRR
Bias
Current
Capacitance
Load
Values 35.21 1064.80 83.66 9.66 171.15 154.98 99.99 10.65 40.69 27.13 100.00 10.00
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:33 UTC from IEEE Xplore. Restrictions apply.

## Page 4

TABLE II. COMPARISON OF CASE STUDIES IN OUR WORK WITH [4]
Method
Execution
Time
N.T. Tool Interface I.S.M.
[4] 11 min 27 sec 24 Fixed Template N/A
Case Study 1 1 min 12 sec 25 Conversational Allowed
Case Study 2 2 min 33 sec 25 Conversational Allowed
We repeat the experiment 5 times and record the longest
TEXE-CASE2 to compare with execution time in [4]. While [4]
takes 11 min 27 sec to execute, our tool only requires 2 min
33 sec on a NVIDIA L4 GPU, as shown in Table II. The
question-answering process of case study 2, that handles
I.S.M. of our tool, is illustrated in Fig. 10 and Fig. 11.
Fig. 10. Inputting a query into our tool in case study 2.
Fig. 11. Main parts of a Best-Matched Design from our tool in case study 2.
TABLE III. PERFORMANCE OF RETRIEVED CIRCUIT IN CASE STUDY 2
Parameters Expected Best-Matched After-Sized
Gain 56.00 48.66 57.28
Phase Margin 60.00 70.13 61.24
Unity Frequency 35.00 90.89 39.19
Bandwidth 50.00 350.58 56.53
Dynamic Power 1200.00 1111.00 1104.40
Static Power 400.00 398.84 398.81
Slew Rate / 40.00 91.28 68.34
Bias Current 400.00 400.00 400.00
Capacitance Load 5.00 5.00 5.00
Estimated Area 2850.00 2779.20 2841.86
The retrieved circuit, shown in Fig. 12, is a 2-stage class
AB OPAMP. Table III illustrates the expected performance,
best-matched performance and performance obtained after the
circuit is sized. The after-sized performance become closer
and slightly more optimized than the expected values. This
demonstrates the capability of our tool to meet the intentions
of users even if the design does not exist in the library.
Fig. 12. The 2-stage classAB OPAMP displayed by the tool in case study 2.
IV. CONCLUSION
In this paper, we present an LLM-assisted AIC design tool
that achieves a competitive execution time to generate a
workable circuit from a query. Future work would be to
include an autoregressive circuit generation engine to enhance
the capability of the tool.
REFERENCES
[1] R. A. Rutenbar, G. G. E. Gielen, and J. Roychowdhury, “Hierarchical
Modeling, Optimization, and Synthesis for System-Level Analog and
RF Designs,” Proceedings of the IEEE, vol. 95., no. 3, pp. 640-669,
April 2007.
[2] B. Nadimi and H. Zheng, “A Multi-Expert Large Language Model
Architecture for Verilog Code Generation,“ IEEE LLM Aided Design
Workshop (LAD), pp. 1–5, June 2024.
[3] P. Vijayaraghavan, L. Shi, E. Degan, and X. Zhang,
“CIRCUITSYNTH: Leveraging Large Language Models for Circuit
Topology Synthesis,“ IEEE LLM Aided Design Workshop (LAD), pp.
1–6, June 2024.
[4] A. Gerlach, J. Scheible, T. Rosahl, and F. Eitrich, “A generic topology
selection method for analog circuits with embedded circuit sizing
demonstrated on the OTA example,“ Design, Automation and Test in
Europe Conference and Exhibition (DATE), March 2017.
[5] Aaron Grattafiori et al, “The Llama 3 Herd of Models,” 2024,
arXiv:2407.21783. [Online]. Available:
https://arxiv.org/abs/2407.21783
[6] T. Kojima, S. S. Gu, M. Reid, Y. Matsuo, and Y. Iwasawa, “Large
Language Models are Zero-Shot Reasoners,“ Proceedings of the 36th
International Conference on Neural Information Processing Systems,
pp. 22199–22213, April 2024.
[7] N. Ukey et al, “Survey on Exact kNN Queries over High-Dimensional
Data Space,“ Sensors, Jan. 2023.
[8] N. Lourenço et al, “On the Exploration of Promising Analog IC
Designs via Artificial Neural Networks,“ International Conference on
Synthesis, Modeling, Analysis and Simulation Methods and
Applications to Circuit Design (SMACD), July 2018.
[9] T. O. Kvålseth, “Cautionary Note about R2,” The American
Statistician, vol. 39, no. 4, part 1, pp. 279-285, Nov. 1985.
[10] V. Karpukhin et al, “Dense Passage Retrieval for Open-Domain
Question Answering,” 2020, arXiv: 2004.04906. [Online]. Available:
https://arxiv.org/abs/2004.04906
[11] S. Robertson and H. Zaragoza, “The Probabilistic Relevance
Framework: BM25 and Beyond,“ Foundations and Trends in
Information Retrieval, vol. 3, no. 4, pp. 333 – 389, April 2009.
[12] L. Shi, M. Kazda, B. Sears, N. Shropshire, and R. Puri, “Ask-EDA: A
Design Assistant Empowered by LLM, Hybrid RAG and Abbreviation
De-hallucination,“ IEEE LLM Aided Design Workshop (LAD), pp. 1–
5, June 2024.
[13] S. Liu et al,“RTLCoder: Outperforming GPT-3.5 in Design RTL
Generation with Our Open-Source Dataset and Lightweight Solution,“
IEEE LLM Aided Design Workshop (LAD), pp. 1–5, June 2024.
[14] T. McConaghy, P. Palmers, G. Gielen, M. Steyaert, “Automated
Extraction of Expert Knowledge in Analog Topology Selection and
Sizing,” IEEE International Conference on Computer-Aided Design,
Nov. 2008.
[15] J. P. S. Rosa, D. J. D. Guerra, N. C. G. Horta, R. M. F. Martins, and
N. C. C. Lourenço, “Using Artificial Neural Networks for Analog
Integrated Circuit Design Automation,” 1st ed., Springer, 2020, pp. 53-
57.
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:33 UTC from IEEE Xplore. Restrictions apply.

