---
title: "LLM-Enhanced Bayesian Optimization for Efficient Analog Layout Constraint Generation"
authors: "Chen, Guojin; Zhu, Keren; Kim, Seunggeun; Zhu, Hanqing; Lai, Yao; Yu, Bei; Pan, David Z."
date: "2024/06/07"
category: "layout"
source_url: "https://arxiv.org/abs/2406.05250"
pdf: "papers/2024_llm-enhanced-bayesian-optimization-for-efficient-analog-layout-constraint-g_2406-05250.pdf"
sha256: "39fe6fc6141f82b39fce0359e740b6e6764b28bf3716430c43e00f1c174c4d1c"
pages_extracted: 5
---

# LLM-Enhanced Bayesian Optimization for Efficient Analog Layout Constraint Generation

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

LLM-Enhanced Bayesian Optimization for Efficient
Analog Layout Constraint Generation
Guojin Chen1,2, Keren Zhu1, Seunggeun Kim2, Hanqing Zhu2, Yao Lai3, Bei Yu1, David Z. Pan2
1Chinese University of Hong Kong 2University of Texas at Austin 3The University of Hong Kong
Abstract—Analog layout synthesis faces significant challenges
due to its dependence on manual processes, considerable time
requirements, and performance instability. Current Bayesian Op-
timization (BO)-based techniques for analog layout synthesis, de-
spite their potential for automation, suffer from slow convergence
and extensive data needs, limiting their practical application.
This paper presents the LLANA framework, a novel approach
that leverages Large Language Models (LLMs) to enhance BO
by exploiting the few-shot learning abilities of LLMs for more
efficient generation of analog design-dependent parameter con-
straints. Experimental results demonstrate that LLANA not only
achieves performance comparable to state-of-the-art (SOTA) BO
methods but also enables a more effective exploration of the
analog circuit design space, thanks to LLM’s superior contextual
understanding and learning efficiency. The code is available at
https://github.com/dekura/LLANA.
I. INTRODUCTION
The increasing demand for advanced analog and mixed-
signal (AMS) integrated circuits (ICs) in sectors such as
automotive and the Internet of Things (IoT) necessitates faster
design processes and quicker time-to-market. However, the
creation of analog layouts remains a predominantly manual,
time-consuming, and error-prone task. Engineers rely on es-
tablished practices and insights from experienced designers,
incurring substantial costs and prolonging the design cycle.
The growing complexities of layout-dependent effects in newer
technology nodes further complicate the accurate prediction
and assessment of post-layout performance. Although attempts
have been made to automate the analog layout process, their
integration into real-world design practices remains limited.
Existing tools for analog layout synthesis often come with
significant design complexities or fail to ensure desired post-
layout outcomes. Optimization-driven tools [1] require circuit
designers to manually input specific layout constraints, which
are then followed during component placement and routing
(P&R). Methods relying on heuristic constraints face chal-
lenges in real-world applications due to their design-specific
nature, lacking adaptability and universality across different
projects. Performance-driven approaches [2] attempt to account
for layout effects by deriving equations, either analytically or
through sensitivity analyses. However, device miniaturization
makes analytical estimations of post-layout effects increasingly
inaccurate. Accurately predicting the impact of design and
layout-dependent phenomena, such as mismatches and par-
asitics, requires extensive empirical simulations due to their
increased complexity in scaled-down technologies.
Constraint generation plays a crucial role in automatic analog
synthesizers [1], [3], tasked with extracting physical constraints
Netlist
PDK
Configs
Inputs
Layout
Synthesizer
Symmetry Constraint
Detection
Placement & Routing
Parasitic Extraction
Simulation
Output GPT-3.5
Performance
LLANA
Enhanced-BO
Next net weighting
Max. Acq. Function
ICL Surrogate Model
ICL sampler
Net weighting
GDSII
Layout
Fig. 1 Overview of the LLANA framework.
to inform and optimize the subsequent placement and rout-
ing flow. These constraints aim to minimize discrepancies
between pre-layout and post-layout simulations, considering
factors such as device matching, electrical current paths, and
thermal management [4]. While recent advancements [5] have
focused on identifying and enforcing matching constraints,
such as symmetry, an exploration into design-dependent P&R
hyperparameters, such as net weighting, remains absent.
Recent progress in analog circuit dimensioning has signifi-
cantly improved the effectiveness of leveraging simulations for
performance enhancement [3], [6]–[9]. These simulation-driven
methods approach analog dimensioning as an optimization
problem that does not reveal its internal workings, relying
on circuit simulators to evaluate performance metrics. This
approach requires minimal preliminary knowledge about circuit
configurations, contrasting with model-dependent techniques
that necessitate a deep understanding of complex design and
performance interactions. The integration of Gaussian Pro-
cess (GP)-based Bayesian Optimization (BO) for automatic
transistor sizing combines the strengths of both paradigms,
achieving significant reductions in simulation requirements
while maintaining adaptability across various circuit designs.
This advancement highlights the importance of incorporating
iterative simulations into the design feedback loop, inspiring
the development of an integrated, feedback-oriented analog
layout synthesizer.
Despite advances in BO-based automated analog synthe-
sis [6], BO faces challenges in high-dimensional tasks due to
its computational demands and the complexity of probabilistic
model evaluation. Its effectiveness is further limited in envi-
ronments with numerous local optima, owing to its dependence
on prior knowledge and the need to balance exploration and
exploitation. The core of these challenges lies in accurately
learning objective functions and generating solutions with min-
imal data, a scenario often associated with the few-shot setting.
Large Language Models (LLMs), with their extensive training
arXiv:2406.05250v3
[cs.AI]
6
Dec
2024

## Page 2

on internet data, excel in few-shot learning, demonstrating re-
markable abilities in prediction, generation [10], and contextual
understanding [11]. This success is partially attributed to their
utilization of encoded priors.
As depicted in Fig. 1, this paper investigates the feasibility
of leveraging LLMs to improve model-based BO for analog
design-dependent parameter generation and fine-tuning, ex-
tending LLM applications beyond traditional natural language
tasks. We propose a novel approach that utilizes natural lan-
guage representations for BO components, aiming to harness
the unique advantages of LLMs. The central inquiry of our
research is whether LLMs’ inherent knowledge and few-shot
learning capabilities can enhance crucial aspects of BO, par-
ticularly in generating analog layout constraints, and assess
the efficiency of an LLM-augmented BO pipeline functioning
seamlessly from start to finish. Our main contributions are as
follows:
1) We formulated the analog layout design-dependent param-
eter space and provided a benchmark for BO based on
Gaussian processes.
2) For the first time, we applied LLM-enhanced BO to
analog design-dependent constraint generation, exploring
the capabilities of LLM-based design space exploration.
3) All the algorithms, experiments and benchmarks are open-
sourced at https://github.com/dekura/LLANA.
II. PRELIMINARIES AND BACKGROUND
A. Analog placement
Analog circuit layout synthesis involves placement and rout-
ing for both building and macro blocks. Placement optimizes
component locations within a predefined bounding box to
minimize wire lengths, while routing finalizes the layout using
established component positions, pin locations, and connec-
tion data. Symmetry constraints are respected to minimize
mismatches. The placement and routing engine is design-
independent, ensuring efficiency and coherence. The objective
function f for analytical global placement is defined as [12]:
f =
X
i
αi · fW Li + β · fAREA + fother, (1)
where fW Li
is the wirelength of net i, fAREA is the total layout
area, αi and β are the net weightings and area minimization
factor, and fother represents other necessary objectives.
B. Analog design-dependent parameter constraint:
Analog design-dependent parameter constraints, along with
symmetric constraints, define the layout and efficiency of
integrated circuits. Net weightings (αi in eq. (1)) direct global
placement strategies, affecting the floorplan and performance.
Higher weights reduce parasitics by aligning connected compo-
nents more closely. Wire widths and routing sequences manage
the trade-off between parasitic resistance and capacitance,
determining layout complexity. Hyperparameters, such as the
area minimization factor (β in eq. (1)) and net spacing, impact
placement and routing optimization, reducing coupling inter-
ference. Design-dependent parameters and hyperparameters are
crucial for optimizing analog circuit performance and layout.
This paper focuses on the impact of net weightings xi on
layout performance.
III. ALGORITHM AND FRAMEWORK
A. BO-based methods as baselines
Prior works of analog constraint generation have primarily
focused on multi-objective Bayesian optimization(MOBO) and
using Gaussian processes(GP) as surrogate model [6], formu-
lated as:
min(f1(x), f2(x), . . . , fm(x)), (2)
where fi(x) represents the post layout performance metric
obtained through simulation after automatic layout generation
using Magical [12], and x ∈ Rd
are the optimal design specific
layout parameters introduced in Section II-B that minimize the
performance metrics.
Algorithm 1 MOBO [6]
Input: Sampled data points xt, {fi(xt)}m
i=1
Output: Next net weighting xt+1
1: Initialize Pareto set P to ∅;
2: function MOBO(xt, {fi(xt)}m
i=1)
3: if t < Nrandom then
4: Random sample xt+1 from the design space;
5: else
6: Update P with xt and reference point r;
7: Update GP models with fi(x);
8: Update GP model with xt, {fi(xt)}m
i=1;
9: Optimize acquisition function to obtain xt+1;
10: return xt+1;
As presented in Algorithm 1, MOBO iteratively samples
data points xt
and their corresponding performance metrics
{fi(xt
)}m
i=1 to update the GP models and optimize the acqui-
sition function to obtain the next net weighting xt+1.
B. Large Language Models (LLMs)-enhanced BO
BO’s effectiveness depends on the quality of surrogate mod-
els and sampling strategies, challenged by limited data, sensi-
tivity to inaccuracies, and the difficulty of incorporating prior
knowledge into new tasks. Recently, there has been a growing
trend of utilizing AI and LLMs to address challenges within
the EDA domain [13]–[42]. [43] explained LLM in-context
learning (ICL) as performing implicit Bayesian inference [10].
LLMs can enhance BO by leveraging: (1) prior knowledge
through ICL for tapping into pre-trained insights [44], (2) the
ability to generalize from limited examples, aiding in efficient
exploration [45], and (3) processing contextual information to
enrich optimization tasks and search strategies [46].
C. LLM-enhanced BO Framework
LLM-enhanced Initial Design: As illustrated in Fig. 2,
by leveraging LLM’s ‘zero-shot’ capabilities, we can generate
better initial designs through prompt engineering.
Assist me with automated machine learning using {model}. Explore these
hyperparameters: {configurations, type, ranges}. Suggest {number of rec-
ommendations} diverse, effective configs for BO hyperparameter tuning,
without “None”. Respond with an un-enumerated list of dictionaries, each
describing a recommended config.
Fig. 2 Prompt for initial design generation

## Page 3

LLM-enhanced surrogate modeling: BO constructs a sur-
rogate model p(f|x) using m observed input-output pairs
Xm := {(xi, fi)}m
i=1. Common models include the GP [6]
and random forests (SMAC) [47]. The LLANA framework
serializes the optimization trajectory into natural text, e.g., for
an RF model, ‘[max depth is 15, min samples split is 0.5,
..., performance is 0.9]’. These text representations, denoted
as Dm, along with the problem description and queried few-
shot examples xm1
k , are input to the LLM, denoted as Dm1
m .
The LLM outputs a predicted score and associated probability:
( ˆ
fk, p( ˆ
fk)) = LLANA (xm1
k , Dm1
m ). A shuffling mechanism
randomly permutes the few-shot examples within Dm1
m to
enhance robustness. The in-context learning prompts for the
LLM-enhanced surrogate model are shown in Fig. 3.
The following are examples of the performance of a {model} measured
in {metric} and the corresponding model hyperparameter configurations.
The model is evaluated on a tabular {task} containing {number of
classes}. The tabular dataset contains {number of samples} samples
and {number of features} features ({number of categorical features}
categorical, {number of continuous features} numerical).
Your response should only contain the predicted accuracy in the format
## performance##.
Hyperparameter configuration: {C1}. Performance: {P1}.
...
Hyperparameter configuration: {Cn}. Performance: {Pn}.
Hyperparameter configuration: {configuration to predict}. Perfor-
mance:{}.
Fig. 3 Prompt for discriminative surrogate model.
Acquisition strategies: Sampling candidate points is crucial
in BO, as high-potential points can accelerate convergence to
the optimum. LLANA introduces a novel mechanism to condi-
tionally generate candidate points based on desired objective
values through ICL. The samples are generated from high-
potential regions by conditioning on a desired objective value
x′
: h̃m ∼ p(f|x′
; Dm), leveraging the few-shot generation
capabilities of LLMs. The desired objective value is defined
as x′
= xmin − α × (xmax − xmin), where xmax and xmin
are the worst and best observed points at related objective
values, and α is an exploration hyperparameter. Positive α
sets x′
to improve over xmin, while negative α results in a
more conservative target value within the observed range. We
implement p(f|x′
; Dm) through ICL, generating M candidate
points independently, i.e., h̃k ∼ LLANA(x′
, Dm1
m ), and select
the point that maximizes the acquisition function as the next
point to evaluate, using a sampling-based approximation to
optimize the acquisition function.
End-to-end LLANA framework: The end-to-end procedure
iteratively performs three steps, as depicted in Algorithm 2.
(1) sample M candidate points {h̃m}M
m=1 through ICL. (2)
evaluate M points using the ICL surrogate model, i.e. p(f |
h̃m) to obtain scores {a(h̃m)}M
m=1 according to an acquisition
function. (3) select point with the highest score to evaluate next,
h = argmaxh̃∈{h̃m}M
m=1
a(h̃). In LLANA , we use expected
improvement (EI), a(h̃m) = E[max(p(f | h̃m) − f(hbest ), 0)].
The following are examples of the performance of a {model} in {metric}
and corresponding hyperparameter configs, evaluated on a tabular {task}
task with {number of classes} classes, {number of samples} samples,
{number of features} features ({number of categorical features} categor-
ical, {number of continuous features} numerical). Hyperparameter ranges:
{configuration and type}.
Recommend a config to achieve {target score}, avoiding min/max/rounded
values, using highest precision. Respond with only the predicted config,
as ## configuration ##.
Performance: {P1}. Hyperparameter config: {C1}
...
Performance: {Pn}. Hyperparameter config: {Cn}
Performance: {performance used to sample configuration}
Hyperparameter config:{}.
Fig. 4 Prompt for candidate sampling.
Algorithm 2 End-to-end LLANA-BO with ICL
Input: Initial best point hbest
Output: Optimal point h⋆
1: function LLANA(hm)
2: Sample M candidate points {h̃m}M
m=1 through ICL;
3: Evaluate p(f | h̃m) using the ICL surrogate model;
4: EI score a(h̃m) = E[max(p(f | h̃m) − f(hbest), 0)];
5: h = argmaxh̃∈{h̃m}M
m=1
a(h̃);
6: if f(h) > f(hbest) then
7: hbest = h;
8: return hbest as h;
IV. EXPERIMENTS
The LLANA framework is implemented in Python, utilizing
the ‘gpt-3.5-turbo’ model from OpenAI with hyperparameters
set to α = −0.1 and M = 20. The designs used in the
experiments are two-stage operational amplifiers from [6], and
the performance benchmark is evaluated by Cadence Spectre
after layout generation using Magical [12]. The design contains
a total of 36 devices with 14 design-dependent parameters of
the critical net weighting selected by the MOBO algorithm in-
troduced in Algorithm 1. A dataset of 500 design-performance
pairs is prepared, with 400 pairs used for training and 100
pairs for testing. The optimization objectives chosen are the
common-mode rejection ratio (CMRR) and absolute input-
referred offset (Offset) voltage. The experiments include three
ML models: RandomForest (RF), AdaBoost, and DecisionTree.
The scoring function is the mean squared error (MSE), and the
acquisition function is expected improvement (EI).
A. Evaluation of the surrogate model
The performance of the LLANA framework is first evaluated
against single-objective GP and SMAC [47] on both CMRR
and Offset datasets. For GP and SMAC, the average result of
three ML models is used. The performance of the surrogate
model is assessed using prediction performance and uncertainty
calibration metrics. The Normalized Root Mean Square Error
(NRMSE) and the coefficient of determination (R2
score) of
the prediction on the test set are used as performance metrics.
Calibration is evaluated using the log predictive density (LPD)
and the normalized regret. The regret metric is defined as
minh∈Ht
(f(h) − f∗
min)/(f∗
max − f∗
min ), where Ht denotes the
points chosen up to trail t, and f∗
max,f∗
min represent the best
and worst scores [47].

## Page 4

5 10 15 20 25 30
0.2
0.4
0.6
0.8
1.0
NRMSE ( )
5 10 15 20 25 30
10
5
0
R2 Score ( )
5 10 15 20 25 30
0.05
0.10
0.15
0.20
0.25
Regret ( )
5 10 15 20 25 30
4
2
0
2
LPD ( )
CMRR dataset
GP SMAC LLANA
Fig. 5 Comparison of LLANA , GP, and SMAC [47] on CMRR
dataset. The x-axis is the number of observed points.
Fig. 5 plots the performance of LLANA against GP and
SMAC on the CMRR dataset. LLANA outperforms GP and
SMAC in terms of NRMSE and R2
score. Moreover, this trend
becomes more apparent when there are more observed points.
As for normalized regret, LLANA attains lower regret than GP
and SMAC when n < 15. For uncertainty calibration, LLANA
achieves lower LPD than GP and SMAC, producing the best
uncertainty quantification.
Fig. 6 shows the performance of LLANA against GP and
SMAC on the Offset dataset. This time, as the number of
observed points increases, LLANA can achieve NRMSE and R2
scores close to those of GP and SMAC when n = 30. However,
the regret and LPD of LLANA consistently remain far ahead of
GP and SMAC. Additionally, we observe that, consistent with
prior findings, LLANA excels in earlier stages of the search,
when fewer observations are available. As such, empirical
evidence supports that permuting few-shot examples, while
straightforward in implementation, improves both uncertainty
quantification and prediction performance, both critical aspects
of balancing exploration and exploitation. LLANA performs ef-
fectively as an end-to-end pipeline, exhibiting sample-efficient
search. Its modularity further enables individual components
to be integrated into existing frameworks. Surrogate models
implemented through ICL can produce effective regression
estimates with uncertainty, although there is a tradeoff of
stronger prediction performance with worse calibration than
probabilistic methods. The LLM’s encoded prior is crucial to
improving the efficacy of such surrogate models.
V. CONCLUSION AND DISCUSSION
This work introduces LLANA , a novel framework that
integrates LLMs with BO to address the challenge of gen-
erating analog net weighting constraints. The approach in-
corporates two key enhancements: surrogate models of the
objective function through ICL and a candidate point sampler
capable of conditional generation for specific target values.
The study on the analog constraint problem reveals perfor-
mance improvements across the three integrations, particularly
5 10 15 20 25 30
1.0
1.5
2.0
2.5
3.0
NRMSE ( )
5 10 15 20 25 30
150
100
50
R2 Score ( )
5 10 15 20 25 30
0.1
0.2
0.3
0.4
0.5
Regret ( )
5 10 15 20 25 30
2
0
2
4
LPD ( )
Offset dataset
GP SMAC LLANA
Fig. 6 Comparison of LLANA , GP, and SMAC [47] on Offset
dataset. The x-axis is the number of observed points.
when working with limited sample sizes. Moreover, LLANA
demonstrates potential as a stand-alone BO method, exhibit-
ing slightly better results on CMRR and Offset benchmarks
compared to existing techniques. However, further research is
necessary to fully assess the extent of LLANA ’s capabilities
and its potential for generalization across a broader range of
optimization tasks. Codes and experiment results are available
at https://github.com/dekura/LLANA.
Limitations and future work. Despite LLANA ’s higher
computational cost due to LLM inference, the findings sug-
gest that it trades off computational complexity for improved
sample efficiency, which is essential in black-box optimization.
This indicates the potential for combining LLANA with more
computationally efficient methods to achieve better solutions.
Furthermore, unlike the multi-objective approach in [6], this
work focuses solely on single-objective BO. A promising
avenue for future research is extending LLANA to handle
multi-objective and higher-dimensional BO tasks with more
complex search spaces, enhancing its applicability and impact
in optimization.
REFERENCES
[1] K. Kunal, M. Madhusudan, A. K. Sharma, W. Xu, S. M. Burns,
R. Harjani, J. Hu, D. A. Kirkpatrick, and S. S. Sapatnekar, “INVITED:
ALIGN – Open-Source Analog Layout Automation from the Ground
Up,” in ACM/IEEE Design Automation Conference (DAC), 2019.
[2] H.-C. Ou, K.-H. Tseng, J.-Y. Liu, I.-P. Wu, and Y.-W. Chang, “Layout-
dependent-effects-aware analytical analog placement,” in ACM/IEEE
Design Automation Conference (DAC), 2015.
[3] H. Chen, M. Liu, X. Tang, K. Zhu, A. Mukherjee, N. Sun, and D. Z. Pan,
“MAGICAL 1.0: An open-source fully-automated ams layout synthesis
framework verified with a 40-nm 1GS/s ∆
P
ADC,” in IEEE Custom
Integrated Circuits Conference (CICC), 2021.
[4] M. P.-H. Lin, Y.-W. Chang, and C.-M. Hung, “Recent research develop-
ment and new challenges in analog layout synthesis,” in IEEE/ACM Asia
and South Pacific Design Automation Conference (ASPDAC), 2016.
[5] K. Zhu, H. Chen, M. Liu, and D. Z. Pan, “Automating analog constraint
extraction: From heuristics to learning: (invited paper),” in IEEE/ACM
Asia and South Pacific Design Automation Conference (ASPDAC), 2022.
[6] M. Liu, K. Zhu, J. Gu, L. Shen, X. Tang, N. Sun, and D. Z. Pan, “Closing
the design loop: Bayesian optimization assisted hierarchical analog layout
synthesis,” in ACM/IEEE Design Automation Conference (DAC), 2020.

## Page 5

[7] K. Zhu, H. Chen, M. Liu, X. Tang, N. Sun, and D. Z. Pan, “Effective
analog/mixed-signal circuit placement considering system signal flow,”
in IEEE/ACM International Conference on Computer-Aided Design (IC-
CAD), 2020.
[8] P. Xu, G. Chen, K. Zhu, T. Chen, T.-Y. Ho, and B. Yu, “Performance-
driven analog routing via heterogeneous 3dgnn and potential relaxation,”
in ACM/IEEE Design Automation Conference (DAC), 2024.
[9] H. Zhu, K. Zhu, J. Gu, H. Jin, R. T. Chen, J. A. Incorvia, and D. Z. Pan,
“Fuse and mix: Macam-enabled analog activation for energy-efficient
neural acceleration,” in Proceedings of the 41st IEEE/ACM International
Conference on Computer-Aided Design, ser. ICCAD ’22. New York,
NY, USA: Association for Computing Machinery, 2022.
[10] T. Brown, B. Mann, N. Ryder, M. Subbiah, J. D. Kaplan, P. Dhariwal,
A. Neelakantan, P. Shyam, G. Sastry, A. Askell et al., “Language
models are few-shot learners,” Annual Conference on Neural Information
Processing Systems (NeurIPS), 2020.
[11] J. Wei, Y. Tay, R. Bommasani, C. Raffel, B. Zoph, S. Borgeaud,
D. Yogatama, M. Bosma, D. Zhou, D. Metzler et al., “Emergent abilities
of large language models,” arXiv preprint arXiv:2206.07682, 2022.
[12] B. Xu, K. Zhu, M. Liu, Y. Lin, S. Li, X. Tang, N. Sun, and D. Z.
Pan, “MAGICAL: Toward Fully Automated Analog IC Layout Lever-
aging Human and Machine Intelligence: Invited Paper,” in IEEE/ACM
International Conference on Computer-Aided Design (ICCAD), 2019.
[13] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“AnalogCoder: Analog circuit design via training-free code generation,”
arXiv preprint arXiv:2405.14918, 2024.
[14] Y. Lai, J. Liu, D. Z. Pan, and P. Luo, “Scalable and effective arith-
metic tree generation for adder and multiplier designs,” arXiv preprint
arXiv:2405.06758, 2024.
[15] H. Ren and J. Hu, Machine Learning Applications in Electronic Design
Automation. Springer, 2022.
[16] S. Pujar, L. Buratti, X. Guo, N. Dupuis, B. Lewis, S. Suneja, A. Sood,
G. Nalawade, M. Jones, A. Morari, and R. Puri, “Invited: Automated
code generation for information technology tasks in yaml through large
language models,” in ACM/IEEE Design Automation Conference (DAC),
2023.
[17] Z. He, H. Wu, X. Zhang, X. Yao, S. Zheng, H. Zheng, and B. Yu,
“ChatEDA: A Large Language Model Powered Autonomous Agent for
EDA,” in ACM/IEEE Workshop on Machine Learning CAD (MLCAD),
2023.
[18] Z. Zhao, F. Fan, W. Liao, and J. Yan, “Grounding and enhancing
grid-based models for neural fields,” in Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition (CVPR), June
2024.
[19] Z. Zhao and J. Jia, “End-to-end view synthesis via nerf attention,” 2022.
[20] G. Chen, W. Chen, Y. Ma, H. Yang, and B. Yu, “DAMO: Deep agile mask
optimization for full chip scale,” in IEEE/ACM International Conference
on Computer-Aided Design (ICCAD), 2020.
[21] J. Blocklove, S. Garg, R. Karri, and H. Pearce, “Chip-Chat: Challenges
and Opportunities in Conversational Hardware Design,” in ACM/IEEE
Workshop on Machine Learning CAD (MLCAD), 2023.
[22] L. Wen, Y. Zhu, L. Ye, G. Chen, B. Yu, J. Liu, and C. Xu, “LayouTrans-
former: Generating layout patterns with transformer via sequential pattern
modeling,” in IEEE/ACM International Conference on Computer-Aided
Design (ICCAD), 2022.
[23] W. Zhao, X. Yao, Z. Yu, G. Chen, Y. Ma, B. Yu, and M. D. F. Wong,
“AdaOPC: A self-adaptive mask optimization framework for real design
patterns,” in IEEE/ACM International Conference on Computer-Aided
Design (ICCAD), 2022.
[24] S. Thakur, B. Ahmad, H. Pearce, B. Tan, B. Dolan-Gavitt, R. Karri,
and S. Garg, “VeriGen: A Large Language Model for Verilog Code
Generation,” ACM Transactions on Design Automation of Electronic
Systems (TODAES), 2023.
[25] Z. Wang, Y. Shen, W. Zhao, Y. Bai, G. Chen, F. Farnia, and B. Yu, “Diff-
Pattern: Layout pattern generation via discrete diffusion,” in ACM/IEEE
Design Automation Conference (DAC), 2023.
[26] S. Liu, W. Fang, Y. Lu, Q. Zhang, H. Zhang, and Z. Xie, “Rtlcoder:
Outperforming gpt-3.5 in design rtl generation with our open-source
dataset and lightweight solution,” arXiv preprint arXiv:2312.08617, 2023.
[27] G. Chen, Z. Yu, H. Liu, Y. Ma, and B. Yu, “DevelSet: Deep neural
level set for instant mask optimization,” in IEEE/ACM International
Conference on Computer-Aided Design (ICCAD), 2021.
[28] Z. He and B. Yu, “Large Language Models for EDA: Future or Mirage?”
in ACM International Symposium on Physical Design (ISPD), 2024.
[29] G. Chen, Z. Pei, H. Yang, Y. Ma, B. Yu, and M. Wong, “Physics-
informed optical kernel regression using complex-valued neural fields,”
in ACM/IEEE Design Automation Conference (DAC), 2023.
[30] M. Liu, T.-D. Ene, R. Kirby, C. Cheng, N. Pinckney, R. Liang, J. Alben,
H. Anand, S. Banerjee, I. Bayraktaroglu et al., “ChipNeMo: Domain-
Adapted LLMs for Chip Design,” arXiv preprint arXiv:2311.00176,
2023.
[31] Z. Pei, F. Liu, Z. He, G. Chen, H. Zheng, K. Zhu, and B. Yu, “AlphaSyn:
Logic synthesis optimization with efficient monte carlo tree search,” in
IEEE/ACM International Conference on Computer-Aided Design (IC-
CAD), 2023.
[32] M. Liu, N. Pinckney, B. Khailany, and H. Ren, “Verilogeval: Evaluating
large language models for verilog code generation,” in IEEE/ACM
International Conference on Computer-Aided Design (ICCAD), 2023.
[33] G. Chen, Z. Wang, B. Yu, D. Z. Pan, and M. D. Wong, “Ultra-Fast
Source Mask Optimization via Conditional Discrete Diffusion,” IEEE
Transactions on Computer-Aided Design of Integrated Circuits and
Systems (TCAD), 2024.
[34] Y. Lu, S. Liu, Q. Zhang, and Z. Xie, “Rtllm: An open-source benchmark
for design rtl generation with large language model,” arXiv preprint
arXiv:2308.05345, 2023.
[35] G. Chen, H. He, P. Xu, H. Geng, and B. Yu, “Efficient bilevel source mask
optimization,” in ACM/IEEE Design Automation Conference (DAC), 7
2024.
[36] L. Chen, Y. Chen, Z. Chu, W. Fang, T.-Y. Ho, Y. Huang, S. Khan, M. Li,
X. Li, Y. Liang et al., “The Dawn of AI-Native EDA: Promises and
Challenges of Large Circuit Models,” arXiv preprint arXiv:2403.07257,
2024.
[37] B. Zhu, S. Zheng, Z. Yu, G. Chen, Y. Ma, F. Yang, B. Yu, and M. Wong,
“L2O-ILT: Learning to Optimize Inverse Lithography Techniques,” IEEE
Transactions on Computer-Aided Design of Integrated Circuits and
Systems (TCAD), 2023.
[38] R. Liang, A. Agnesina, G. Pradipta, V. A. Chhabria, and H. Ren, “Invited
Paper: CircuitOps: An ML Infrastructure Enabling Generative AI for
VLSI Circuit Optimization,” in IEEE/ACM International Conference on
Computer-Aided Design (ICCAD), 2023.
[39] W. Li, G. Chen, H. Yang, R. Chen, and B. Yu, “Learning Point Clouds
in EDA,” in ACM International Symposium on Physical Design (ISPD),
2021.
[40] M. DeLorenzo, A. B. Chowdhury, V. Gohil, S. Thakur, R. Karri, S. Garg,
and J. Rajendran, “Make Every Move Count: LLM-based High-Quality
RTL Code Generation Using MCTS,” arXiv preprint arXiv:2402.03289,
2024.
[41] Y. Lai, J. Liu, Z. Tang, B. Wang, J. Hao, and P. Luo, “ChiPFormer: trans-
ferable chip placement via offline decision transformer,” in Proceedings
of the 40th International Conference on Machine Learning, ser. ICML’23.
JMLR.org, 2023.
[42] Y. Lai, Y. Mu, and P. Luo, “MaskPlace: Fast chip placement via rein-
forced visual representation learning,” in Advances in Neural Information
Processing Systems, vol. 35, 2022.
[43] S. M. Xie, A. Raghunathan, P. Liang, and T. Ma, “An explanation
of in-context learning as implicit bayesian inference,” arXiv preprint
arXiv:2111.02080, 2021.
[44] S. Mirchandani, F. Xia, P. Florence, B. Ichter, D. Driess, M. G. Arenas,
K. Rao, D. Sadigh, and A. Zeng, “Large language models as general
pattern machines,” arXiv preprint arXiv:2307.04721, 2023.
[45] T. Liu, N. Astorga, N. Seedat, and M. van der Schaar, “Large
Language Models to Enhance Bayesian Optimization,” arXiv preprint
arXiv:2402.03921, 2024.
[46] C. Yang, X. Wang, Y. Lu, H. Liu, Q. V. Le, D. Zhou, and X. Chen, “Large
language models as optimizers,” arXiv preprint arXiv:2309.03409, 2023.
[47] M. Lindauer, K. Eggensperger, M. Feurer, A. Biedenkapp, D. Deng,
C. Benjamins, T. Ruhkopf, R. Sass, and F. Hutter, “SMAC3: A versatile
bayesian optimization package for hyperparameter optimization,” Journal
of Machine Learning Research (JMLR), 2022.

