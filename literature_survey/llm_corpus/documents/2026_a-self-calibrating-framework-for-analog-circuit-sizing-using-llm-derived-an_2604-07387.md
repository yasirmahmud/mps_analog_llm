---
title: "A Self-Calibrating Framework for Analog Circuit Sizing Using LLM-Derived Analytical Equations"
authors: "Bujana, Antonio J.; Karsilayan, Aydin I."
date: "2026/04/08"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2604.07387"
pdf: "papers/2026_a-self-calibrating-framework-for-analog-circuit-sizing-using-llm-derived-an_2604-07387.pdf"
sha256: "9a65f0aa595ca6ab583cc8da02edd5f83755c802d4940e2599ef027d0c647dc9"
pages_extracted: 14
---

# A Self-Calibrating Framework for Analog Circuit Sizing Using LLM-Derived Analytical Equations

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

1
This work has been submitted to the IEEE for possible publication. Copyright may be transferred without notice, after which this version may
no longer be accessible.
A Self-Calibrating Framework for Analog Circuit
Sizing Using LLM-Derived Analytical Equations
Antonio J. Bujana1,2
, Graduate Student Member, IEEE, and Aydin I. Karsilayan1
, Member, IEEE
Abstract— We present a design automation framework for
analog circuit sizing that produces calibrated, topology-specific
analytical equations from raw circuit netlists. A large language
model (LLM) derives a complete Python sizing function in which
each device dimension is traceable to a specific design rationale
— a form of interpretable output absent from existing
optimization-based and LLM-based sizing methods. A
deterministic calibration loop extracts process-dependent
parameters from a single DC operating point simulation, while a
prediction-error feedback mechanism compensates for analytical
inaccuracies. We validate the framework on circuits ranging
from 8 to 30 transistors — spanning two-stage Miller-
compensated, current-mirror, folded cascode, nested Miller-
compensated, and complementary class-AB output topologies —
across three process nodes (40 nm, 90 nm, 180 nm). On matched-
specification benchmarks, including the class-AB opamp case, the
framework converges in 2–7 simulations. Despite large initial
prediction errors, convergence depends on the measurement-
feedback architecture, not prediction accuracy. The one-shot
calibration automatically captures process-dependent variations,
enabling cross-node portability without modification, retraining,
or per-process characterization.
Index Terms— Analog circuit sizing, computer-aided design
(CAD), electronic design automation (EDA), interpretable design
equations, large language model (LLM), self-calibration.
I. INTRODUCTION
NALOG circuit sizing remains a labor-intensive task
requiring expert knowledge to navigate complex
tradeoffs among competing performance metrics —
including gain, bandwidth (gain-bandwidth product, GBW),
phase margin (PM), slew rate (SR), and power consumption
— that characterize analog circuit design. Despite decades of
research into automated sizing — spanning optimization-
based, knowledge-based, and large language model (LLM)-
based methods (reviewed in Section II) — to the authors’
knowledge, no existing approach produces executable,
topology-specific design equations directly from raw circuit
netlists, and no prior work provides quantitative analysis of
how prediction accuracy relates to convergence.
This paper introduces a fundamentally different role for the
LLM: deriving the analytical design equations themselves.
The LLM receives a raw circuit netlist and produces a
complete Python sizing function (compute_sizing())
1
Department of Electrical and Computer Engineering, Texas A&M
University, College Station, TX, USA. 2
Department of Engineering,
Benedictine College, Atchison, KS, USA. Correspondence to: A. J. Bujana
tonybujana@tamu.edu
encoding topology-specific equations — equations consistent
with standard analog design theory, but generated
automatically from the netlist. A deterministic calibration loop
then anchors these equations in process-accurate simulation
data by extracting process-dependent parameters from a single
DC operating point (OP) simulation. The key insight is that
LLM-generated equations need only be structurally correct —
even if their numerical predictions are inaccurate. We define
structural correctness as the property that the equations (i)
correctly identify which devices govern each specification, (ii)
correctly capture the direction of each device's influence, and
(iii) correctly represent how specifications trade off against
each other through shared devices. Structural correctness is
essential: it ensures that the prediction-error feedback drives
the design toward convergence rather than oscillation. When a
specification falls short, structurally correct equations direct
the LLM to adjust the right devices in the right direction, even
if the quantitative adjustment is imprecise. The calibration
loop corrects numerical errors, and the prediction-error
feedback compensates for systematic analytical inaccuracies
that persist after calibration. We validate the framework
against existing methods on matched circuits and
specifications, and analyze topology-dependent convergence
behavior across process nodes.
The main contributions of this work are as follows:
1) An end-to-end, self-calibrating sizing workflow
validated on circuits ranging from 8 to 30 transistors
across five topology families, including single-stage,
two-stage, and three-stage compensation architectures.
Unlike existing LLM-based sizing approaches that
produce only final device dimensions, this framework
preserves calibrated, human-readable design equations
— commented Python code tracing each device
dimension to a specific design rationale. The only per-
circuit inputs are the netlist, target specifications, and
supply voltages; all scripts and prompt templates are
shared across topologies and nodes with no modification.
2) A calibration feedback architecture that decouples
convergence from the LLM's numerical prediction
accuracy. The LLM-derived equations must be
structurally correct — correctly mapping specifications
to governing devices and influence directions — but
need not be numerically accurate. Quantitative
prediction-error analysis across 78 simulation rounds
confirms this decoupling (Section V-B).
3) Cross-node portability via self-calibration. The one-shot
DC OP extraction automatically captures process-
dependent parameters (μCox, αgm, λ, Vth) that vary
significantly across process nodes, enabling the same
A

## Page 2

2
framework to self-calibrate at 40 nm, 90 nm, and 180 nm
without modification, per-process characterization data,
or retraining.
II. PRIOR WORK
Existing automated sizing approaches fall into three broad
categories: optimization-based methods that treat the circuit as
a black box, knowledge-based methods that rely on pre-
characterized device data, and LLM-based methods that
leverage language models in various roles. Each is reviewed
below.
A. Optimization-Based Methods
Black-box optimization methods formulate circuit sizing as
a numerical optimization problem. Bayesian optimization
(BO) [1] uses Gaussian process surrogates to efficiently
explore the design space. Traditional implementations require
hundreds to thousands of simulations to converge, though
recent work on initialization strategies [2] has substantially
reduced this for OTA-class circuits — including two-stage
Miller-compensated, folded cascode, and telescopic cascode
topologies at 45 nm CMOS — achieving design closure in as
few as 21 total simulations. Geometric programming [3]
achieves faster convergence through convex formulations but
requires the designer to manually construct posynomial
models for each topology. Reinforcement learning methods
[4] train agents through simulation rewards but require per-
topology training episodes and produce no analytical insight.
These methods achieve good performance but treat the circuit
as a black box — the final sizing comes with no interpretable
rationale explaining why specific dimensions were chosen.
B. Knowledge-Based Methods
The gm/ID methodology [5] represents the most widely
adopted knowledge-based approach. It uses pre-characterized
lookup tables relating transconductance efficiency to inversion
level, enabling systematic operating-point selection. However,
it requires per-process characterization sweeps (typically W
and L sweeps across the full operating range), and a human
expert must interpret efficiency curves, select operating points,
and manually derive the design equations linking
specifications to device dimensions. These two requirements
— manual equation derivation and per-process
characterization — represent the primary bottlenecks of the
gm/ID approach.
C. LLM-Based Sizing Methods
The landscape of LLM-based analog sizing methods has
expanded rapidly. These approaches can be categorized by the
role assigned to the LLM.
1) LLM as optimizer: EEsizer [6] uses an LLM directly to
propose sizing adjustments based on simulation feedback. The
LLM analyzes performance gaps and suggests parameter
changes iteratively. Testing across eight LLMs on basic
circuits, the authors scaled to a 20-transistor opamp and
demonstrated cross-node applicability across 180 nm, 130 nm,
and 90 nm, achieving target specifications across multiple test
groups. The approach relies on the LLM's ability to propose
good parameter adjustments — a capability that varies
significantly across models and topologies.
2) LLM as search-space guide: LEDRO [7] uses an LLM to
iteratively reduce the design search space for a TuRBO
optimizer. By having the LLM propose refined parameter
ranges based on simulation feedback, LEDRO reduces the
exploration complexity for the downstream optimizer. Tested
on 22 op-amp topologies across four FinFET nodes, it
demonstrates broad generalizability. However, convergence
ultimately depends on the TuRBO optimizer, and the LLM's
role is limited to search-space pruning rather than analytical
reasoning about circuit behavior.
3) LLM as knowledge transfer agent: LLM-USO [8]
introduces structured knowledge representations encoding
relationships between performance metrics, circuit sub-
structures, and sizing parameters. This enables transfer
learning across circuits with similar sub-structures via a
knowledge summary mechanism, mirroring expert designers'
ability to reuse insights. The framework uses 45 simulations
per circuit. ADO-LLM [9] combines in-context learning with
BO, using the LLM to propose promising design points within
a BO framework.
4) LLM as multi-agent system: AnaFlow [10] employs
specialized LLM agents that collaborate in a multi-agent
workflow, providing structured reasoning and explainability.
Atelier [11] extends this with a graph-of-thoughts architecture
integrating a curated knowledge base, achieving superior
performance over black-box methods on specific topologies.
The framework uses 100–400 simulation evaluations per
sizing. AutoSizer [12] introduces a reflective meta-
optimization framework where LLM agents adaptively select
and configure optimization algorithms within a two-loop
architecture, tested on 24 circuits including oscillators,
switched-capacitor circuits, and filters. TopoSizing [13]
performs circuit understanding from raw netlists using graph
algorithms to organize circuits into hierarchical device–
module–stage representations, then uses LLM agents in a
hypothesis-verification loop to produce annotations that guide
BO. While TopoSizing produces structured circuit
annotations, it does not derive executable analytical sizing
equations.
5) Lightweight and cross-node methods: EasySize [14] fine-
tunes a lightweight Qwen3-8B model for cross-node
applicability, achieving strong performance at 180 nm, 45 nm,
and 22 nm nodes despite training only on 350 nm data.
LLMACD [15] operates in transistor behavioral-parameter
space, using a behavioral model to map LLM-generated
parameters to device sizes with manually embedded design
knowledge. AmpAgent [16] uses literature retrieval to port
amplifier designs across processes and performance targets.

## Page 3

3
III. PROPOSED METHODOLOGY
Despite the breadth of existing approaches (Section II), LLM-
based sizing methods share a common architectural pattern:
the LLM serves as either an optimizer — suggesting
parameter adjustments based on simulation feedback — or a
search-space guide that reduces the region for a downstream
optimizer. Table I positions the proposed framework against
these methods; the key differentiators are detailed in the
contributions (Section I).
Existing methods that claim explainability (AnaFlow,
Atelier) provide reasoning traces explaining why the LLM
made specific parameter adjustments — valuable for
understanding the optimization trajectory, but not equivalent
to calibrated design equations. TopoSizing [13] produces
structured annotations but uses them to guide BO rather than
deriving executable sizing equations. The proposed framework
outputs a complete sizing function where each line of code
maps a specification to device dimensions through explicit
analytical relationships. This provides two distinct advantages
beyond explainability. First, traceability for design review:
before committing to layout, a designer can inspect which
specification constrained each device dimension — a property
valued in production design flows that black-box methods
cannot provide. Second, diagnostic transparency: when
convergence is slow or a specification is marginal, the
calibrated equations identify whether the root cause is a
parametric error (e.g., inaccurate μCox) or a fundamental
topological limitation (e.g., the topology cannot
simultaneously meet all specifications at the given supply
voltage). As discussed in Section VI-A, the equations also
provide the LLM with a causal model of the circuit that
anchors its sizing adjustments across rounds — ensuring
convergent rather than oscillatory behavior.
A. Framework Overview
The proposed framework operates in iterative rounds, as
shown in Fig. 1. Each round comprises four steps: prompt
assembly, LLM equation generation or update, sizing
execution, and simulation with calibration.
1) Round 0: A prompt-assembly script
(build_prompt.py) combines the circuit netlist, target
specifications, constraint rules, and a structured template into
a single prompt. The LLM receives this prompt and produces
two outputs: (i) initial estimates of process-dependent device
parameters for every transistor (μCox, αgm, λ, Vth), informed by
device constraints specified in the netlist, and (ii) a Python
sizing function, compute_sizing(), encoding the
topology's design equations. This task requires the LLM to
interpret the raw netlist — recognizing the circuit topology,
identifying device roles and signal paths, and deriving
structurally correct design equations directly from the netlist.
While the calibration feedback architecture is model-agnostic
in principle, the framework's effectiveness depends on the
LLM’s netlist-interpretation capability. A deterministic script
(solve.py) executes the sizing function using the LLM's
parameter estimates, producing design variables (W, L, bias
currents, etc). These are simulated in Cadence Virtuoso using
the Spectre circuit simulator. The testbench includes stability
TABLE I COMPARISON WITH EXISTING METHODS
Method LLM Role Interpretable
Equations
Pre-characterization Cross-Node Prediction-Error
Analysis
BO [1] None No
(black box)
None Re-run required*
N/A
BO-Init [2] None No
(black box)
Initial samples
(16+)
Not demonstrated
(45 nm only)
No
gm/ID [5] None Partial
(efficiency curves)
Lookup tables Manual porting N/A
EEsizer [6] Optimizer No None Yes (3 nodes) No
LEDRO [7] Search-space guide No None Yes (4 nodes) No
LLM-USO [8] Knowledge transfer No None Not specified No
AnaFlow [10] Multi-agent Partial
(reasoning traces)
None Not demonstrated
(45 nm only)
No
Atelier [11] Multi-agent + KB Partial Knowledge base Yes (2 nodes) No
AutoSizer [12] Meta-optimizer No None Not demonstrated
(130 nm only)
No
TopoSizing [13] Circuit understanding Partial
(annotations)
None Not demonstrated
(55 nm only)
No
EasySize [14] Fine-tuned optimizer No Training data
(350nm)
Yes (4 nodes)†
No
This work Equation derivation Yes
(Python code)
None
(self-calibrating)
Yes (3 nodes) Yes
(78 rounds)
*
‘Re-run required’ indicates the method must be re-executed from scratch at each new node
†
Trained on 350 nm; tested at 180 nm, 45 nm, and 22 nm nodes.

## Page 4

4
(STB), transient, AC, noise, and DC analyses for all circuits;
the metrics extracted for convergence checking depend on the
target specifications. DC OP analysis extracts per-device gm,
gds, Vth, Id, Vgs, and Vds. Finally, a calibration script
(calibrate.py) processes the simulation results: it extracts
four calibration parameters per device from the DC OP,
computes predicted-versus-actual performance, and assembles
the Round 1 prompt.
2) Round N ≥ 1: The LLM receives the prompt assembled
by the previous round's calibration output and generates only
an updated sizing function — it does not re-estimate device
parameters, as these are now provided by calibrate.py's
simulation-extracted values. The updated function is executed
by solve.py, Cadence simulates, and calibrate.py
extracts updated parameters and assembles the next prompt.
The loop completes when all specifications are met.
B. One-Shot Calibration
In each round, the calibration extracts device-level
parameters from a single DC OP simulation. No additional test
structures, characterization sweeps, or lookup tables are
required. For each transistor in the circuit, calibrate.py
computes four parameters.
1) Effective transconductance parameter μCox (A/V²):
Derived from the saturation current equation
Id = ½·μCox·(W/L)·Vov², where Vov = Vgs − Vth is the overdrive
voltage. This captures the combined effect of carrier mobility,
oxide capacitance, and mobility degradation at the actual
operating point.
2) Transconductance non-ideality factor αgm: The ratio of
actual gm to the ideal value 2·Id/Vov. This factor accounts for
velocity saturation and short-channel effects that reduce gm
below the long-channel prediction. In the tested processes, αgm
is typically below 1.0, depending on device type, channel
length, and overdrive voltage.
3) Channel-length modulation coefficient λ (1/V):
Computed as gds/Id, where gds is the output conductance from
the OP. This parameter varies significantly with channel
length and is critical for determining output resistance and
voltage gain.
4) Threshold voltage Vth (V): Extracted directly from the
OP, including body-effect shifts for devices with non-
grounded source terminals.
These four parameters capture the dominant process-specific
effects — velocity saturation, channel-length modulation
variation with L, body-effect threshold shifts, and mobility
degradation — that vary significantly between process nodes.
The one-shot extraction automatically adapts to the process
node without modification to the calibration scripts. Since
these parameters depend on the device's operating point
(which changes as W, L, and bias currents are adjusted each
round), calibrate.py re-extracts all four parameters from
every simulation, providing the LLM with updated values that
reflect the current sizing.
The operating-point dependence of the extracted parameters
is substantial. Table II shows representative calibration data
for the differential-pair input device (M1) sampled across
different topologies, process nodes, and rounds. The topology
abbreviations are: 2SMC, two-stage Miller-compensated; CM,
current-mirror; FC, folded cascode; with -N and -P denoting
NMOS- and PMOS-input variants, respectively. These
samples were drawn from different points in the design space
and illustrate three key observations.
First, μCox varies by 2.9× (1,614 to 4,710 μA/V²) among
NMOS devices at the same node and channel length (40 nm,
L = 0.20 μm), driven by operating-point differences. Second,
for the same topology and device between rounds (2SMC-P at
180 nm, R5 vs. R8), μCox shifts by 2.1× (211 to 98 μA/V²) as
the sizing evolves, confirming the necessity of per-round re-
extraction. Third, αgm is below 1.0 in all Table II entries,
varying from 0.21 to 0.79 — a 3.8× range — consistent with
actual transconductance falling well below the ideal long-
channel value.
The calibration data is presented to the LLM in a structured
build_prompt.py
LLM: (Fresh Session)
• Sizing function:
compute_sizing
()
• Initial estimate: 𝜇𝐶ox , 𝛼𝑔m, 𝜆, 𝑉th
(Round 0 only)
Inputs: Circuit netlist, target specs, prompt template design rules
START
calibrate.py
• Extract 𝜇𝐶ox , 𝛼𝑔m, 𝜆, 𝑉th
• Compute predicted vs
measured error
• Assemble next-round
prompt
Specs
met?
Cadence Virtuoso / Spectre
• Performance metrics
• DC OP per device
𝐼d, 𝑉
gs , 𝑉ds , 𝑔m, 𝑔ds , 𝑉th
Design
variables
𝑊, 𝐿, 𝐼𝑏𝑖𝑎𝑠 …
solve.py
• Executes sizing
function
• Outputs design
variables
Round 0
Prompt
END
Yes
No
Round N
Prompt
TABLE II CALIBRATION PARAMETER VARIATION (DEVICE M1)
Topology Node Round Type L (μm) Vov (mV) μCox (μA/V²) αgm λ (1/V)
2SMC-N 40nm R0 NMOS 0.2 49 1614 0.36 0.64
CM-N 40nm R2 NMOS 0.2 43 1934 0.32 0.65
FC-N 40nm R5 NMOS 0.2 27 4710 0.21 0.56
2SMC-P 180nm R5 PMOS 0.3 44 211 0.36 0.31
2SMC-P 180nm R8 PMOS 0.3 89 98 0.58 0.26
FC-P 180nm R8 PMOS 0.5 178 64 0.79 0.08
Fig. 1. Self-calibrating framework architecture. The loop iterates
until all target specifications are met.

## Page 5

5
per-device table (Fig. 2) that includes the device type, W/L,
measured current, overdrive voltage, transconductance,
saturation status, and all four extracted parameters. The table
also flags devices operating in the triode region (|Vds| < Vov),
which alerts the LLM to bias-point failures requiring
correction in the next round.
C. Prediction-Error Feedback
After each simulation round, calibrate.py computes the
discrepancy between the LLM's predicted performance and the
Cadence-measured results. The next-round prompt presents a
side-by-side comparison and includes metric-type-specific
margin formulas embedded directly in the prompt text. For
linear metrics (e.g., GBW), the instruction reads: "if over-
predicted by X%, design for (1 + X/100) × target." For
logarithmic metrics (e.g., Av in dB): "if over-predicted by Y
dB, design for target + Y dB." For phase margin: additive
degree margin — “if over-predicted by Z°, design for
target + Z°." These are explicit, formulaic instructions — not
vague guidance — enabling the LLM to compute corrected
design targets arithmetically. A cumulative historical log of all
prior rounds is included, enabling the LLM to identify
persistent prediction biases and distinguish between
parametric errors (which shrink with calibration) and
analytical model errors (which persist across rounds).
The prediction-error feedback also includes targeted
diagnostic guidance derived from failure modes observed
during framework development. For example, a PM prediction
exception instructs the LLM that if its predicted PM is
catastrophically low but the Cadence-measured PM passes, the
analytical PM equation is inaccurate and the LLM should trust
the measured value rather than distorting the sizing to fix a
prediction artifact. Such instructions encode domain
knowledge about common analytical failure modes into the
feedback loop; additional static constraints embedded in the
prompt template are detailed in Section III-D.
This feedback is critical because certain analytical
inaccuracies — such as systematic GBW over-prediction
caused by unmodeled parasitic capacitances — are inherent in
the analytical model, not in the extracted parameters, and
persist after calibration. The feedback compensates by
instructing the LLM to add design margin proportional to the
observed over-prediction.
D. Structured Prompt Design
The prompt enforces a multi-section output structure that
requires the LLM to perform systematic topology analysis
before writing any sizing code. The Round 0 prompt mandates
seven sections in sequence:
(1) Device Roles — labeling each transistor's function (e.g.,
"NMOS diff-pair input," "PMOS diode-connected active
load")
(2) Device Constraints — classifying each device
(3) Signal Path — tracing the signal from input to output
(4) Design Equations — deriving symbolic expressions for
gain, bandwidth, slew rate, and compensation
(5) DC Bias Verification — checking that all branch
voltages support device saturation
(6) DC OP Consistency — verifying numerical
compatibility of inter-stage node voltages
(7) Python Code with initial device estimates
This forced decomposition prevents the LLM from jumping
directly to code generation without first reasoning about the
circuit structure — a failure mode observed during early
development that produced syntactically correct but
topologically wrong sizing functions.
Within this structure, several specific constraints encode
analog design knowledge that the LLM lacks.
1) Device classification rules require the LLM to label each
transistor as MIRROR (sized by width ratio W/Wref relative to
a reference device), MATCHED (identical W and L to a
partner), or INDEPENDENT (sized directly by a performance
equation). Each classification enforces a specific sizing
formula in the Python code: mirror devices use
W = Wref × Itarget/Iref × L/Lref, matched devices copy their
partner's dimensions, and independent devices use the square-
law equation W = 2·Id·L / (μCox·Vov²). Mirror devices must use
the same channel length as their reference device, because Vth
depends on L in short-channel processes. Violating this
constraint was observed to cause mirror current errors
exceeding 50%.
2) An anti-circularity rule requires the LLM to compute
inter-stage node voltages strictly from the driving stage's
device parameters (e.g., Vnet5 = Vdd − Vth,p − Vov,8), preventing
circular reasoning in which the LLM assumes a bias point and
Dev Type W/L |Ids| Vov Vds gm Region mu_Cox a_gm lam ro Vth
(um/um) (uA) (mV) (mV) (uS) (uA/V2) (1/V) (kOhm) (mV)
M1 NMOS 0.7/0.20 6.97 49.0 526.4 101.37 SAT 1613.7 0.356 0.6368 225.2 438.2
M2 NMOS 0.7/0.20 7.22 55.6 435.3 103.12 SAT 1298.4 0.397 0.6756 204.9 438.2
M3 PMOS 1.3/0.20 6.97 68.7 517.4 89.24 SAT 447.8 0.440 0.4032 355.6 448.7
M4 PMOS 1.3/0.20 7.22 69.3 608.5 91.87 SAT 455.9 0.441 0.3745 369.5 448.1
M5 NMOS 2.0/0.20 14.20 83.0 56.2 145.49 TRIODE 404.1 0.425 12.9403 5.4 430.2
M6 NMOS 1.7/0.20 25.00 83.0 513.2 313.43 SAT 864.0 0.520 0.5812 68.8 430.2
M7 NMOS 4.8/0.20 72.68 83.0 543.4 907.97 SAT 879.1 0.518 0.5663 24.3 430.2
M8 PMOS 5.2/0.20 72.68 160.0 556.6 634.49 SAT 220.1 0.698 0.3326 41.4 448.5
WARNING: M5 in TRIODE (|Vds|=56mV < Vov=83mV)
Fig. 2. Calibration table from calibrate.py output for 2SMC-N at 40 nm, as provided to the LLM in the Round 1 prompt. M5 is flagged
as operating in the triode region.

## Page 6

6
then verifies it against itself.
3) DC OP consistency rules require the LLM to show
explicit numerical computation at every constrained node,
verifying that the voltage one stage produces is compatible
with what the next stage requires. The prompt requires explicit
arithmetic at every headroom check, prohibiting the LLM
from making vague assertions such as 'self-adjusts' or
'consistent by construction’ without numerical verification.
4) A parasitic capacitance warning informs the LLM that
large device widths create parasitic Cgs and Cdb at internal
nodes, degrading bandwidth and phase margin. The prompt
instructs the LLM to consider reducing W through higher
overdrive voltage as a first recourse before attempting other
compensation adjustments. This single prompt addition
resolved a systematic convergence failure observed during
development in which the LLM's response to failing PM was
to increase current or device widths, inadvertently worsening
the problem.
The LLM's output for each topology is a complete,
commented Python function (compute_sizing()) that
typically spans 200–300 lines. Fig. 3 shows a representative
excerpt from the 2SMC-P 180 nm Round 0 output. Three
features are notable: the LLM derives a joint GBW/SR
constraint that selects the binding specification automatically,
it applies all three device classification types
(INDEPENDENT, MIRROR, MATCHED) with the correct
sizing formula for each, and it cancels the RHP zero by setting
R = 1/gm8 — all without human guidance.
The framework requires no topology-specific prompt
tuning. The prompt structure applies identically across circuits
ranging from 8-transistor two-stage Miller-compensated
circuits to a 30-transistor class-AB opamp, and across process
nodes from 40 nm to 180 nm.
IV. EXPERIMENTAL SETUP
A. Technology and Specifications
The framework is evaluated at three process nodes through
two sets of experiments.
• 180 nm CMOS: Vdd = +0.9 V, Vss = −0.9 V.
• 90 nm CMOS: Vdd = +0.6 V, Vss = −0.6 V.
• 40 nm CMOS: Vdd = +0.55 V, Vss = −0.55 V.
1) OTA validation: The first set validates generality across
OTA topologies and process nodes. Six OTA topologies
(Section IV-B) are tested at 180 nm (CL = 1 pF) and 40 nm
(CL = 0.5 pF), with target specifications shown in the Target
rows of Table III. The 40 nm technology supports higher
operating frequencies due to shorter channel lengths and
higher device ft, motivating the 2× higher GBW and SR
targets. The voltage gain (Av) target is reduced from 60 dB to
40 dB and CL from 1 pF to 0.5 pF to maintain comparable
design difficulty; the reduced voltage headroom (1.1 V vs. 1.8
V) tightens the coupling between gain, bandwidth, and bias-
point feasibility, while the higher GBW and SR targets
demand greater transconductance and bias current under
tighter headroom constraints.
2) Matched-specification comparisons: The second set
validates the framework against four published methods on
matched circuits and specifications: James et al. [2] (BO-based
sizing at 40 nm), LLM-USO [8] (LLM-augmented BO at 90
nm), Atelier [11] (multi-agent LLM at 180 nm), and EEsizer
[6] (LLM-as-optimizer at 180 nm). Each comparison uses five
independent trials with target specifications, load capacitance,
and load resistance matched to the reference work. Full details
are provided in Section IV-C.
B. Topologies Tested
Three OTA topology families are tested, each with NMOS-input
and PMOS-input variants, yielding six topologies per node and
12 total. All circuits are internally biased through current mirrors
driven by a single ideal reference current source (Iref). The two-
stage Miller-compensated topologies also include an ideal
# -- Current design: jointly set by GBW and SR --
# Relationship: SR_internal = 2*pi*GBW*Vov1
SR_internal_from_GBW = (
2.0 * math.pi * GBW * Vov1)
if SR_internal_from_GBW >= SR_target:
GBW_design = GBW # GBW-limited
else:
GBW_design = ( # SR-limited
SR_target / (2.0 * math.pi * Vov1))
Cc = 0.5e-12
gm1_target = 2.0 * math.pi * GBW_design * Cc
Itail = gm1_target * Vov1
Id1 = Itail / 2.0
# Second-stage current: output SR and PM
Id2_SR = SR_target * CL
gm8_for_fp2 = ( # fp2 >= 2.2*GBW
2.2 * GBW * 2.0 * math.pi * CL)
Id2_fp2 = gm8_for_fp2 * Vov8 / 2.0
Id2 = max(Id2_SR, Id2_fp2)
# -- Device widths --
# M6: INDEPENDENT (PMOS ref, carries Iref)
W6 = 2.0*Iref*L6 / (mu_Cox_p * Vov6**2)
# M5: MIRROR of M6, carries Itail
W5 = W6 * (Itail / Iref) * (L5 / L6)
# M7: MIRROR of M6, carries Id2
W7 = W6 * (Id2 / Iref) * (L7 / L6)
# M1, M2: MATCHED diff pair (each carries Id1)
W1 = 2.0*Id1*L1 / (mu_Cox_p * Vov1**2)
W2 = W1
# M8: INDEPENDENT (NMOS CS gain, carries Id2)
W8 = 2.0*Id2*L8 / (mu_Cox_n * Vov8**2)
# -- Compensation (nulling RHP zero) --
gm8_sized, Vov8_v = calc_gm("M8", Id2, W8, L8)
R = 1.0 / gm8_sized
Fig. 3. Excerpt from the LLM-generated compute_sizing()
function for 2SMC-P at 180 nm, Round 0. The complete function
(~200 lines) is executed unmodified.

## Page 7

7
compensation capacitor (Cc) and nulling resistor (Rz); the single-
stage topologies (folded cascode, current-mirror) require no
Miller compensation network. The NMOS-input variant of each
family is shown in Fig. 4(a)-(c). The PMOS-input variants use
the complementary device types with the same topology
structure.
1) Current-mirror OTA with cascode output (CM-N, CM-P):
Fig. 4(a). 18 transistors in a single-stage topology where the gain
mechanism relies on current-mirror amplification with cascode
devices at the output for high output impedance.
2) Folded cascode (FC-N, FC-P): Fig. 4(b). 18 transistors in a
single-stage topology with cascode loads providing high output
impedance. The folded cascode represents the most constrained
topology due to tight coupling between gain, bandwidth, and
phase margin in a single stage.
3) Two-stage Miller-compensated (2SMC-N, 2SMC-P): Fig.
4(c). 8 transistors plus a Miller compensation capacitor Cc and
nulling resistor Rz. The two-stage architecture decouples gain
from bandwidth through the compensation network.
In addition to the six OTA topologies, two additional circuits
are tested for comparison with published methods:
4) Nested Miller-compensated opamp (NMC): Fig. 4(d). 16
transistors in a three-stage topology with two series RC Miller
compensation networks (R1-C1, R2-C2) and a feedforward path.
The NMC introduces nested pole-zero interactions across three
gain stages, making it the most complex compensation
architecture tested. This circuit is used for comparison with
Atelier [11], which validates on NMC topologies at 180 nm.
5) Complementary class-AB opamp (30T): Fig. 4(e). 30
transistors with complementary NMOS/PMOS input stages,
folded cascode first stage, a class-AB push-pull output stage, and
two Miller compensation capacitors (CC1,CC2). All 30 transistors
— including 10 bias transistors — are treated as design variables
sized by the LLM's analytical equations. For comparison,
EEsizer [6] validates a 20-transistor variant of this topology in
which bias voltages are set externally, resulting in fewer design
variables. This is the most complex circuit tested in terms of
biasing.
Testing five topology families — three with both input
polarities at two process nodes, plus NMC and 30T at 180 nm —
provides 16 independent validation points for the framework's
generality across circuit complexity levels, compensation
architectures, and process nodes. The 2SMC-N and FC-N
topologies also serve as comparison circuits against James et al.
[2] at 40 nm, and the FC-N against LLM-USO [8] at 90 nm, with
matched specifications documented in Section IV-C.
C. Matched-Specifications Comparison Methodology
To enable controlled comparison with published methods, we
applied the framework to circuits and specification targets
matching those reported in four recent works. For each
comparison, we match the circuit topology family and
specification targets while noting differences in process
technology. All comparison experiments use five independent
trials with fresh LLM sessions conducted in a single sitting to
minimize variability from model serving conditions.
1) James et al. [2]: 2SMC-N and FC-N at 40 nm, targeting
Av ≥ 45 dB, GBW ≥ 20 MHz, PM ≥ 45°, Power ≤ 200 μW.
James et al. validate at 45 nm; we use 40 nm with matched
specifications. Load capacitance was not specified in [2]; we use
CL = 0.5 pF as a conservative estimate of parasitic loading.
2) LLM-USO [8]: FC-N at 90 nm, targeting Av ≥ 60 dB, unity-
gain frequency (UGF) ≥ 1 MHz, common-mode rejection ratio
(CMRR) ≥ 80 dB, CL = 1 pF. The process node is not specified
in [8]; we adopt 90 nm while matching their specification targets.
3) Atelier [11]: NMC at 180 nm, targeting Av > 85 dB,
GBW > 0.7 MHz, PM > 55°, Power < 50 μW, input offset
(Offset) < 3 mV, input-referred noise (Noise) < 500 nV/rtHz at 1
kHz, transistor area (Area) < 250 μm2
, CL = 10 pF, RL = 1 MΩ.
4) EEsizer [6]: 30-transistor opamp at 180 nm, targeting
Av ≥ 65 dB, UGBW ≥ 10 MHz, PM ≥ 50°, Power ≤ 10 mW,
CMRR ≥ 100 dB, total harmonic distortion (THD)≤ -26 dB,
input-referred offset (Offset) ≤ 1 mV, Output Swing ≥ 1.53 V,
input common-mode range (ICMR) ≥ 1.53 V, CL = 10 pF,
RL = 1 kΩ. EEsizer specifies swing and ICMR as Vdd − Vss (1.8
V) with a 5% tolerance applied to all metrics; we adopt 85% of
Vdd − Vss (1.53 V) as the swing and ICMR target, consistent with
practical class-AB output headroom requirements at this supply
voltage. EEsizer validates a 20-transistor version of this
TABLE III CONVERGENCE RESULTS
Topology Family Node Sims Av
(dB)
GBW
(MHz)
PM
(°)
SR+
(V/μs)
SR−
(V/μs)
Target — 180nm — ≥ 60 ≥ 100 ≥ 60 ≥ 50 ≥ 50
2SMC-N Miller 180nm 6 60.4 101.3 74.8 97.2 83.5
2SMC-P Miller 180nm 3 64.5 139.4 65.0 70.4 170.4
CM-N Current mirror 180nm 2 61.8 115.6 60.2 95.1 99.7
CM-P Current mirror 180nm 4 64.4 118 64.5 63.3 106.8
FC-N Folded cascode 180nm 8 60.6 100.4 63.6 64.4 68.8
FC-P Folded cascode 180nm 16 60.2 102.3 60.1 100.1 82.5
Target — 40nm — ≥ 40 ≥ 200 ≥ 60 ≥ 100 ≥ 100
2SMC-N Miller 40nm 8 43.1 215.6 66.6 183.8 129.5
2SMC-P Miller 40nm 3 50.4 231.8 71.4 105.5 144.0
CM-N Current mirror 40nm 4 45.3 337.6 80.0 194.1 131.9
CM-P Current mirror 40nm 7 40.8 241.6 76.6 115.2 171.7
FC-N Folded cascode 40nm 9 44.1 302.1 66.6 107.2 137.2
FC-P Folded cascode 40nm 8 42.3 371.4 82.7 131.8 205.0

## Page 8

8
topology with a manually designed bias circuit; our
implementation includes all 30 transistors as design variables.
D. Experimental Controls
The following controls ensure reproducibility and prevent data
contamination: Each LLM call uses a fresh session with no
conversation history, as discussed in Section VI-B. Each
topology is tested independently with no cross-topology
learning. The primary LLM is Claude Opus 4.6 (Anthropic),
accessed through the claude.ai web interface. The 30T opamp
was additionally tested with GPT-5.4 and o3 (OpenAI) to
evaluate cross-model generalization. Only the structured prompt
and calibration data are provided — no per-topology design
hints, manual parameter adjustments, or iterative coaching. All
compute_sizing() code generated by the LLM is executed
unmodified.
The Cadence testbench is identical across all topologies,
configured in unity-gain feedback. The low-frequency gain (Av)
at 1Hz, GBW, and PM are extracted from Spectre's loop-gain
STB analysis, while SR is measured from a transient large-signal
step response. CMRR is obtained from the ratio of differential to
common-mode gain in AC at 1Hz, with nominal DC voltage set
M1 M2
Vi- Vi+
Vout
M7 M8
M9 M10
M5
Iref
M6
VDD
VSS
M3 M4 M13
M11
M12
M14
C2
C1
M16
M15
R2
R1
Biasing Amplifier
M1 M2
M3 M4
M5
M6 M7
M8
Cc
Rz
Iref
VDD
VSS
Vi- Vi+ Vout
M1 M2
Vi- Vi+
Vout
M7 M8
M11 M12
M9 M10
M13 M14
M5
Iref
M21
M23
M24
M26
M25
M29
M30
VDD
VSS
M3 M4
Vi-
Vi+
M16
M15
M18
M17
M19
M20
CC1
CC2
M6
M22
M27
M28
Biasing Amplifier
M1 M2
Vi- Vi+
Vout
M3 M4
M7 M8
M9 M10
M5
Iref
M6
M13
M14
M16
M15
M18
M17
VDD
VSS
M11
M12
Biasing Amplifier Biasing Amplifier
M1 M2
Vi- Vi+
Vout
M3 M4
M7
M8
M5 M6
M9 M10
M11
Iref
M12
M13
M14
M16
M15
M18
M17
VDD
VSS
(a) (b)
(c) (d)
(e)
Fig. 4. Schematic diagrams of the tested topologies. (a) Current-mirror (CM-N). (b) Folded cascode (FC-N). (c) Two-stage
Miller-compensated (2SMC-N). (d) Nested Miller-compensated (NMC). (e) Complementary class-AB opamp (30T). NMOS-
input variants shown for (a)–(c).

## Page 9

9
to 0 V. Input-referred noise is the spot noise density at 1 kHz.
Power and input-referred offset are computed from the DC OP,
and THD is computed from a transient simulation with a
100 kHz sinusoidal input at an amplitude matching the swing
target. Finally, ICMR and output swing are measured using a DC
sweep, identifying the input and output voltage ranges over
which dVout/dVin ≥ 0.9.
Convergence is defined as simultaneously meeting all target
specifications in a single simulation. Simulation counts include
Round 0 (the initial sizing from LLM estimates); thus, a
topology converging at Round N requires N + 1 total
simulations. Each OTA topology–node combination was tested
in a single run (Table III), while the matched-specification
comparisons (Tables IV–VIII) use five independent trials per
configuration. Due to the inherent non-determinism of LLM
responses, round counts may vary across different runs; the
single-run OTA results should be viewed as demonstrations of
feasibility, while the five-trial comparisons provide statistical
validation. All experiments were conducted between February
and April 2026.
Practical deployment. The three deterministic scripts
(build_prompt.py, solve.py, calibrate.py) are fully
reproducible and execute in seconds. The non-deterministic
component is the LLM's equation generation, which produces a
~300-line Python function per round. The initial round requires
one LLM call (~2,000 input tokens, ~8,000 output tokens) and
one Cadence simulation (30-45 seconds); subsequent rounds
require larger input (~5,000 tokens, including the previous sizing
function and calibration data) and smaller output (~3,500 tokens,
the updated sizing function). These token counts reflect the 30T
opamp, the most complex circuit tested; simpler topologies
require fewer tokens. The framework can be deployed with any
LLM capable of reliably interpreting circuit netlists and deriving
governing design equations from them; the fresh-session
architecture requires no persistent LLM state between rounds.
The prompt templates, design-rule constraints, circuit netlists,
Cadence OCEAN simulation scripts, and Python automation
code are available from the authors upon request.
V. RESULTS
Sections V-A through V-C present convergence results and
prediction-error analysis for the 12 OTA topology–node
combinations. Sections V-D through V-G present the matched-
specification comparisons against [2], [8], [11], and [6],
respectively. Sections V-H and V-I analyze convergence
behavior and structural correctness.
A. Convergence Summary
Table III presents convergence results for all 12 OTA
topology–node combinations, each meeting all target
specifications. Of the 12 combinations, 11 converge in 2–9
simulations (median: 6.5). The CM-N at 180 nm converges
fastest (2 simulations), while the FC-P at 180 nm is the sole
outlier at 16 simulations — analyzed in Section V-H. At 40
nm, all six topologies converge within 3–9 simulations with
no outlier, despite the LLM's substantially worse initial
predictions at this node, as detailed below.
These single-trial results demonstrate framework generality
across topology families and process nodes. Statistical
validation via five independent trials per configuration is
provided in the matched-specification comparisons (Sections
V-D through V-G).
B. Prediction Accuracy and Cross-Node Comparison
The cross-node comparison reveals a pattern that directly
supports a key claim of this work: convergence depends on the
measurement-feedback architecture, not on prediction
accuracy. The initial predictions (Round 0) at 40 nm are
substantially worse than at 180 nm: gain over-predictions
range from +23.7 to +57.6 dB (versus +11.3 to +34.4 dB at
180 nm); GBW over-predictions reach +469 MHz at 40 nm
versus +61 MHz at 180 nm, with the worst case (CM-N)
predicting 562 MHz against an actual 93 MHz — a 6.0x error.
Despite this, all six 40 nm topologies converge within 3–9
simulations because the framework feeds back Spectre-
measured performance, not predictions, at every round.
The decoupling also holds within each node: at 40 nm, CM-
N has the largest Round 0 GBW error (+469 MHz) yet
converges fastest (4 simulations), while 2SMC-N has the
smallest gain error (+23.7 dB) yet requires 8 simulations.
Convergence speed correlates with topology-specific
feasibility margins (Section V-H), not with initial numerical
prediction accuracy. The comparison experiments reported in
Sections V-D through V-G provide further confirmation
across additional circuits, specifications, and a third process
node (90 nm), with consistent convergence in 2–7 simulations
despite comparably large initial prediction errors.
C. Prediction Error Patterns
Aggregating across all 78 simulation rounds, several
systematic patterns emerge. Gain prediction improves
dramatically after calibration: Round 0 errors range from +11
to +58 dB but reduce to within ±6 dB by the converging
round, confirming the effectiveness of the one-shot μCox and λ
calibration. The remaining three metrics — GBW, PM, and
SR — are all affected by parasitic device capacitances (Cgs,
Cdb) that the calibration loop does not extract. The four
extracted parameters (μCox, αgm, λ, Vth) capture resistive and
transconductance behavior, but nodal capacitances — which
include significant parasitic contributions beyond the explicit
CL and CC — remain uncalibrated. This is most pronounced in
GBW, which is over-predicted in 90% of rounds with a mean
error of +87%. PM prediction errors are large but
bidirectional, reflecting the sensitivity of phase margin to
multiple interacting poles and zeros whose positions shift with
parasitic loading. SR errors are further compounded by
inaccurate bias-current estimation before calibration: incorrect
μCox values produce wrong mirror ratio estimates, directly
affecting the tail and output-stage currents that set SR = I/C. In
all cases, the prediction-error feedback compensates for these
analytical limitations by instructing the LLM to add design

## Page 10

10
margin proportional to the observed discrepancy.
With these convergence and prediction characteristics
established, Sections V-D through V-G compare against
published sizing methods, summarized in Table IV.
D. Comparison with BO-Based Sizing [2]
James et al. [2] require 16 initial design points to seed a
Gaussian process surrogate, achieving design closure in 21–28
total simulations. The proposed framework requires no initial
samples, converging in 2.4 ± 0.5 simulations (mean ± std) for
the 2SMC and 5.0 ± 1.2 for the FC across five independent
trials. Beyond simulation count, the two approaches differ in
output form: BO produces a point design (final device
dimensions), whereas the proposed framework additionally
produces calibrated analytical equations constituting an
auditable sizing rationale. The framework also self-calibrates
across nodes without modification, whereas BO-based
workflows are typically trained on simulation data from a
specific node. Table V presents the detailed comparison.
E. Comparison with LLM-Augmented BO [8]
LLM-USO [8], combines Bayesian optimization with LLM-
generated knowledge summaries to enable transfer learning
across circuits sharing similar substructures. Their framework
uses 45 total simulations (5 initial LLM-generated points plus
40 simulations from 20 BO-LLM iterations) and produces
structured natural-language summaries describing parameter-
performance relationships. The proposed framework
converges in 2.6 ± 0.5 simulations with no initialization phase
and produces executable analytical equations rather than
descriptive summaries. Both methods use the folded cascode
topology at 90 nm. Table VI presents the detailed comparison.
F. Comparison with Multi-Agent LLM Methods [11]
Atelier [11] integrates multiple LLM agents with a curated
knowledge base in a graph-of-thoughts architecture.
Specialized agents handle circuit analysis, topology selection,
and design decisions, while parameter tuning is performed by
an evolutionary optimizer using 100–400 transistor-level
simulation evaluations. Although Atelier's full pipeline
encompasses topology selection and modification in addition
to sizing, the comparison here is on the sizing stage: both
methods start from a given NMC topology at 180 nm with the
same specifications, and we compare the resulting
performance and simulation cost. The proposed framework
converges in 3.0 ± 1.0 simulations from the raw netlist alone
— requiring no curated knowledge base, topology library, or
pre-built design templates — and produces calibrated
analytical equations in executable form, whereas Atelier
produces reasoning traces and final dimensions through its
multi-agent workflow. The NMC result is architecturally
significant: three-stage nested compensation introduces
interacting pole-zero dynamics substantially more complex
than two-stage Miller compensation, confirming the
framework's applicability beyond single-stage and two-stage
topologies. Table VII presents the detailed comparison.
G. Comparison with LLM-as-Optimizer [6]
EEsizer [6] uses an LLM directly as a sizing optimizer,
iteratively proposing parameter adjustments based on
simulation feedback. The authors benchmarked eight LLMs
on basic circuits and selected three — OpenAI o3, GPT-4.1,
TABLE IV CONVERGENCE COMPARISON SUMMARY
Method Circuit Transistors Node Sims (mean ± std) Success
This work 2SMC-N 8 40nm 2.4 ± 0.5 5/5
This work FC-N 18 40nm 5.0 ± 1.2 5/5
James et al. [2] 2SMC/FC 8–15 45nm 21–28 1/1
This work FC-N 18 90nm 2.6 ± 0.5 5/5
LLM-USO(C) [8] FC 16 90nm 45 1/1
This work NMC 16 180nm 3.0 ± 1.0 5/5
Atelier [11] NMC 13-16*
180nm 100–400 10/10
This work (Opus) 30T opamp 30 180nm 4.6 ± 1.1 5/5
This work (GPT) 30T opamp 30 180nm 4.4 ± 1.1 5/5
EEsizer [6] 20T opamp 20 180nm 20.6†
2/5
*
Amplifier core is 13 transistors; up to 3 additional bias transistors not shown in [11].
†
Unsuccessful runs capped 25-iteration per [6]; two converged, three capped.
TABLE V COMPARISON WITH JAMES ET AL [2]
Metric Target 2SMC-N
(mean ± std)
FC-N
(mean ± std)
James [2]
Sims — 2.4 ± 0.5 5.0 ± 1.2 21–28
Success rate — 5/5 5/5 1/1
Sims per trial — 2,2,2,3,3 5,7,4,5,4 —
Av (dB) ≥ 45 46.5 ± 1.1 45.9 ± 0.7 48.3
GBW (MHz) ≥ 20 41.2 ± 18.6 29.4 ± 5.1 21.0
PM (°) ≥ 45 71.0 ± 8.6 87.6 ± 2.0 46.1
Power (mW) ≤ 0.2 0.098 ± 0.049 0.056 ± 0.005 0.150
*
This work at 40 nm, CL = 0.5 pF; [2] at 45 nm, CL not specified.
TABLE VI COMPARISON WITH LLM-USO [8]
Metric Target FC-N
(mean ± std)
LLM-USO(C) [8]
Sims — 2.6 ± 0.5 45
Success rate — 5/5 1/1
Sims per trial — 3,3,2,3,2 —
Av (dB) ≥ 60 68.6 ± 6.2 63.0
UGF (MHz) ≥ 1 4.9 ± 4.3 2.18
CMRR (dB) ≥ 80 90.7 ± 10.6 85.2

## Page 11

11
and Gemini 2.0 Flash — for the 20-transistor opamp at
180 nm, with a maximum of 25 iterations per attempt. All
three models achieved a 2/5 success rate under a 5% tolerance
criterion applied to all metrics. Detailed results are presented
for Gemini 2.0 Flash, the strongest reported model at the 180
nm node in [6]; under an exact-target interpretation, gain falls
short in both tolerance-passing attempts and phase margin
falls short in one. The three failed attempts exhibited failures
across multiple metrics — including gain as low as 28 dB in
one attempt and offset exceeding 6 mV in another —
indicating sizing challenges beyond a single specification. In
EEsizer's formulation, the 10 bias transistors are excluded
from the optimization — their operating points are derived
from a manually designed bias circuit — yielding fewer
design variables.
The proposed framework operates on the full 30-transistor
circuit, treating all devices, including bias transistors, as
design variables, and achieves 5/5 success in 4.6 ± 1.1
simulations with Opus 4.6 and 4.4 ± 1.1 simulations with
GPT-5.4. For this comparison, swing and ICMR targets are set
to 85% of Vdd − Vss (1.53 V), reflecting practical output
headroom; all other metrics are evaluated against exact target
values with no tolerance. A third model, o3, converged in one
trial but did not reliably maintain structurally correct equations
across rounds and is therefore excluded from the multi-trial
results. Notably, direct prompting of Opus 4.6 and GPT-5.4
without the calibration loop — providing only the netlist and
target specifications in a standard conversational session —
also failed to converge, supporting the conclusion that
convergence is a property of the calibration feedback
architecture, not of the LLM alone. As with the previous
comparisons, a key qualitative distinction is that the proposed
framework produces calibrated analytical equations rather
than final device dimensions alone. The resulting EEsizer
comparison is summarized in Table VIII.
H. Convergence Analysis: The FC-P 180 nm Outlier
This section analyzes the most difficult convergence case
from the OTA results (Table III) — a phenomenon not
TABLE VII COMPARISON WITH ATELIER [11]
Metric Target NMC
(mean ± std)
Atelier [11]
Sims — 3.0 ± 1.0 100–400
Success rate — 5/5 10/10
Sims per trial — 4,3,2,2,4 —
Av (dB) ≥ 85 96.9 ± 4.3 102.3
GBW (MHz) ≥ 0.7 1.28 ± 0.53 7.569
PM (°) ≥ 55 74.3 ± 17.5 66.5
Power (mW) ≤ 0.05 0.04 ± 0.01 0.037
Offset (mV) ≤ 3 0.65 ± 0.84 0.095
Noise (nV/√Hz ) ≤ 500 378.6 ± 74.8 146.3
Area (µm²) ≤ 250 50.6 ± 13.4 49.8
†
This work: mean ± std across 5 independent trials. Atelier:
best result across 10 independent runs, as reported in [11].
observed in the matched-specification comparisons (Tables
IV–VIII), where all trials converged within seven simulations.
The PMOS-input folded cascode at 180 nm required 16
simulations to converge — nearly twice the next-longest case
(FC-N at 40 nm, 9 simulations). The near-miss trajectory
reveals an extremely narrow feasible region, illustrated by the
following representative rounds:
• R4: Av = 59.3 (short by 0.7 dB), PM = 46.9° (short by
13.1°) — GBW passes at 115.3 but gain and PM fail.
Pushing GBW higher cost gain and PM.
• R10: Av = 60.5 dB passes, GBW = 99.9 MHz (short by
0.1 MHz), PM = 58.7° (short by 1.3°) — three metrics
within 2% of target but two fail.
• R13: Av = 59.9 dB (short by 0.1 dB), all other metrics
pass — fails by 0.1 dB on gain. The subsequent R14
similarly fails only PM by 0.3°.
• R15: Av = 60.2, GBW = 102.3, PM = 60.1°,
SR+ = 100.1, SR− = 82.5 — all pass with narrow
margins.
This trajectory illustrates how a narrow feasible region
increases the number of rounds required for convergence. SR
specifications are met throughout; the difficulty lies in the
tight coupling among Av, GBW, and PM in this single-stage
topology, where shared devices and parasitic effects cause
TABLE VIII COMPARISON WITH EESIZER [6]
Metric Target 30T (mean±std) 30T (mean±std) EEsizer 20T [6]1
LLM Model — Opus 4.6 GPT-5.4 Gemini 2.0 Flash
Transistors — 30 (var. bias) 30 (var. bias) 20 (fixed bias)
Sims — 4.6 ± 1.1 4.4 ± 1.1 13 / 15
Success — 5/5 5/5 2/5
Sims per trial — 4, 3, 5, 6, 5 4, 5, 4, 3, 6 13, 15, fail, fail, fail
Av (dB) ≥ 65 80.1 ± 8.5 83.3 ± 10.3 64.0 / 62.52
UGBW (MHz) ≥ 10 15.5 ± 2.7 12.0 ± 2.8 12.6 / 50.1
PM (°) ≥ 50 72.1 ± 9.0 75.8 ± 4.3 53.6 / 49.5
Power (mW) ≤ 10 1.96 ± 1.24 1.38 ± 0.82 9.73 / 5.10
CMRR (dB) ≥ 100 103.8 ± 1.6 108.3 ± 8.1 127.1 / 120.7
Swing (V) ≥ 1.533
1.69 ± 0.03 1.64 ± 0.06 1.72 / 1.71
ICMR (V) ≥ 1.533
1.70 ± 0.03 1.64 ± 0.06 1.75 / 1.72
Offset (mV) ≤ 1 0.04 ± 0.02 0.06 ± 0.07 0.16 / 0.46
THD (dB) ≤ −26 −55.8 ± 1.6 −54.6 ± 2.6 −26.0 / −27.3
Some measurement methods differ between works; see Section IV-D for details.
1
A 5% tolerance is applied to all metrics in [6]; this work applies no tolerance except where noted.
2
Below target without 5% tolerance.
3
Set to 85% of Vdd − Vss, reflecting practical output headroom; [6] specifies Vdd − Vss with 5% tolerance (1.71 V)

## Page 12

12
improving one metric to worsen another. Within the first 8
rounds, Av and PM were within 1–2 units of their targets
(Av ≈ 58–59 dB, PM ≈ 59°), and GBW had stabilized near
105–111 MHz, indicating the framework reached the feasible
region quickly. The remaining 8 rounds navigated this narrow
tradeoff space, with the per-round pass/fail pattern (R13 fails
only Av by 0.1 dB; R14 fails only PM by 0.3°) confirming
tight metric coupling rather than systematic failure. The
margins at convergence (0.2 dB, 0.1°, 2.3 MHz) confirm the
topology operates near its feasibility boundary.
Notably, FC-P converges in 8 simulations at 40 nm —
among the longest at 40 nm but far fewer than the 16 required
at 180 nm. The difference reflects the interaction between
specification targets and topology constraints: the higher gain
target at 180 nm (60 dB vs. 40 dB) pushes the folded cascode
closer to its feasibility boundary, narrowing the design space
further. This confirms that convergence difficulty is driven by
how tightly the specifications constrain a given topology, not
by the process node itself.
I. Structural Correctness: Representative Analysis
Structural correctness — the requirement that the LLM
correctly identifies which devices govern each specification
and the direction of their influence — is central to the
convergence claim. While 100% convergence across all tested
configurations provides indirect evidence of structural
soundness, this section provides direct verification by
examining LLM-generated equations from a representative
excerpt of the 2SMC-N topology at 40 nm.
Table IX shows the LLM's equation for each specification
alongside the governing devices it identifies, and the
correctness assessment based on standard analog circuit
theory. The LLM correctly identified: (a) the two-stage gain
decomposition with the appropriate gm and ro terms for each
stage; (b) that GBW is set by the first-stage gm1 and
compensation capacitor, not by the second stage; (c) that SR+
is limited by the tail current charging Cc (internal slewing),
while SR− is limited by the output-stage current discharging
CL (external slewing) — a distinction specific to Miller-
compensated topologies; (d) that the non-dominant pole
depends on gm8/CL, making M8 the governing device for phase
margin; and (e) the RHP zero cancellation strategy using the
nulling resistor. These device-to-specification mappings are
consistent with standard two-stage OTA design theory and
ensure that the prediction-error feedback steers sizing
adjustments in the correct direction. The folded cascode and
current-mirror topologies require the LLM to additionally
identify cascode bias voltage generation, folded current
steering paths, and multi-branch mirror networks across 18
transistors. Convergence of the 16-transistor NMC and 30-
transistor class-AB opamp further extends this evidence to
three-stage compensation and complex bias networks,
confirming structural soundness beyond the OTA topologies
analyzed in detail here.
VI. DISCUSSION
A. Why the Framework Works
The framework succeeds by decomposing analog sizing into
three complementary tasks, each handled by the component
best suited to it. First, the LLM provides structural circuit
understanding: it derives fresh, topology-specific equations
from each netlist, providing a causal model of the circuit that
anchors its sizing adjustments across rounds — ensuring
convergent rather than oscillatory behavior. This causal
anchoring is what distinguishes the framework from LLM-as-
optimizer approaches, where the LLM proposes adjustments
without an explicit model of how specifications relate to
device dimensions. This is what enables the framework to
generalize across topologies without per-circuit modification:
the equations emerge from the LLM's structural analysis of
each individual netlist.
Second, the calibration loop provides parametric accuracy
that the LLM cannot achieve from first principles. Third, the
prediction-error feedback provides robustness against
analytical model errors that persist after calibration. Together,
these three components explain why convergence speed
correlates with topology-specific feasibility margins (Section
V-H) rather than with prediction accuracy (Section V-B) —
the measurement infrastructure, not the equation quality,
TABLE IX STRUCTURAL CORRECTNESS VERIFICATION – LLM-DERIVED EQUATIONS FOR 2SMC-N (40 NM)
Specification LLM-Derived Equation Governing Devices Direction Correct?
DC Gain Av = gm1 (ro2 ‖ ro4) · gm8 (ro8 ‖ ro7) M1/M2 (1st-stage gm)
M3/M4 (1st-stage load)
M8 (2nd-stage gm)
M7 (2nd-stage load)
✓ Increasing gm1 or gm8, or increasing ro of
load devices, increases gain
GBW GBW = gm1 / (2π·Cc) M1 (input transconductance)
Cc (compensation)
✓ Increasing gm1 or decreasing Cc
increases GBW
SR+ SRpos = Itail / Cc M5 (tail current source),
Cc
✓ Increasing tail current or decreasing Cc
increases positive SR
SR− SRneg = I7 / CL M7 (output-stage current sink),
CL
✓ Increasing M7 current increases
negative SR
PM PM ≈ 90° − tan-1
(ωugb / p2),
p2 = gm8/CL
M8 (non-dominant pole via gm8),
CL
✓ Increasing gm8 pushes p2 higher,
improving PM
RHP Zero z = gm8/ Cc,
cancelled by R = 1/gm8
M8 (zero location)
R = Rz (nulling resistor)
✓ Setting R = 1/gm8 moves zero to infinity

## Page 13

13
determines the convergence trajectory.
B. Architectural Robustness and Structural Correctness
While the deterministic calibration scripts bound the impact
of parametric errors — measured values override analytical
estimates at every round — the framework does require that
the LLM's equations be structurally correct. A structurally
incorrect equation (e.g., predicting that gain increases when it
actually decreases with a parameter change) would drive the
design away from the target rather than toward it. Section V-I
provides empirical verification of this property. The failure of
o3 to converge reliably on the 30T opamp (Section V-G)
further illustrates this requirement. This property distinguishes
the proposed approach from black-box methods: the LLM
must understand which devices govern each specification, not
merely search a parameter space.
The fresh-session design (no conversation history across
rounds) is a deliberate methodological choice representing a
worst-case operating mode. The LLM receives each round's
prompt without memory of prior rounds, so convergence
depends entirely on the information provided by
calibrate.py, not on the LLM's reasoning continuity or
incremental tuning. This demonstrates that convergence is a
property of the framework architecture; a conversational mode
with cumulative history could potentially reduce round counts
but is not required.
C. Scalability
The framework successfully sizes circuits from 8 transistors
(two-stage Miller-compensated) through 16 transistors (three-
stage NMC) to 30 transistors (complementary class-AB
opamp), spanning single-stage, two-stage, and three-stage
compensation architectures — all using the same prompt
structure without modification. The NMC converges in 3.0 ±
1.0 simulations despite being the most complex compensation
architecture tested, and the 30T class-AB opamp converges in
4.6 ± 1.1 simulations with all 30 devices — including bias
transistors — sized through LLM-derived equations. The
framework also operates across three process nodes (40 nm,
90 nm, 180 nm) without modification, with the one-shot
calibration automatically capturing node-specific parameters.
D. Runtime
Total wall-clock time per round is dominated by the LLM
response, which varies from approximately 5 to 20 minutes
depending on server load and model reasoning depth; the
Cadence simulation (30–45 seconds) and Python execution
(negligible) contribute minimally. For a typical 5-round
convergence, total time ranges from approximately 25 minutes
to 2 hours.
VII. CONCLUSION
We have demonstrated a design automation framework for
analog circuit sizing that combines LLM-derived analytical
equations with one-shot DC OP calibration and prediction-
error feedback. The framework achieves all target
specifications on circuits ranging from 8 to 30 transistors
across three process nodes, spanning five topology families
that include single-stage, two-stage, and three-stage
compensation architectures — despite large initial prediction
errors, showing that convergence is driven by the
measurement-feedback architecture rather than by numerical
equation accuracy. In the matched-specification comparisons,
convergence requires 2–7 simulations across all successful
trials, fewer than the reported baselines under matched circuits
and specifications. The calibrated equations provide an
interpretable, auditable sizing rationale — the primary
differentiator of this work — a property absent from both
black-box optimization and existing LLM-based sizing
methods. Future work includes extension to broader circuit
classes and cross-model evaluation across all topologies. This
demonstrated ability of a frontier LLM to interpret raw circuit
netlists, analyze device roles and bias conditions, and derive
structurally correct design equations — without training, fine-
tuning, or per-topology prompt customization — suggests that
LLMs can serve as circuit-aware design automation tools,
complementing rather than replacing the optimization-based
and knowledge-based methods that dominate the current
landscape.
ACKNOWLEDGMENT
AI tools were used to assist in manuscript preparation. The
authors take full responsibility for the content of this paper.
REFERENCES
[1] W. Lyu, F. Yang, C. Yan, D. Zhou, and X. Zeng, "Batch Bayesian
optimization via multi-objective acquisition ensemble for automated
analog circuit design," in Proc. Int. Conf. Mach. Learn. (ICML), 2018, pp.
3306–3314.
[2] A. James, Z. H. Kong, S. K. Ata, K. M. M. Aung, K. S. Yeo, C. S. Foo,
and A. James, "The initialization effect: Refining Bayesian optimization
for circuit design," IEEE Trans. Circuits Syst. Artif. Intell., vol. 3, no. 1,
pp. 38–47, 2026.
[3] M. del Mar Hershenson, S. P. Boyd, and T. H. Lee, "Optimal design of a
CMOS op-amp via geometric programming," IEEE Trans. Comput.-
Aided Design Integr. Circuits Syst., vol. 20, no. 1, pp. 1–21, Jan. 2001.
[4] K. Settaluri, A. Haj-Ali, Q. Huang, K. Hakhamaneshi, and B. Nikolić,
"AutoCkt: Deep reinforcement learning of analog circuit designs," in
Proc. Design, Autom. Test Eur. Conf. (DATE), 2020, pp. 490–495.
[5] P. G. A. Jespers and B. Murmann, Systematic Design of Analog CMOS
Circuits: Using Pre-Computed Lookup Tables. Cambridge, U.K.:
Cambridge Univ. Press, 2017.
[6] C. Liu and D. Chitnis, "EEsizer: LLM-based AI agent for sizing of analog
and mixed signal circuit," IEEE Trans. Circuits Syst. I, Reg. Papers, 2026,
doi: 10.1109/TCSI.2025.3646359.
[7] D. V. Kochar, H. Wang, A. P. Chandrakasan, and X. Zhang, "LEDRO:
LLM-enhanced design space reduction and optimization for analog
circuits," in Proc. IEEE Int. Conf. LLM-Aided Design (ICLAD), 2025,
pp. 141–148.
[8] N. S. K. Somayaji and P. Li, "LLM-USO: Large language model-based
universal sizing optimizer," IEEE Trans. Comput.-Aided Design Integr.
Circuits Syst., vol. 45, no. 4, pp. 1907–1920, Apr. 2026, doi:
10.1109/TCAD.2025.3604643.
[9] Y. Yin, Y. Wang, B. Xu, and P. Li, "ADO-LLM: Analog design Bayesian
optimization with in-context learning of large language models," in Proc.
IEEE/ACM Int. Conf. Comput.-Aided Design (ICCAD), 2024, pp. 1–9.
[10]M. Ahmadzadeh, K. Chen, and G. Gielen, "(Invited Paper) AnaFlow:
Agentic LLM-based workflow for reasoning-driven explainable and
sample-efficient analog circuit sizing," in Proc. IEEE/ACM Int. Conf.
Comput.-Aided Design (ICCAD), 2025, doi:
10.1109/ICCAD66269.2025.11240818.

## Page 14

14
[11]J. Shen, Z. Chen, J. Zhuang, J. Huang, F. Yang, L. Shang, Z. Bi, C. Yan,
D. Zhou, and X. Zeng, "Atelier: An automated analog circuit design
framework via multiple large language model-based agents," IEEE Trans.
Comput.-Aided Design Integr. Circuits Syst., vol. 45, no. 1, pp. 31–44,
2026.
[12]X. Yu, D. Torbunov, S. Mandal, and Y. Ren, "AutoSizer: Automatic
sizing of analog and mixed-signal circuits via large language model
(LLM) agents," arXiv preprint arXiv:2602.02849, 2026.
[13]Z. Wei, Z. Kong, Y. Wang, D. Z. Pan, and X. Tang, "TopoSizing: An
LLM-aided framework of topology-based understanding and sizing for
AMS circuits," arXiv preprint arXiv:2509.14169, 2025.
[14]X. Wu et al., "EasySize: Elastic analog circuit sizing via LLM-guided
heuristic search," arXiv preprint arXiv:2508.05113, 2025.
[15]S. Xu, H. Zhi, J. Li, and W. Shan, "LLMACD: An LLM-based analog
circuit designer driven by behavior parameters," in Proc. Int. Symp.
Electron. Design Autom. (ISEDA), 2025, pp. 157–162.
[16]C. Liu, W. Chen, A. Peng, Y. Du, L. Du, and J. Yang, "AmpAgent: An
LLM-based multi-agent system for multi-stage amplifier schematic design
from literature for process and performance porting," arXiv preprint
arXiv:2409.14739, 2024.
Antonio J. Bujana (Graduate Student
Member, IEEE) received the B.S. degrees in
engineering (electrical and biomedical) and
the M.S. degree in engineering (electrical)
from LeTourneau University, Longview,
TX, USA, in 2015 and 2016, respectively.
He is currently pursuing the Ph.D. degree in
electrical engineering at Texas A&M
University, College Station, TX, USA.
He has over 10 years of industry
experience with Collins Aerospace. He is currently an Assistant
Professor of electrical engineering at Benedictine College, Atchison, KS,
USA. His research interests include large language model (LLM)-assisted
circuit design automation.
Aydin I. Karsilayan (Member, IEEE)
received the B.S. and M.S. degrees in
electrical engineering from Bilkent
University, Ankara, Turkey, in 1993 and
1995, respectively, and the Ph.D. degree
from Portland State University, Portland, OR,
USA, in 2000.
He joined Texas A&M University, College
Station, TX, USA, in 2000, where he is
currently an Associate Professor in electrical
and computer engineering. His current
research interests include high-frequency analog filters, data converters,
automatic tuning, mixed-mode integrated circuit design, RF
communication circuits, and power harvesting. He has served as an
Organizing Committee Member for the IEEE MWSCAS 2014. He has
served as an Associate Editor for the IEEE TRANSACTIONS ON CIRCUITS
AND SYSTEMS—I: Regular Papers, from 2002 to 2004, and the IEEE
Transactions on Circuits and Systems—II: Express Briefs, from 2006 to
2010.

