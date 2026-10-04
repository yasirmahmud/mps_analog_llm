---
title: "A Large Language Model-based Multi-Agent Framework for Analog Circuits' Sizing Relationships Extraction"
authors: "Liu, Chengjie; Chen, Weiyu; Xu, Huiyao; Du, Yuan; Yang, Jun; Du, Li"
date: "2025/06/23"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2506.18424"
pdf: "papers/2025_a-large-language-model-based-multi-agent-framework-for-analog-circuits-sizi_2506-18424.pdf"
sha256: "f0eb53a6c51e331b4062e0566d732589107b45cfe0cdbb5591283a5fbb116d34"
pages_extracted: 7
---

# A Large Language Model-based Multi-Agent Framework for Analog Circuits' Sizing Relationships Extraction

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

arXiv:2506.18424v1
[cs.AI]
23
Jun
2025
A Large Language Model-based Multi-Agent
Framework for Analog Circuits’ Sizing
Relationships Extraction
Chengjie Liu
School of Electronic Science
and Engineering, Nanjing University
National Center of Technology
Innovation for EDA
Nanjing, China
cjliu phd@smail.nju.edu.cn
Weiyu Chen
School of Electronic Science
and Engineering, Nanjing University
National Center of Technology
Innovation for EDA
Nanjing, China
wychen@smail.nju.edu.cn
Huiyao Xu
Nanjing Tech University
National Center of Technology
Innovation for EDA
Nanjing, China
202121018094@njtech.edu.cn
Yuan Du
School of Electronic Science
and Engineering, Nanjing University
Nanjing, China
yuandu@nju.edu.cn
Jun Yang
School of Integrated Circuits
Southeast University
National Center of Technology
Innovation for EDA
Nanjing, China
dragon@seu.edu.cn
Li Du
School of Electronic Science
and Engineering, Nanjing University
Nanjing, China
ldu@nju.edu.cn
Abstract—In the design process of the analog circuit pre-layout
phase, device sizing is an important step in determining whether
an analog circuit can meet the required performance metrics.
Many existing techniques extract the circuit sizing task as a math-
ematical optimization problem to solve and continuously improve
the optimization efficiency from a mathematical perspective. But
they ignore the automatic introduction of prior knowledge, fail
to achieve effective pruning of the search space, which thereby
leads to a considerable compression margin remaining in the
search space. To alleviate this problem, we propose a large
language model (LLM)-based multi-agent framework for analog
circuits’ sizing relationships extraction from academic papers.
The search space in the sizing process can be effectively pruned
based on the sizing relationship extracted by this framework.
Eventually, we conducted tests on 3 types of circuits, and the
optimization efficiency was improved by 2.32 ∼ 26.6×. This
work demonstrates that the LLM can effectively prune the search
space for analog circuit sizing, providing a new solution for the
combination of LLMs and conventional analog circuit design
automation methods.
Index Terms—Analog circuits, Computer-aided design, Elec-
tronic design automation, Large language model, Optimization
algorithm, Search Space Pruning
I. INTRODUCTION
Analog circuits are an essential component of integrated
circuits (ICs). Its design process consists of the pre-layout
and layout phases. The sizing process in its pre-layout phase
requires manual repetitive parameter tuning to achieve higher
circuit performance as much as possible, resulting in a con-
siderable consumption of repetitive manpower. To address this
*Li Du is the corresponding author.
issue, many works have implemented automatic parameter
generation or optimization through deep learning [1], rein-
forcement learning [2], or optimization algorithms [3]–[5].
Among them, the optimization algorithm transforms the
sizing problem of analog circuits into a black-box optimization
problem. The goal of optimization algorithms is to achieve
higher optimization efficiency and accuracy rate for complex
circuits with more devices from a mathematical perspective.
However, analog circuits are not completely black-box opti-
mization problems. Introducing knowledge related to analog
circuits into the optimization process to provide optimization
directions or pruning the search space can be effectively com-
bined with existing optimization algorithms, thereby further
improving the efficiency of the optimization algorithm from
the perspective of analog circuit design.
Recently, many efforts [6]–[9] have explored how to utilize
large language models (LLMs) to enhance the efficiency of
analog circuit design. And they have been fully verified that
LLMs can improve the efficiency and accuracy rate [8],
[9] in the analog circuit sizing process by providing initial
values for conventional optimization algorithms. However, the
existing general LLMs, such as GPT-4o1
, DeepSeek R12
, and
Claude33
, have not undergone specialized training for analog
circuit design [9]. This results in their inability to understand
complex or relatively new-structured analog circuits, and they
1https://chatgpt.com/
2https://www.deepseek.com/
3https://claude.ai

## Page 2

Fig. 1. The overview of the sizing relationship extraction
tend to either be only applicable to classic analog circuits [6],
[8] or require a large number of fixed computational processes
[9] to provide relatively good initial values. Thus, directly
using general LLMs to provide initial values in the analog
circuit sizing process to accelerate the optimization algorithm
efficiency is limited.
In order to continuously and fully leverage the reasoning
capabilities of the general LLMs as prior knowledge and be
applicable to a broader range of circuits, we thus commence
from the perspective of search space pruning to further en-
hance the efficiency of the optimization algorithm.
In this paper, we will introduce a large language model-
based multi-agent framework for analog circuits’ sizing rela-
tionship extraction. This method converts the circuit descrip-
tion from the paper into sizing relationships, thereby pruning
the search space and improving the search efficiency. Unlike
the methods that require a large amount of computation [9]
or have a strong analog-circuit-domain zero-shot answering
ability [8] for providing initial values, sizing relationships
extraction relies more on the text reasoning ability that LLM
is good at. It eliminates the reasoning process of mathematical
formulas in the field of analog circuits and significantly
improves the reliability of the LLMs’ output.
This method is a universal method applicable to multiple
common analog circuit modules, such as Operational Ampli-
fier (OPA), Bandgap Reference (BGR), and Low DropOut
regulator (LDO). We conducted tests on these 3 types of
circuits, and the optimization efficiency was improved by
2.32 ∼ 26.6× and successfully optimized the circuits that
conventional optimization algorithms failed in limited itera-
tions. Overall, this method provides an innovative solution for
improving the automation efficiency of analog circuit sizing.
It also provides a solution for high-quality analog circuit data
generation in the current background of analog circuit data
shortage.
Our paper is organized as follows: Section II will introduce
the background and the reasons why we adopt general LLMs
instead of training a Domain-specific LLM. Section III will
introduce the specific workflow. Section IV will specifically
describe the experimental results. Finally, we will summarize
and discuss our work.
II. PRELIMINARIES
A. LLMs and related research
Large Language Models (LLMs) are a type of artificial in-
telligence technology based on deep learning neural networks
that contain billions (even more) of parameters [10], which are
trained on massive text data [11]. These allow them to grasp
the nuances, context, and complexity of human language.
As a result, LLMs can perform a wide range of language-
related tasks, such as translation, summarization, question
answering, and content generation, with a remarkable degree
of proficiency.
Many studies [12] have carried out the use of LLM as an
agent to make decisions on various tasks, and it has accom-
plished tasks in fields such as autonomous driving [13], video
games [14], and even Nobel-Prize-level chemical experiments
[15] outstandingly.
To further enable LLMs to handle more complex system-
level tasks, many studies [16] have investigated the adoption of
multi-agent solutions. Through cooperation and debate among
multiple agents, the collective wisdom of multiple agents has
been fully brought into play. Through this type of method,
multi-agent systems have accomplished engineering problems
that require the expertise of many more human experts, such
as software development [17], the crystallinity of MOFs and
COFs optimization [18], and medical diagnosis assistance
[19].
Through the breakthroughs of the above LLM agents in
various fields, we are confident in using a set of LLM-based
multi-agent systems to handle engineering problems in analog
circuit design.
B. LLMs for Integrated Circuits
In the field of integrated circuits, LLMs have been widely
applied in digital circuit design, such as RTL verification
[20], bug locations [21], and even CPU generation [22],
significantly improving the development efficiency of digital
circuits. Compared with LLM for digital circuits, LLM for
analog circuits has the problem of a shortage of training
data. Many existing training-based LLMs for analog circuit
auto-design techniques [23], [24] rely on self-organized data.
However, compared with other datasets used for the training

## Page 3

of LLMs [25], where a single pre-trained dataset can reach
hundreds of terabytes in size, and even a single dataset in the
supervised fine-tuning stage can contain millions of data pairs,
the data collected in [23], [24] is of a very small quantity. This
limited data leads to the LLMs they trained being capable
of solving only circuits with fixed structures or potentially
encountering the problem of overfitting.
Therefore, how to combine LLMs with the existing analog
circuit automation design methods and generate high-quality
diverse analog circuit data is what makes more significant
contributions of LLMs for analog circuits at present. In this
work, we focus on accelerating the process of extracting actual
circuit data from papers based on existing general LLMs:
through the extraction of the sizing relationship of the circuits
by LLMs, pruning of the search space is carried out, thereby
improving the efficiency and reliability of the sizing process,
and ultimately obtaining high-quality analog circuit data.
III. OVERVIEW
The overview of the multi-agent framework is shown in
Fig.1. The papers will first undergo preprocessing to extract
the circuit diagram, netlist, and design method related to the
circuits mentioned in the papers. These data will be taken to-
gether as input to the multi-agent team for iterative discussion
and analysis. After reaching a fixed number of iterations, the
conclusion information summarized by the team will be sent
as the extracted sizing relationship to the conventional sizing
algorithm for optimizing the circuit netlist. Next, a detailed
elaboration will be provided for each module.
A. Paper Pre-Processing
Since the input of existing LLMs is generally text or text
along with images, in order to extract the sizing relationship
information from the papers, we need to preprocess the PDF
format files of the papers into text first. Here, we have adopted
the commercial tool Mathpix4
. After this step, we obtained
the circuit diagrams in the papers and the design information
related to the circuits.
After obtaining the circuit diagrams, in order to convert
them into simulatable netlists, we adopted a similar technique
as in [26]. We will identify the devices, connection relation-
ships, and text in the pictures through CV solutions such as
Yolo-V8 [27] and OCR, thereby restoring the corresponding
netlist from the circuit diagrams in the papers. Next, a script
specifically designed for module detection will identify and
detect special modules in the netlist and recognize the common
and necessary ones, such as differential pairs and current
mirrors. Through this method, the pressure on multi-agents
for fine-grained module recognition can be alleviated, and
the accuracy of the final sizing relationship extraction can be
improved.
4https://mathpix.com
B. Multi-Agent Team
As mentioned in Part A of Section II, the multi-agent system
has achieved good results in multiple engineering tasks. Here,
we also adopt a multi-agent system to achieve the accurate
extraction of the sizing relationship of the analog circuits in
the papers.
The construction methods of the multi-agent system mainly
unfold around three aspects [16], including 1) Communication
Paradigms: the styles and methods of interaction between
agents; 2) Communication Structure: the organization and
architecture of communication networks within the multi-agent
system; and 3) Communication Content: what kind of message
is exchanged between agents.
Communication Paradigms we adopt include cooperation
and debate: First of all, these models need to cooperate to
discuss the correct sizing relationship. Compared with the
single-agent system, the multi-agent system is more likely to
achieve comprehensive coverage of the extraction of the sizing
relationship. Secondly, since existing general models have not
received specialized training for analog circuits, a single agent
may have the possibility of extracting the relationship wrongly,
which leads to the occurrence of opinion conflicts among
multiple agents. At this time, a debate is required to allow
agents with different opinions to discuss, making them pay
more attention to extracting strong evidence based on the
papers, thereby making the extracted sizing relationship more
reliable.
Communication Structure we adopt is the Shared Message
Pool, which is proposed in [28]. These agents can see the
sizing relationships and specific ideas extracted by each other,
improving the communication efficiency among agents.
Communication Content includes the structured extraction
results of sizing relationships and the thinking processes
involved in extracting these relationships. By sharing the
thinking processes, agents can better analyze the reasons
for extracting sizing relationships from each other, thereby
further enhancing the reliability of the overall agent system
in extracting sizing relationships.
In addition to the common multi-agent construction men-
tioned above, we assigned different models to agents of dif-
ferent roles. In this work, We divided the roles into one expert
and multiple employees. In terms of personnel allocation, we
found that if the agents were all focused on the discussion, they
only tended to extract the sizing relationship from the paper
in the first round (before the discussion), and in the remaining
iteration rounds, they only focused on the discussion, resulting
in the possible problem of missed extraction of the sizing
relationship. Therefore, we ultimately chose to have the Expert
Agent not participate in the discussion and propose new sizing
relationships based on the academic paper in each iteration
round, thereby making the coverage of the extracted sizing re-
lationships more extensive. Combined with the characteristics
of each role, the expert agent is more inclined to use an LLM
with a long thought chain, while the employee is inclined to
use a relatively low-cost LLM. The specific LLM selection

## Page 4

TABLE I
THE CONFIGURATIONS OF THE MULTI-AGENT TEAM
Expert Agent Employee Agent
Model Num. Temp. Model Num. Temp.
CFG 1 GPT-o1 1 × GPT-4o 3 0.5
CFG 2 GPT-o1 1 × Claude3.5-opus 3 0.5
×GPT-o1 does not support temperature adjustment.
will be elaborated in detail in the experimental section.
Through the above configuration, this multi-agent team can
stably and efficiently extract the sizing relationships of various
circuits in the papers to be reproduced accurately. A detailed
extraction process will be described in the experimental sec-
tion.
C. Sizing Relationship Format
The sizing relationship we extracted mainly includes three
key pieces of information: device name, related parameters,
and quantitative relationship, as shown in Fig.1. The device
name will be used to match the corresponding device in the
netlist for matching. The related parameters will be used to
specify what parameters have the relationship set. The quanti-
tative relationship specifically records the specific quantitative
magnitude between different device parameters. The output
of the multi-agent team will be structured and output in the
above format, thus cascading with the subsequent optimization
algorithm.
IV. EXPERIMENT
In this section, we will present in detail an example of
extracting sizing relationships using this method. Further-
more, we extracted the sizing relationships for three circuits
and compared them with the efficiency and accuracy of the
optimization algorithm when no sizing relationships were
provided.
A. Experiment Setting
In this section, we will provide a detailed explanation of
the experimental setup. Initially, it is important to note that
the proposed method is designed for general purposes, and the
application of LLMs from various sources, provided they are
of comparable levels, will not significantly alter the outcomes
of the multi-agent team. To substantiate this consideration, we
employed two distinct state-of-the-art general LLMs configu-
rations for the multi-agent team, as detailed in Table.I. 5
The
number of discussion rounds of the multi-agent team is set to
5. The multi-agent team is based on LangChain6
.
In terms of the optimization algorithm used to perform
sizing, the running CPU is Intel®
Xeon®
CPU E5-2698 v4 @
2.20GHz, and the development platform is based on Tsinghua
Electronic Design (TED) [29]. The optimization algorithm
5Because among the current long reasoning LLMs, only GPT-o1 has the
ability of image understanding, GPT-o1 was selected as the only expert agent
here.
6https://python.langchain.com/
TABLE II
EXTRACTION STABILITY OF THE MULTI-AGENT TEAM
BGR [30] OP [31] LDO [32]
CFG1 CFG2 CFG1 CFG2 CFG1 CFG2
Attempt1 11 12 5 4 10 11
Attempt2 11 11 3 5 9 10
Attempt3 12 12 3 3 10 10
Max Variation↓ 1 1 2 2 1 1
adopts MACE [3], its maximum number of iteration rounds
is set to 100, the batch number is set to 40, and its initial
point is set to 40. The academic papers we selected for sizing
extraction are [30]–[32].
B. Extraction Case
To show the extraction process of the multi-agent team more
explicitly, we have presented some of the contents output by
the multi-agent team in the first two rounds when extracting
sizing relationship from Paper [30] under CFG1, as shown in
Fig.2.
Some of the thinking processes, as well as the opposing
and agreeing viewpoints, have been omitted due to space
limitations. However, it can be seen that in the first round,
each agent extracted a considerable number of similar sizing
relations, but there were some differences in each detail. It is
worth noting that the Expert was the only one to propose the
quantitative relationship of M3 and M4, which was calculated
based on the parameter table in the paper. In the second
round, each agent agreed and refuted each other’s viewpoints.
Furthermore, the Expert continued to introduce new sizing
relationships and further paid attention to the relationship
between the current mirror and other quantitative relationships.
We also tested the extraction sizing relationship stability for
these academic papers under two CFGs. We conducted three
tests for each paper under each CFG and counted the number
of final valid sizing relationships. Among them, a valid sizing
relationship means that, for example, if some sizing extracted
two items: M1 = M2, M2 = M3, it is actually equivalent
to M1 = M2 = M3, and is actually counted as one valid
piece sizing relationship. The specific experimental results are
shown in Table.II. It can be seen from Table.II that regardless
of which CFG is used, the number of extracted effective sizing
relationships is relatively stable, proving the stability of the
multi-agent team in this task and that the method proposed in
this paper is a general one. As long as the LLM ability levels
are comparable, it will not have an impact on the final result.
C. Efficiency Improvement
In this part, we will compare the efficiency of the op-
timization algorithm with and without sizing relations. The
demand indicators of the three circuits and the optimization
results with and without sizing relationships are presented
in Table.III. It can be seen that after combining the sizing
relationship, the optimization algorithm can achieve that all
the indicators of the experimental circuit can be satisfied.
However, using only the conventional algorithm, within fixed

## Page 5

Fig. 2. Examples when extracting sizing relationship from [30]
TABLE III
CIRCUIT OPTIMIZATION OBJECTIVE CONFIGURATION
AND OPTIMIZATION RESULTS COMPARISON
Attribute Target
w/ sizing
Relationship
w/o sizing
Relationship
BGR [30]
@SMIC180nm
Vref 0.5 ∼ 1.3V 1.24V 1.39V%
TC↓ < 25ppm 23ppm 28ppm%
PSR↓ < −40dB -43dB -45dB
Power↓ < 1µW 242nW 270nW
OP [31]†
@TSMC65nm
DC Gain↑ > 70dB 71.1dB 55.7dB%
UGB↑ > 8kHz 33kHz 332kHz
Phase Margin↑ > 40◦ 75◦ 42.4◦
PSRR↑ > 50dB 63dB 53.0dB
CMRR↑ > 60dB 85.8dB 66.0dB
Offset < 5mV 0.1mV 1mV
Slew Rate↑ > 10V/ms 27.5V/ms 381V/ms
Power < 1µW 265nW 477nW
LDO [32]
@TSMC65nm
Load Regulation↓ < 0.2V/A 0.017V/A 0.046V/A
Line Regulation↓ < 0.2V/V 0.08V/V 1.1V/V
Output Voltage 1.05 ∼ 1.15V 1.09V 1.11V
PSRR↑ > 50dB 54.6dB 59.3dB
Power↓ < 100µW 46.5µW 84.1µW
† Though performing well when no sizing relationship, oscillation occurs.
iterations, it is impossible to optimize the circuit parameters
that meet all the indicator requirements. Additionally, it is
worth mentioning that although the conventional optimization
algorithm yields good final indicators when designing the
OP, oscillation problems occur, which leads to the optimized
circuit having no application value.
Moreover, the above experiments were carried out on two
different process nodes, SMIC 180nm and TSMC 65nm,
verifying the robustness of the method in terms of process
migration.
Furthermore, the efficiency of the optimization algorithm
with and without sizing relations is compared in Table.IV. The
cases of optimization failure are also included in the calcula-
tion of the average time consumption, and the consumption
time of each case is the time consumed until the maximum
number of iterations is reached. However, there is still room
for improvement in the current scheme. The optimization pass
rate still cannot reach 100%. How to further complement
each other between LLM and the optimization algorithm to
improve efficiency will be the key to our next research.
From Table.IV, it can be seen that after adding the sizing
relationship, the time consumption of the algorithm decreased
by 2.32 ∼ 26.6× and successfully optimized the circuits
that conventional optimization algorithms failed in limited
iterations. This experiment further verifies the effectiveness
of pruning the search space into the optimization algorithm
by bringing the sizing relationships. However, as can be seen
from the experiment, although this method has improved
the optimization efficiency to a large extent, it still cannot
guarantee a 100% optimization pass rate, thereby increasing
the average consumption time.
Overall, through the experiments in this section, we can

## Page 6

TABLE IV
OPTIMIZATION EFFICIENCY AND PASS RATE COMPARISON
w/ sizing Relationship w/o sizing Relationship
Time(s)↓ Pass Rate↑ Time(s)↓ Pass Rate↑
BGR [30]
@SMIC180nm
3201.3 8/10 7442 0/10
OP [31]
@TSMC65nm
275.17 10/10 7323 0/10
LDO [32]
@TSMC65nm
1027.8 10/10 5081 5/10
observe that the extraction of sizing relationships from papers
through the multi-agent system proposed in this work not
only has strong stability but also significantly improves the
efficiency and accuracy of the optimization algorithm by
pruning the search space.
V. CONCLUSION
In this work, we propose an LLM-based multi-agent frame-
work for extracting the analog circuits sizing relationships
from academic papers, thereby pruning the search space
and ultimately improving the efficiency of the optimization
algorithm. We conducted experiments on three types of dif-
ferent circuits and obtained that this method can increase the
optimization efficiency of the original optimization algorithm
by 2.32 ∼ 26.6× and successfully optimized the circuits
that conventional optimization algorithms failed in limited
iterations.
Under the current background of the shortage of analog
circuit data, this method provides a new solution for distill-
ing knowledge from analog circuit academic papers, thereby
accelerating the construction process of high-quality data of
analog circuits and providing a new scheme for the efficient
collection of analog circuit data.
REFERENCES
[1] Z. Dong, W. Cao, M. Zhang, D. Tao, Y. Chen, and X. Zhang, “Cktgnn:
Circuit graph neural network for electronic design automation,” in The
Eleventh International Conference on Learning Representations, 2023.
[2] H. Wang, K. Wang, J. Yang, L. Shen, N. Sun, H.-S. Lee, and S. Han,
“Gcn-rl circuit designer: Transferable transistor sizing with graph neural
networks and reinforcement learning,” in 2020 57th ACM/IEEE Design
Automation Conference (DAC). IEEE, 2020, pp. 1–6.
[3] W. Lyu, F. Yang, C. Yan, D. Zhou, and X. Zeng, “Batch
Bayesian optimization via multi-objective acquisition ensemble for
automated analog circuit design,” in Proceedings of the 35th
International Conference on Machine Learning, ser. Proceedings
of Machine Learning Research, J. Dy and A. Krause, Eds.,
vol. 80. PMLR, 10–15 Jul 2018, pp. 3306–3314. [Online]. Available:
https://proceedings.mlr.press/v80/lyu18a.html
[4] S. Zhang, F. Yang, C. Yan, D. Zhou, and X. Zeng, “An efficient
batch-constrained bayesian optimization approach for analog circuit
synthesis via multiobjective acquisition ensemble,” IEEE Transactions
on Computer-Aided Design of Integrated Circuits and Systems, vol. 41,
no. 1, pp. 1–14, 2021.
[5] K. Touloupas and P. P. Sotiriadis, “Locomobo: A local constrained
multiobjective bayesian optimization for analog circuit sizing,” IEEE
Transactions on Computer-Aided Design of Integrated Circuits and
Systems, vol. 41, no. 9, pp. 2780–2793, 2022.
[6] C. Liu, Y. Liu, Y. Du, and L. Du, “Ladac: Large language model-driven
auto-designer for analog circuits,” Authorea Preprints, 2024.
[7] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“Analogcoder: Analog circuit design via training-free code generation,”
arXiv preprint arXiv:2405.14918, 2024.
[8] Y. Yin, Y. Wang, B. Xu, and P. Li, “Ado-llm: Analog design bayesian
optimization with in-context learning of large language models,” arXiv
preprint arXiv:2406.18770, 2024.
[9] C. Liu, W. Chen, A. Peng, Y. Du, L. Du, and J. Yang, “Ampagent:
An llm-based multi-agent system for multi-stage amplifier schematic
design from literature for process and performance porting,” 2024.
[Online]. Available: https://arxiv.org/abs/2409.14739
[10] W. X. Zhao, K. Zhou, J. Li, T. Tang, X. Wang, Y. Hou, Y. Min, B. Zhang,
J. Zhang, Z. Dong et al., “A survey of large language models,” arXiv
preprint arXiv:2303.18223, 2023.
[11] M. Shanahan, “Talking about large language models,” Communications
of the ACM, vol. 67, no. 2, pp. 68–79, 2024.
[12] Z. Xi, W. Chen, X. Guo, W. He, Y. Ding, B. Hong, M. Zhang, J. Wang,
S. Jin, E. Zhou et al., “The rise and potential of large language model
based agents: A survey,” Science China Information Sciences, vol. 68,
no. 2, p. 121101, 2025.
[13] L. Wen, D. Fu, X. Li, X. Cai, T. Ma, P. Cai, M. Dou, B. Shi, L. He, and
Y. Qiao, “Dilu: A knowledge-driven approach to autonomous driving
with large language models,” arXiv preprint arXiv:2309.16292, 2023.
[14] G. Wang, Y. Xie, Y. Jiang, A. Mandlekar, C. Xiao, Y. Zhu, L. Fan, and
A. Anandkumar, “Voyager: An open-ended embodied agent with large
language models,” arXiv preprint arXiv:2305.16291, 2023.
[15] D. A. Boiko, R. MacKnight, B. Kline, and G. Gomes, “Autonomous
chemical research with large language models,” Nature, vol. 624, no.
7992, pp. 570–578, 2023.
[16] T. Guo, X. Chen, Y. Wang, R. Chang, S. Pei, N. V. Chawla, O. Wiest,
and X. Zhang, “Large language model based multi-agents: A survey of
progress and challenges,” arXiv preprint arXiv:2402.01680, 2024.
[17] C. Qian, X. Cong, C. Yang, W. Chen, Y. Su, J. Xu, Z. Liu, and
M. Sun, “Communicative agents for software development,” arXiv
preprint arXiv:2307.07924, 2023.
[18] Z. Zheng, O. Zhang, H. L. Nguyen, N. Rampal, A. H. Alawadhi,
Z. Rong, T. Head-Gordon, C. Borgs, J. T. Chayes, and O. M. Yaghi,
“Chatgpt research group for optimizing the crystallinity of mofs and
cofs,” ACS Central Science, vol. 9, no. 11, pp. 2161–2170, 2023.
[19] X. Tang, A. Zou, Z. Zhang, Z. Li, Y. Zhao, X. Zhang, A. Cohan, and
M. Gerstein, “Medagents: Large language models as collaborators for
zero-shot medical reasoning,” arXiv preprint arXiv:2311.10537, 2023.
[20] Y. Hu, J. Ye, K. Xu, J. Sun, S. Zhang, X. Jiao, D. Pan, J. Zhou,
N. Wang, W. Shan et al., “Uvllm: An automated universal rtl verification
framework using llms,” arXiv preprint arXiv:2411.16238, 2024.
[21] B. Yao, N. Wang, J. Zhou, X. Wang, H. Gao, Z. Jiang, and N. Guan,
“Location is key: Leveraging large language model for functional bug
localization in verilog,” arXiv preprint arXiv:2409.15186, 2024.
[22] X. Wang, G.-W. Wan, S.-Z. Wong, L. Zhang, T. Liu, Q. Tian, and
J. Ye, “Chatcpu: An agile cpu design and verification platform with llm,”
in Proceedings of the 61st ACM/IEEE Design Automation Conference,
2024, pp. 1–6.
[23] C.-C. Chang, Y. Shen, S. Fan, J. Li, S. Zhang, N. Cao, Y. Chen, and
X. Zhang, “Lamagic: Language-model-based topology generation for
analog integrated circuits,” arXiv preprint arXiv:2407.18269, 2024.
[24] Z. Chen, J. Huang, Y. Liu, F. Yang, L. Shang, D. Zhou, and X. Zeng,
“Artisan: Automated operational amplifier design via domain-specific
large language model,” in Proceedings of the 61st ACM/IEEE Design
Automation Conference, 2024, pp. 1–6.
[25] Y. Liu, J. Cao, C. Liu, K. Ding, and L. Jin, “Datasets for large language
models: A comprehensive survey,” arXiv preprint arXiv:2402.18041,
2024.
[26] Z. Tao, Y. Shi, Y. Huo, R. Ye, Z. Li, L. Huang, C. Wu, N. Bai, Z. Yu,
T.-J. Lin et al., “Amsnet: Netlist dataset for ams circuits,” arXiv preprint
arXiv:2405.09045, 2024.
[27] R. Varghese and S. M., “Yolov8: A novel object detection algorithm
with enhanced performance and robustness,” in 2024 International
Conference on Advances in Data Engineering and Intelligent Computing
Systems (ADICS), 2024, pp. 1–6.
[28] S. Hong, X. Zheng, J. Chen, Y. Cheng, J. Wang, C. Zhang, Z. Wang,
S. K. S. Yau, Z. Lin, L. Zhou et al., “Metagpt: Meta programming for
multi-agent collaborative framework,” arXiv preprint arXiv:2308.00352,
2023.
[29] Z. Ye, Z. Wang, J. Xin, Y. Wang, Q. Qin, C. Chai, Y. Lu, J. Hao, J. Xiao,
and Y. Wang, “Ted: A python-based analog design environment for agile

## Page 7

circuit development,” in 2023 International Symposium of Electronics
Design Automation (ISEDA), 2023, pp. 5–10.
[30] K. Ueno, T. Hirose, T. Asai, and Y. Amemiya, “A 300 nw, 15 ppm/c,
20 ppm/v cmos voltage reference circuit consisting of subthreshold
mosfets,” IEEE Journal of solid-state circuits, vol. 44, no. 7, pp. 2047–
2054, 2009.
[31] L. H. Ferreira, T. C. Pimenta, and R. L. Moreno, “An ultra-low-voltage
ultra-low-power cmos miller ota with rail-to-rail input/output swing,”
IEEE Transactions on Circuits and Systems II: Express Briefs, vol. 54,
no. 10, pp. 843–847, 2007.
[32] K. N. Leung and P. K. Mok, “A capacitor-free cmos low-dropout
regulator with damping-factor-control frequency compensation,” IEEE
Journal of Solid-State Circuits, vol. 38, no. 10, pp. 1691–1702, 2003.

