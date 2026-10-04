---
title: "ORACLE: A Multi-Objective Reinforcement Learning-Based Analog Circuit Design Optimizer with Large Language Models-Guided Exploration"
authors: "Brempong, Osei; Habib, Mohammed Ayman; Poddar, Vivan; Fayazi, Morteza"
date: "2026/08/05"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2608.04999"
pdf: "papers/2026_oracle-a-multi-objective-reinforcement-learning-based-analog-circuit-design_2608-04999.pdf"
sha256: "e4460830c89c8c00f3cd9650c848d32df82c03981e568a05ca98f3b3ddea1313"
pages_extracted: 7
---

# ORACLE: A Multi-Objective Reinforcement Learning-Based Analog Circuit Design Optimizer with Large Language Models-Guided Exploration

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

ORACLE: A Multi-Objective Reinforcement Learning-Based
Analog Circuit Design Optimizer with Large Language
Models-Guided Exploration
Osei Brempong, Mohammed Ayman Habib, Vivan Poddar, Morteza Fayazi
Email: {osei.b, m.habib, vivan.poddar, fayazi}@utah.edu
University of Utah, Salt Lake City, UT, USA
Abstract—Analog circuit design automation using reinforcement learn-
ing (RL) has emerged as a promising approach for reducing manual
effort. However, many existing RL-based methods focus on single-
objective optimization. Even methods designed for multi-objective (MO)
problems often reduce multiple design specifications to a single scalar
reward. This simplification limits the ability to capture the true Pareto
trade-off among competing objectives and often leads to suboptimal
designs. Moreover, requiring the model to be retrained from scratch
whenever the desired MO specifications change remains a key limitation.
To address these challenges, we present ORACLE, an open-source
RL-based framework for MO analog circuit design optimization that
replaces scalar reward optimization with vector-valued learning and
preference-aware conditioning. ORACLE represents a true MO analog
circuit design optimizer that uses a preference vector to specify the
relative weights of multiple objectives, enabling a single trained model to
generate designs across diverse trade-off settings without retraining. We
further propose two preference-guidance strategies, namely normalized-
weight guidance and cosine-aligned guidance, to improve convergence. In
addition, we incorporate a large language model (LLM)-guided action
selection mechanism to filter actions that are likely to lead to suboptimal
designs or increased runtime. Our results show that, on multiple circuit
topologies with 2,000 test cases, ORACLE reduces runtime by 20.4x -
104.4x compared to state-of-the-art approaches. It also meets 99.9% of
the 2,000 target specifications, and achieves 5.1x - 318.6x better figure of
merit in the resulting output specs.
Index Terms—Analog circuit design automation, open-source, multi-
objective reinforcement learning, preference vector conditioning, large
language models (LLMs), normalized weight, cosine aligned guidance.
I. INTRODUCTION
The growing demand for high-performance analog circuits in-
creases the need for automated design flows that reduce manual
effort while preserving the accuracy [1]. Traditional analog circuit
sizing methods generally fall into knowledge-based, model-based,
and simulation-based categories, each presenting challenges in adapt-
ability, data efficiency, and computational cost [2]–[5].
To address these limitations, reinforcement learning (RL) has
emerged as a promising approach for analog circuit design by learning
device-parameter update policies through repeated interaction with
circuit simulations [6], [7]. This gives RL a key advantage over
traditional design methods. Instead of relying on fixed expert heuris-
tics or separately trained surrogate models, an RL agent interacts
with the circuit environment, observes the current design state, and
applies parameter-update actions. Moreover, the RL agent updates its
policy using a reward signal derived from the circuit specifications
and design constraints. This closed-loop learning process allows the
agent to learn how changes in device parameters affect circuit-level
performance, thereby enabling more effective exploration of large and
highly constrained analog design spaces [8].
Despite these advances, most current RL-based analog circuit
design automation approaches only consider single-objective opti-
mization functions [9], [10]. This drastically limits their applications
as circuit design problems usually are multi-objective (MO) optimiza-
tion problems [11]. Even approaches designed for MO often reduce
multiple design specifications to a single scalar reward [12]. Although
such approaches simplify training, they hide the true Pareto trade-offs
among competing objectives and can therefore lead to suboptimal de-
signs. In addition, because the optimization is driven by a single scalar
reward, changes in the desired multi-objective specifications typically
require retraining the model from scratch, which is a major limitation
for practical analog circuit design automation.automation [10].
A common problem in MO-RL is the use of fixed-weight scalariza-
tion, where multiple objectives are compressed into a single reward
using predefined weights. When objectives have different numerical
values, those with larger values can dominate the optimization
process, even if they are not the most important under the desired
preference. [13], [14]. A more suitable alternative is to keep the
reward as a vector and learn policies that can recover multiple non-
dominated solutions instead of a single weighted-sum solution [15].
Preferences are still often specified by a weight vector, but a simple
weighted sum can remain biased toward high-magnitude objectives.
Cosine-aligned guidance addresses this issue by favoring actions
whose predicted value vector points in the same direction as the
preference vector [14]. In other words, it selects actions that improve
the objectives in a way that better matches the desired trade-off, rather
than simply choosing the action with the largest scalarized value.
In this work, we propose ORACLE, an open-source1
RL-based
multi-objective analog circuit design optimizer approach guided by a
large language model (LLM) agent. ORACLE replaces scalar-reward
optimization with vector-valued learning, where each design objective
has an associated reward signal and the reward is represented as a
vector rather than a single scalar value. Consistent with this setting,
ORACLE adopts a multi-objective version of Q-learning in which
the network learns vectorized Q-values instead of standard scalarized
value updates, thereby enabling explicit modeling of trade-offs across
specifications.
The goals of ORACLE are threefold: (a) produce multiple trade-
off solutions for each target spec rather than favoring a single reward
solution; (b) support preference conditioning so that a single trained
policy targets different trade-off regions by changing a preference
vector without retraining; (c) improve practical efficiency by reducing
unproductive exploration in the discrete parameter-update space.
To enable preference conditioning, ORACLE uses two preference-
guidance mechanisms. The first is normalized weight guidance, which
uses preference-weighted action selection over normalized objectives.
The second is cosine-aligned guidance, which adds a cosine-similarity
term so that the agent prefers actions that are not only numerically
1https://anonymous.4open.science/r/ORACLE-2026/README.md
arXiv:2608.04999v1
[eess.SY]
5
Aug
2026

## Page 2

large in value, but also point in the same direction as the desired
preference trade-off [14].
To improve efficiency, beyond using value guidance, ORACLE
applies LLM-guided action masking to add domain knowledge to
exploration of the discrete design space. The action masking in
discrete RL improves exploration efficiency by filtering actions that
are unlikely to be beneficial in the current design state [16], [17].
This helps the RL agent make more informed step-by-step parameter
changes.
We validate ORACLE on several circuit topologies with 2,000
test cases and compare it against state-of-the-art (SOTA) techniques.
Experimental results show that ORACLE meets 99.9% of the tar-
get specifications. ORACLE also achieves 20.4x - 104.4x runtime
speedup. Furthermore, it delivers 5.1x - 318.6x higher figure of merit
(FoM) for the resulting output specs.
The main contributions of this paper are summarized as follows:
• Proposing ORACLE, an open-source MO analog circuit design
optimizer framework that replaces scalar reward optimization with
vector-valued learning and explicit trade-off modeling.
• Introducing preference controllability through normalized weight
and cosine aligned guidance to steer optimization toward different
trade-off regions under changing specification priorities.
• Integrating LLM-guided action masking to reduce unbeneficial
exploration steps in discrete parameter update spaces, improving
both efficiency and reliability.
• Testing on a benchmark comprising 2,000 target specifications
of multiple topologies and compared against SOTA techniques
ORACLE meets 99.9% of the specifications, achieves up to a
104.4x runtime speedup, and delivers up to a 318.6x higher FoM
for the resulting designs.
II. BACKGROUND & RELATED WORK
A. Analog Circuit Design Optimization
Most analog circuit design problems are naturally multiobjective
as the desired specifications, e.g. gain, bandwidth, etc., should be
optimized simultaneously [18]. In other words, the goal of analog
circuit optimization is to determine the design parameters (e.g. DC
bias voltages and size of transistors) such that
minimize f1(x), . . . , fM (x)
subject to: c1(x) < 0, . . . , cN (x) < 0,
(1)
where f1, . . . , fM are the desired specifications of the circuit [19].
c1, . . . , cN are constraints such as xj ∈ [p−
j , p+
j ] or bandwidth (BW)
> 1GHz.
B. Analog Circuit Design Automation
Previous work on automating analog circuit optimization includes
knowledge-based, equation-based, simulation-based, and learning-
based methods. They either suffer from generalization across different
topologies or are sample-inefficient, which requires restarting the
search when the target specification changes [12], [20]–[22]. Re-
cently, RL-based methods have improved search efficiency by learn-
ing parameter-update policies directly from circuit simulation [14],
[23]. However, many of them still use a single scalar reward, which
combines all design objectives into one score and leads to only one
fixed trade-off solution [6], [12], [16].
MO-RL methods learn policies across multiple preference set-
tings instead of optimizing a single scalarized objective. Preference-
conditioned approaches, such as dynamic-weight, Envelope-based,
Circuit netlist
VDD
VDD
M1 M2
M3 M4
M5
CL
VDD
M1 M2
M3 M4
M5
CL
Circuit netlist
VDD
M1 M2
M3 M4
M5
CL
Simulation testbench
Simulation testbench
Multi-objective
target problem
Maximize: gain
Minimize: power
PM > 60º
Maximize: gain
Minimize: power
PM > 60º
User Inputs
ORACLE
Design parameters
dt = [w1, w2, …, CL]
Circuit simulation
environment
Circuit simulation
environment
Observed
performance
Action
Observation
Z*
:
Zt:
Agent: Preference-Conditioned MO-DDQN
(st, at, rt,
st+1, ft, wt)
(st, at, rt,
st+1, ft, wt)
Replay buffer
Online network
Target network
Update
Q-network
Loss
function
Loss
function
Action
at+1
*
at+1
*
LLM-Guided Action Masking
LLM
Unmasked
actions
𝒜
𝒜
ORACLE
Action space
𝒜
Action space
𝒜
Zt, dt
Z* rt, st
Obs
Normalized
Reward
Normalized
Reward
Fig. 1. Overall ORACLE framework.
and Preference-Driven MO RL (PD-MORL) methods, use the pref-
erence vector to guide policy or value learning with a single net-
work [14], [24], [25]. Other recent MO-RL methods, including Pareto
set learning and decomposition-based approaches, further improve
Pareto coverage by explicitly modeling multiple preference directions
or subproblems [26]–[28]. However, in analog circuit design, it is still
difficult to efficiently learn many diverse trade-off solutions because
the design space is complex and each evaluation requires costly
simulation. Similarly, knowledge-guided exploration has been studied
to improve RL sample efficiency by filtering clearly unpromising
actions through masking or other structured guidance [16], [17].
Furthermore, LLMs have shown promising results in analog circuit
design automation [29]–[32].
III. PROPOSED APPROACH
A. Overview
ORACLE is a preference-conditioned MO-RL framework for ana-
log circuit design optimization. The goal is to replace the scalarized
reward used in conventional RL-based sizing with a vector-valued
reward during learning. Instead of training a separate policy for
each predefined objective preference, ORACLE conditions a single
agent on a preference vector, allowing one trained model to generate
multiple trade-off solutions for the same target specification. This
addresses a key limitation of scalar-reward RL, which typically
produces only one design solution for a given training setup.
The desired circuit specifications is formulated as a MO Markov
decision process [33]. As shown in Fig. 1, the framework of ORACLE
has three main modules: (a) the environment as a normalized reward
vector that measures per-objective progress toward the target spec-
ification; (b) an LLM-guided masking module to filter undesirable
actions before simulation to reduce wasted exploration and improve
search efficiency; and (c) a preference-conditioned MO-double deep
Q-network (MO-DDQN) to learn an MO action-value function and
uses the preference vector for action selection.
B. MO Environment and Normalized Reward
ORACLE uses a simulator-based circuit environment in which the
agent iteratively updates circuit design parameters and observes the
resulting circuit specifications, e.g., gain, phase margin, etc. At each
step, the agent receives the current circuit state, selects a discrete
parameter-adjustment action, runs a circuit simulation, and obtains
feedback. Concretely, the action is a tuple of discrete choices, one
per design parameter: {“decrease”, “no change”, “increase”}. Thus,
for each parameter the agent can decrease the current parameter
by one unit, apply no change, or increase the parameter by two

## Page 3

units; the updated parameters are then clipped to valid bounds before
simulation.
At environment step t, the state st includes the current observed
circuit specifications zt, the target specifications z∗
, and the current
circuit design parameter values dt (e.g., size of transistors or capac-
itors value).
st =

zt, z∗
, dt

,
zt =
h
spec
(1)
t , . . . , spec
(N)
t
i
,
z∗
=

spec(1)∗
, . . . , spec(N)∗
,
dt =
h
param
(1)
t , . . . , param
(M)
t
i
,
(2)
where N and M are the number of target specifications and the
number of circuit design parameters, respectively.
To prevent objectives with different scales from dominating the
reward, the environment returns a normalized N-dimensional reward
vector (rt):
rt =
h
rspec(1)
t , . . . , rspec(N)
t
i⊤
. (3)
Each element in this vector represents the relative difference between
one objective and its target specification:
rspec(i)
t =
spec(i)
t − spec(i)∗
|spec(i)∗
| + ϵ
, (4)
where spec(i)∗
denotes the target specification i, and ϵ > 0 is a small
positive constant introduced to avoid instability. For specifications to
be maximized, a positive reward indicates progress toward or beyond
the target. Conversely, for specifications to be minimized, the reward
term is multiplied by −1 so that lower values yield higher rewards.
Consequently, all objectives are expressed in a unified maximization
framework.
This reward design keeps the objective terms separate during
learning and reduces the effect of scale differences across objectives.
As a result, the agent identifies whether an action improves each spec,
instead of relying on a single combined reward that may hide these
differences.
C. Preference-Conditioned MO-DDQN
The main learning module in ORACLE is a preference-conditioned
MO-DDQN. The Q-network (Qθ) takes a weight vector (ω) to
prioritize the specs and the circuit state (s) as inputs and outputs
Q-values for each action (a), where Qθ(s, a, ω) ∈ RN
[23]. In
ORACLE, the current circuit state and weight vector are combined
and passed through a multilayer perceptron, and the output predicts
the effect of each action on each design target.
To select an action, the predicted vector Q-values are converted
into a scalar score using cosine-based similarity with the preference
vector ω [14]:
Qcos(s, a, ω) = cos Qθ(s, a, ω), ω

∥Qθ(s, a, ω)∥2 , (5)
where cos(·, ·) measures the directional alignment between the pre-
dicted Q-vector and the preference vector, and ∥ · ∥2 denotes the
Euclidean norm. Thus, the cosine term favors actions aligned with
the desired trade-off direction, while the norm term favors actions
with high Q-values.
Training follows the DDQN framework with an online Q-network
Qθ, a target Q-network Qθ̄, and a replay buffer D. The two networks
share the same architecture, but the online network is updated by
gradient descent at each training step, whereas the target network
is updated less frequently to provide a more stable reference for Q-
value updates [34], [35]. The replay buffer stores past transitions and
enables random mini-batch sampling, which improves stability by
reducing temporal correlation in the training data.
a) Online and target networks: The online network Qθ is used
for action selection. Given the current state st and preference vector
wt, it predicts vector Q-values for all discrete actions,
Qθ(st, wt) ∈ R|A|×4
,
and the action is selected by scalarizing these vector Q-values with
wt and applying arg max. The target network Qθ̄ is used only during
training to compute the next-state target [35]
b) Replay buffer: The replay buffer stores transitions
et = st, at, rt, st+1, ft, wt

,
where ft ∈ {0, 1} is the terminal flag, and wt ∈ R4
is the preference
vector. During training, mini-batches are uniformly sampled from D
to improve data reuse and reduce temporal correlation. Storing wt
ensures consistency with the preference-conditioned Q-value update.
c) DDQN target computation: For each sampled transition
(st, at, rt, st+1, ft, wt), the next action is selected by the online
network and then evaluated by the target network, consistent with
the DDQN framework [34]. The best next action is
a∗
t+1 = arg max
a′∈A
Scalarize Qθ(st+1, a′
, wt), wt

, (6)
and the vector-valued DDQN target is
yt = rt + γ(1 − ft) Qθ̄(st+1, a∗
t+1, wt) , (7)
where γ ∈ [0, 1] is the discount factor [36].
d) Training update: At each gradient step, the mean-squared
error (MSE) loss is computed between the online prediction for the
executed action and the target Q-vector,
L(θ) = ∥Qθ(st, at, wt) − yt∥2
2 .
The online parameters θ are optimized by backpropagation to min-
imize L(θ). Every K steps, the target network parameters are set
equal to those of the online network:
θ̄ ← θ.
Updating Qθ̄ less frequently provides more stable targets during
learning.
e) Cosine and Normalized Weighted (NW) scalarization: In
ORACLE, Scalarize(·, w) uses cosine similarity to rank actions under
the preference vector w. This favors actions whose predicted Q-
vectors align with the desired trade-off direction. However, because
cosine similarity primarily reflects directional agreement rather than
improvement magnitude, it may become less discriminative when ob-
jective scales differ. In such cases, actions with similar directions can
receive similar scores even when their absolute gains are substantially
different.
In NW, the same DDQN update is used, but the scalarization in
Eq. (6) is replaced by a weighted-sum/NW score:
QNW(s, a, w) = w⊤
Q(s, a, w).
This addresses the limitation of cosine-based selection by consid-
ering not only the direction of the predicted Q-vector, but also the
preference-weighted size of its values. NW changes only the action-
selection step. The target computation in Eq. (7) and the rest of the
replay-based DDQN training procedure are the same as in Cosine.

## Page 4

VDD
VDD
M1
30 µA
VDD
VDD
M2
M3 M4
M5
M6 M7
M8
Cc
CL
Vo
Vin+
Vin-
Fig. 2. Two-stage OPAMP schematic.
Phase Margin (º)
74 75 76 77 78 79 80 81 82 83 84 85
0
1
2
3
4
5
6
7
8
9
10
PM vs I-Bias
I
bias
(mA)
Phase Margin (º)
74 75 76 77 78 79 80 81 82 83 84 85
0
1
2
3
4
5
6
7
8
9
10
PM vs I-Bias
I
bias
(mA)
Gain (dB)
46 47 48 49 50 51 52 53 54 55 56
0
5
10
15
20
25
30
35
40
45
Gain vs UGBW
UGBW
(MHZ)
Gain (dB)
46 47 48 49 50 51 52 53 54 55 56
0
5
10
15
20
25
30
35
40
45
Gain vs UGBW
UGBW
(MHZ)
(a) (b)
Desired Spec
:
ORACLE
:
AutoCKT
:
Desired Spec
:
ORACLE
:
AutoCKT
:
Fig. 3. Comparison of 20 randomly sampled target specifications from the
1,000 MO problem two-stage OPAMP benchmark and the corresponding
results from AutoCkt [12] and ORACLE.
The loss is defined as
L(θ) = E(st,at,rt,st+1,ft)

∥Qθ(st, at, ω) − yt∥2
2

. (8)
This loss measures the mean-squared error between the Q-vector
predicted by the online network and the target Q-vector over sampled
transitions. A smaller loss means the prediction is closer to the target.
Therefore, minimizing L(θ) trains the online network to produce
more accurate Q-vector estimates.
The advantage of this design is that a single trained model can
generate different trade-off solutions for the same target specification
by changing only the preference vector during inference. In ORA-
CLE, we instantiate this capability using a fixed set of 10 preference
vectors per target specification, so that it produces 10 solutions for
each target without retraining separate policies.
D. LLM-Guided Action Masking
Analog circuit simulation is computationally expensive, so re-
ducing unproductive exploration is important during training. To
improve exploration efficiency in the discrete design space, ORACLE
incorporates an LLM-guided action masking module. At each step,
ORACLE compares the current simulated specifications with the
target objectives by computing the specification gap
∆
(i)
t = speci
t − speci∗
, (9)
where ∆
(i)
t denotes the deviation of objective i from its target at
step t. These gaps are discretized into four qualitative categories,
such as “far below target”, “slightly below target”, “near target”,
and “meets target”, and then are used to prompt a local LLM.
Using this summarized objective-gap description, the LLM applies
circuit-design knowledge to identify parameter changes that are
unlikely to move the circuit closer to the target specifications. For
example, it may block transistor upsizing when Ibias exceeds its target,
TABLE I
PASS-RATE AND FOM COMPARISON ON 1,000 MULTI-OBJECTIVE
TWO-STAGE OP-AMP PROBLEMS.
Method Pass-rate
Average
FoM
Top-20 FoM
MODEBI [37] 9.0% 2.00 3.56
ABCMOBO [38] 75.8% 8.50 34.07
AutoCKT [12] 93.8% 0.43 0.70
ORACLE (Cosine) 100.0% 1.45 1.49
ORACLE (Cosine + LLM) 100.0% 132.3 384.3
ORACLE (NW) 100.0% 138.3 450.1
TABLE II
AVERAGE RUNTIME ON THE TWO-STAGE OP-AMP BENCHMARK.
Method
[37] [12] [38]
ORACLE
(cos)
ORACLE
(cos + LLM)
ORACLE
(NW)
Runtime
(Minutes)
180 85 59.9 6.5 3.2 2.4
or block downsizing when gain remains far below its target. In this
way, globally harmful actions are filtered before simulation.
The LLM output is converted into a binary mask mt ∈ {0, 1}|A|
,
where mt(a) = 0 indicates that action a is filtered out and mt(a) =
1 indicates that action a is retained. Action selection is then restricted
to the unmasked subset,
A′
= {a ∈ A | mt(a) = 1}
a∗
t = arg max
a∈A′
Qcos(st, a, ω),
(10)
so that the policy searches over a smaller and more relevant subset
of actions rather than the full discrete action space. This improves
sample efficiency by reducing unnecessary simulation calls.
IV. EVALUATION
We evaluate ORACLE on a two-stage OPAMP and an unconven-
tional 3-stage OTA topologies benchmark in 45nm BSIM technology
node as shown in Fig. 2 and Fig. 4. The general desired target
optimization problems for both these topologies are as follows:
maximize {G, UGBW, PM}
minimize {Ibias}
subject to: G ≥ G∗
, UGBW ≥ UGBW∗
,
PM ≥ PM∗
, Ibias ≤ I∗
bias.
(11)
Here, (·) and (·)∗
denote the achieved specification and corresponding
target specification, respectively. A solution is considered successful
only if it meets all four target specifications at the same time i.e.
G ≥ G∗
, UGBW ≥ UGBW∗
, PM ≥ PM∗
, and
Ibias ≤ I∗
bias.
The evaluation set for each topology contains 1,000 optimization
problems in (11) format, where the target specifications are sampled
from the ranges specified in (13) and (14) for each topology,
respectively. For each optimization, AutoCKT [12], which uses a
scalar reward, returns one solution. On the other hand, ORACLE
generates 10 solutions (as it is set in its preference settings) for
the same target optimization problem. This is because of the multi-
objective optimization of ORACLE, which generates multiple trade-
off solutions from a single trained model. To have a fair evaluation,
we also compare ORACLE with two MO Bayesian optimization

## Page 5

TABLE III
COMPARISON ON 10,000 GENERATED SOLUTIONS OVER THE SAME 1,000
MULTI-OBJECTIVE TWO-STAGE OPAMP BENCHMARK, WITH 10
SOLUTIONS GENERATED PER TARGET SPECIFICATION.
Method Pass-rate
Average
FoM
Top-20
FoM
MODEBI [37] 1.40% 1.89 3.77
ABCMOBO [38] 40.37% 4.82 35.92
ORACLE (Cosine) 99.19% 107.38 355.37
ORACLE (Cos + LLM) 99.45% 113.35 460.08
ORACLE (NW) 99.80% 107.46 372.40
TABLE IV
PARETO TRADE-OFF QUALITY COMPARISON ON THE TWO-STAGE OPAMP.
PF: PARETO FRONT.
Method
Mean
Hypervolume
Mean
Sparsity
Mean PF Size
AutoCKT [12] 1.40 × 105 0.00 1.00
MODEBI [37] 6.28 × 106 2.94 × 102 10.00
ABCMOBO [38] 3.29 × 106 4.99 × 102 10.00
ORACLE (Cos) 3.55 × 109 8.81 × 105 10.00
ORACLE (NW) 3.57 × 109 9.04 × 105 10.00
methods, MODEBI [37] and ABCMOBO [38], with the same 10-
solution setting. A Figure of Merit (FoM) is defined as follows to
rank these 10 solutions and select the best one:
FoM =
G − G∗
G∗
+
U − U∗
U∗
+
PM − PM∗
PM∗
+
I∗
bias − Ibias
I∗
bias
.
(12)
It should be noted that the FoM is computed only over successful
solutions, i.e., designs that satisfy all four constraints. For each
target specification, the reported FoM corresponds to the highest-
FoM successful solution.
We compare the following methods:
1) AutoCKT: A single-objective RL optimizer producing one so-
lution per target [12] which is one of our SOTA baselines.
2) MODEBI: A MO Bayesian optimizer producing 10 solution per
target [37], which is one of our SOTA baselines.
3) ABCMOBO: An asynchronous batch constrained MO Bayesian
optimizer producing 10 solution per target [38], which is one of
our SOTA baselines.
4) ORACLE (Cosine): Preference-conditioned MO-DDQN with
cosine-aligned action selection.
5) ORACLE (Cosine + LLM): Cosine-based ORACLE augmented
with LLM-guided action masking.
6) ORACLE (NW): ORACLE with normalized weighted-sum
scalarization for action selection.
Our evaluation metrics are as follows:
• Pass-rate: Measures the percentage of the generated designs that
satisfy all four target constraints simultaneously
• FoM (12): Indicates the design quality. Higher FoM values mean
better solutions.
• Hypervolume [27]: Measures how much useful objective space
is covered by the recovered non-dominated solutions.
• Sparsity: Measures how well these solutions are distributed
across the trade-off surface. Lower sparsity means the Pareto
solutions are distributed more evenly, which is better.
• Pareto front size: Measures the number of non-dominated solu-
tions obtained for each target specification.
20 µA
VDD
VDD
M6
M5
VDD
VDD
M1 M2
M3 M4
M8
M9
M10
Cc Rz M11
M12
M7
CL
Vin+ Vin- Vo
Fig. 4. Unconventional three-stage OTA schematic.
Gain (dB)
25 30 35 40 45 50 55 60
0
50
100
150
200
250
300
350
Gain vs UGBW
50 60 70 80 90 100
0
2
4
6
8
10
12
14
PM vs I-Bias
(a) (b)
Phase Margin (º)
I
bias
(mA)
UGBW
(MHZ)
Desired Spec
:
ORACLE
:
AutoCKT
:
Desired Spec
:
ORACLE
:
AutoCKT
:
Fig. 5. Comparison of 10 randomly sampled target specifications from
the 1,000 MO problem benchmark of 3-stage unconventional OTA and the
corresponding results from AutoCkt [12] and ORACLE.
ORACLE is implemented in Python using PyTorch for the NN com-
ponents. Training uses a preference-conditioned DDQN algorithm
with experience replay and ϵ-greedy exploration in the AutoCkt
multi-objective OpenAI Gym environment. The models are trained on
50 target specifications with 10 preference vectors and a maximum
episode length of 30 environment steps. For the LLM-guided method,
a local Ollama backend runs a lightweight Llama 3.2 model for action
masking and preference-boost recommendations. All experiments are
10
5
10
6
10
7
10
8
10
10
Frequency (Hz)
-20
0
20
40
Bode Magnitude
UGBW = 291 MHz
10
5
10
6
10
7
10
8 9
10
10
Frequency (Hz)
0
50
100
150
200
Bode Phase
PM = 57.9
°
Phase
(deg)
10
10
9
Magnitude
(dB)
Fig. 6. The frequency response of the ORACLE-generated circuit for Table VI
target specifications.

## Page 6

TABLE V
PASS-RATE AND FOM COMPARISON ON 1,000 MULTI-OBJECTIVE
UNCONVENTIONAL THREE-STAGE OTA PROBLEMS.
Method Pass-rate
Average
FoM
Top-20
FoM
MODEBI [37] 0.9% 0.52 3.34
AutoCKT [12] 35.7% 3.07 49.20
ABCMOBO [38] 70.4% 2.61 7.54
ORACLE (Cosine) 97.2% 8.13 29.20
ORACLE (NW) 96.4% 13.30 40.70
ORACLE (Cos + LLM) 98.4% 7.15 35.77
TABLE VI
A SAMPLE OF THE TARGET SPECS FROM THE 3-STAGE UNCONVENTIONAL
OTA BENCHMARK AND THE CORRESPONDING RESULT OF ORACLE.
Specs Gain (dB) UGBW (MHz) PM (◦) Ibias (mA)
Target > 31.7 > 4.83 > 50.7 < 13.1
ORACLE 32.4 291.6 57.9 0.53
conducted on a machine equipped with an NVIDIA GA102 GPU.
A. Two-stage OPAMP
The circuit schematic is shown in Fig. 2. The action space for each
circuit component is written in array notation: [start, end, increment].
The action space for every transistor width is [1, 100, 1]×0.5µ. Note
that the size of M1 = M2 and the size of M3 = M4. Moreover,
the action space of the compensation capacitor (Cc) is [0.1, 10.0,
0.1]×1pF. This results in 7 design parameters with a total action space
size of 1014
possible values. The design specifications of interest and
their range in our database are as follows:
Gain : [46dB, 52dB] V/V, PM : [75◦
],
UGBW : [1, 25] MHz, Ibias : [0.1, 10] mA.
(13)
Table I summarizes the overall comparison. ORACLE meets
100% of the target specifications, representing a 1,011% - 6.6%
improvement over the SOTA [12], [37], [38]. Moreover, ORACLE
(NW) delivers 318.6x - 69.15x improvement in FoM compared to
SOTA [12], [37], [38]. Also, comparing ORACLE (Cosine) and
ORACLE (Cosine + LLM) demonstrates a 91x improvement on
average FoM, indicating the positive effect of LLM integration.
Furthermore, we compare the average FoM of top twenty best designs
of each method. ORACLE (NW) achieves 302x - 1.17x better top-20
FoM compared to ORACLE (Cosine) and ORACLE (Cosine + LLM),
respectively. This shows the effectiveness of the proposed normalized
weighted-sum scalarization for action selection.
Fig. 3 shows a comparison between the output of ORACLE and
AutoCKT over 20 randomly sampled target specifications from the
1,000 MO problem benchmark. As illustrated, across all samples,
ORACLE achieves higher gain, UGBW, and phase margin than
AutoCkt, while maintaining lower Ibias. Moreover, as shown in
Table II, ORACLE reduces the average runtime by 75x - 24.8x
compared with SOTA [12], [37], [38].
We also evaluate the results of all 10,000 generated solutions. The
test is still on the same 1,000 MO problem benchmark, but now
we keep all 10 generated solutions for each MO target problem.
The results are listed in Table III. The NW-based method achieves
better performance than the cosine-based approach in all aspects. This
suggests that, in the implementation, direct weighted scoring leads
to more effective action selection and therefore improves the ability
TABLE VII
AVERAGE UNCONVENTIONAL THREE-STAGE OTA RUNTIME COMPARISON.
Method
[12] [37] [38]
ORACLE
(Cos)
ORACLE
(Cos + LLM)
ORACLE
(NW)
Runtime
(Minutes)
338 295 66.1 10.54 11.58 3.24
TABLE VIII
PARETO TRADE-OFF QUALITY COMPARISON ON THE CURRENT MIRROR
OTA. PF DENOTES PARETO FRONT.
Method
Mean
Hypervolume
Mean
Sparsity
Mean PF
Size
AutoCKT [12] 6.12 × 106 0 1
MODEBI [37] 9.91 × 106 9.46 × 105 10
ABCMOBO [38] 1.91 × 107 4.96 × 105 10
ORACLE (NW) 3.54 × 107 2.61 × 105 10
ORACLE (Cos) 1.32 × 107 2.17 × 105 10
ORACLE (Cos + LLM) 1.60 × 107 3.10 × 105 10
to find valid solutions. Moreover, the LLM-guided masking module
reduces wasted exploration and results in more passed solutions.
Table IV compares the Pareto-quality metrics. ORACLE (NW)
produces meaningful trade-off sets, with the highest mean hypervol-
ume compared to all other approaches. This shows that the ORACLE
reward formulation improves more than one operating point; it yields
a substantially richer Pareto front for each target specification.
B. Unconventional Three-stage OTA
The circuit schematic is shown in Fig. 4. The action space for
every transistor width is [4, 40, 2]×0.5µ. Moreover, the action space
of the compensation capacitor (Cc) is [0.05, 2.0, 0.05]×1pF and of
the zero-nulling resistor (Rz) is [0.1, 10.0, 0.1]×1kΩ. This results in
10 design parameters with a total action space size of 1012
possible
values. The design specifications of interest and their range in our
database are as follows:
Gain : [23dB, 53dB] V/V, PM : [45◦
, 60◦
],
UGBW : [4, 33] MHz, Ibias : [0.5, 20] mA.
(14)
Table V reports the overall comparison. ORACLE (Cos + LLM)
meets 98.4% of 1,000 target specifications, which shows 109.3x -
1.4x improvement over SOTA [12], [37], [38]. In addition, ORACLE
(NW)’s average FoM is 5.11x - 25.58x higher than SOTA [12], [37],
[38]. Also, Fig. 5 illustrates 10 of these target specifications and
the corresponding results from ORACLE and SOTA for different
specifications. Table VI lists an example of these target specs and the
ORACLE achieved results for it. Fig. 6 demonstrates the frequency
response of the ORACLE output using the unconventional 3-stage
OTA. Moreover, as shown in Table VII, ORACLE reduces the average
runtime by 104.4x - 20.4x compared with SOTA [12], [37], [38].
Table VIII lists the Pareto-front quality metrics. ORACLE (NW)
achieves the highest mean hypervolume, and ORACLE (Cosine)
achieves the lowest mean sparsity among all.
V. CONCLUSION
This work presents ORACLE, a novel open-source multi-objective
RL-based framework for analog circuit design optimization. ORA-
CLE replaces scalar-reward optimization with preference-conditioned
vector-valued learning and further integrates LLM-guided explo-
ration. By preserving objective-level information throughout training,
ORACLE recovers multiple trade-off solutions for the same target

## Page 7

specification from a single trained model. ORACLE is compared
with SOTA techniques on a benchmark with 2,000 different MO
specifications. The results show that ORACLE meets 99.9% of the
target specifications. Moreover, ORACLE delivers up to a 318.6x
higher FoM for the resulting output specs. In addition, ORACLE
achieves up to a 104.4x runtime speedup.
REFERENCES
[1] M. Fayazi, Z. Colter, E. Afshari, and R. Dreslinski, “Applications of
Artificial Intelligence on the Modeling and Optimization for Analog and
Mixed-Signal Circuits: A Review,” IEEE Transactions on Circuits and
Systems I: Regular Papers, vol. 68, no. 6, pp. 2418–2431, 2021.
[2] K. G. Liakos and F. Plessas, “Analog Design and Machine Learning: A
Review,” Electronics, vol. 14, no. 17, p. 3541, 2025.
[3] M. Fayazi, M. T. Taba, E. Afshari, and R. Dreslinski, “AnGeL: Fully-
Automated Analog Circuit Generator Using a Neural Network Assisted
Semi-Supervised Learning Approach,” IEEE Transactions on Circuits
and Systems I: Regular Papers, vol. 70, no. 11, pp. 4516–4529, 2023.
[4] A. Lberni et al., “Analog Circuit Sizing Based on Evolutionary Algo-
rithms and Deep Learning,” Expert Systems with Applications, vol. 237,
p. 121480, 2024.
[5] M. Fayazi, Z. Colter, Z. Benameur-El Youbi, J. Bagherzadeh, T. Ajayi,
and R. Dreslinski, “FASCINET: A Fully Automated Single-Board
Computer Generator Using Neural Networks,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 41,
no. 12, pp. 5435–5448, 2022.
[6] Y. Uhlmann, M. Essich, L. Bramlage, J. Scheible, and C. Curio, “Deep
Reinforcement Learning for Analog Circuit Sizing with an Electrical De-
sign Space and Sparse Rewards,” in Proceedings of the 2022 ACM/IEEE
Workshop on Machine Learning for CAD, 2022, pp. 21–26.
[7] W. Cao, J. Gao, T. Ma, R. Ma, M. Benosman, and X. Zhang, “RoSE-
Opt: Robust and Efficient Analog Circuit Parameter Optimization With
Knowledge-Infused Reinforcement Learning,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 44,
no. 2, pp. 627–640, 2024.
[8] M. Ahmadzadeh and G. G. Gielen, “Using Probabilistic Model Rollouts
to Boost the Sample Efficiency of Reinforcement Learning for Auto-
mated Analog Circuit Sizing,” in Proceedings of the 61st ACM/IEEE
Design Automation Conference, 2024, pp. 1–6.
[9] S. Kim, Z. Wang, S. Lee, Y. Oh, H. Zhu, D. Kim, and D. Z. Pan,
“PPAAS: PVT and Pareto Aware Analog Sizing via Goal-conditioned
Reinforcement Learning,” arXiv preprint arXiv:2507.17003, 2025.
[10] C. F. Hayes, R. Rădulescu, E. Bargiacchi, J. Källström, M. Macfarlane,
M. Reymond, T. Verstraeten, L. M. Zintgraf, R. Dazeley, F. Heintz,
E. Howley, A. A. Irissappane, P. Mannion, A. Nowé, G. Ramos,
M. Restelli, P. Vamplew, and D. M. Roijers, “A practical Guide to
Multi-objective Reinforcement Learning and Planning,” Autonomous
Agents and Multi-Agent Systems, vol. 36, p. 26, 2022.
[11] K. Somayaji NS and P. Li, ‘Pareto Optimization of Analog Circuits Us-
ing Reinforcement Learning,” ACM Transactions on Design Automation
of Electronic Systems, vol. 29, no. 2, p. Article 37, 2024.
[12] K. Settaluri, A. Haj-Ali, Q. Huang, K. Hakhamaneshi, and B. Nikolic,
“AutoCkt: Deep Reinforcement Learning of Analog Circuit Designs ,”
in Proceedings of the Design, Automation & Test in Europe Conference
(DATE), 2020.
[13] C. F. Hayes, R. Rădulescu, E. Bargiacchi, J. Kallstrom, M. Macfarlane,
M. Reymond, T. Verstraeten, L. M. Zintgraf, R. Dazeley, F. Heintz
et al., “A Brief Guide to Multi-Objective Reinforcement Learning and
Planning,” in Proceedings of the 2023 International Conference on
Autonomous Agents and Multiagent Systems, 2023, pp. 1988–1990.
[14] T. Basaklar, S. Gumussoy, and U. Y. Ogras, “PD-MORL: Preference-
Driven Multi-Objective Reinforcement Learning Algorithm,” in The
Eleventh International Conference on Learning Representations (ICLR),
2023.
[15] K. S. NS and P. Li, ‘Pareto Optimization of Analog Circuits Using
Reinforcement Learning ,” ACM Transactions on Design Automation of
Electronic Systems, vol. 29, no. 2, pp. 1–14, 2024.
[16] W. Cao, J. Gao, T. Ma, R. Ma, M. Benosman, and X. Zhang, “RoSE-
Opt: Robust and Efficient Analog Circuit Parameter Optimization With
Knowledge-Infused Reinforcement Learning,” IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems, vol. 44,
no. 2, pp. 627–640, 2025.
[17] Z. Wang, X. Li, L. Sun, H. Zhang, H. Liu, and J. Wang, “Learning
State-Specific Action Masks for Reinforcement Learning,” Algorithms,
vol. 17, no. 2, p. 60, 2024.
[18] J. Li, Y. Li, and Y. Zeng, “Robust Circuit Optimization under PVT
Variations Via Weight Optimization Problem Reformulation,” Expert
Systems with Applications, vol. 248, p. 123301, 2024.
[19] M. Fayazi, M. T. Taba, A. Tabatabavakili, E. Afshari, and R. Dreslinski,
“FuNToM: Functional Modeling of RF Circuits Using a Neural Network
Assisted Two-Port Analysis Method,” in 2023 IEEE/ACM International
Conference on Computer Aided Design (ICCAD), 2023, pp. 1–8.
[20] M. Barros, J. Guilherme, and N. Horta, “Analog Circuits Optimization
Based on Evolutionary Computation Techniques,” Integration, the VLSI
Journal, vol. 43, no. 1, pp. 136–155, 2010.
[21] W. Daems, G. Gielen, and W. Sansen, “An Efficient Optimization–
Based Technique to Generate Posynomial Performance Models for
Analog Integrated Circuits,” in Proceedings of the 39th annual Design
Automation Conference, 2002, pp. 431–436.
[22] M. W. Cohen, M. Aga, and T. Weinberg, “Genetic Algorithm Software
System for Analog Circuit Design,” Procedia CIRP, vol. 36, pp. 17–22,
2015.
[23] H. van Hasselt, A. Guez, and D. Silver, “Deep Reinforcement Learning
with Double Q-Learning,” in Proceedings of the AAAI Conference on
Artificial Intelligence, vol. 30, no. 1, 2016.
[24] A. Abels, D. M. Roijers, T. Lenaerts, A. Nowé, and D. Steckelmacher,
“Dynamic Weights in Multi-Objective Deep Reinforcement Learning,” in
Proceedings of the 36th International Conference on Machine Learning,
ser. Proceedings of Machine Learning Research, vol. 97, 2019, pp. 11–
20.
[25] R. Yang, X. Wang, M. Sun, X. Fang, K. Narasimhan, S. S. Du,
M. Hashemi, R. S. Sutton, and D. Schuurmans, “A Generalized Al-
gorithm for Multi-Objective Reinforcement Learning and Policy Adap-
tation,” in Advances in Neural Information Processing Systems 32, 2019.
[26] F. Felten, E.-G. Talbi, and G. Danoy, “Multi-Objective Reinforcement
Learning Based on Decomposition: A Taxonomy and Framework,”
Journal of Artificial Intelligence Research, vol. 79, pp. 679–723, 2024.
[27] E. Liu, Y.-C. Wu, X. Huang, C. Gao, R.-J. Wang, K. Xue, and C. Qian,
“Pareto Set Learning for Multi-Objective Reinforcement Learning,” in
Proceedings of the AAAI Conference on Artificial Intelligence, vol. 39,
no. 18, 2025, pp. 18 789–18 797.
[28] T. Hu and B. Luo, “PA2D-MORL: Pareto Ascent Directional Decom-
position Based Multi-Objective Reinforcement Learning,” Proceedings
of the AAAI Conference on Artificial Intelligence, vol. 38, no. 11, pp.
12 547–12 555, 2024.
[29] J. Shen et al., “Atelier: An Automated Analog Circuit Design Framework
via Multiple Large Language Model-Based Agents,” IEEE Transactions
on Computer-Aided Design of Integrated Circuits and Systems, 2025.
[30] Y. Lai et al., “AnalogCoder: Analog Circuit Design via Training-Free
Code Generation,” in Proceedings of the AAAI Conference on Artificial
Intelligence, vol. 39, no. 1, 2025, pp. 379–387.
[31] P. Abbineni et al., “MuaLLM: A Multimodal Large Language Model
Agent for Circuit Design Assistance with Hybrid Contextual Retrieval-
Augmented Generation,” in 2026 31st Asia and South Pacific Design
Automation Conference (ASP-DAC). IEEE, 2026, pp. 646–652.
[32] S. Aldowaish, Y. Karumanchi, K.-C. Chiang, S. Noorzad, and M. Fayazi,
“SINA: A Circuit Schematic Image-to-Netlist Generator Using Artificial
Intelligence,” Design, Automation and Test in Europe Conference and
Exhibition (DATE), 2026.
[33] M. Rezaei and H. Nezamabadi-Pour, “A taxonomy of Literature Reviews
and Experimental Study of Deep Reinforcement Learning in Portfolio
Management,” Artificial Intelligence Review, vol. 58, p. 94, 2025.
[34] V. Mnih et al., “Human-level control through deep reinforcement learn-
ing,” Nature, vol. 518, no. 7540, pp. 529–533, 2015.
[35] H. van Hasselt, A. Guez, and D. Silver, “Deep Reinforcement Learning
with Double Q-Learning,” in Proceedings of the AAAI Conference on
Artificial Intelligence, vol. 30, no. 1, 2016, pp. 2094–2100.
[36] D. Hafner, J. Pasukonis, J. Ba, and T. Lillicrap, “Mastering Diverse
Control Tasks through World Models,” Nature, vol. 640, pp. 604–610,
2025.
[37] C. Vişan et al., “Automated Circuit Sizing with Multi-Objective Op-
timization Based on Differential Evolution and Bayesian Inference,”
Knowledge-Based Systems, vol. 258, p. 109987, 2022.
[38] X. Zhao et al., “Asynchronous Batch Constrained Multi-Objective
Bayesian Optimization for Analog Circuit Sizing,” in 29th Asia and
South Pacific Design Automation Conference (ASP-DAC). IEEE, 2024.

