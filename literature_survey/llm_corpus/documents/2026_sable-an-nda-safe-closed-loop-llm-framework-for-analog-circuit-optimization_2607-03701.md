---
title: "SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows"
authors: "Li, Xunqi; Kim, Chris H."
date: "2026/07/04"
category: "sizing_optimization"
source_url: "https://arxiv.org/abs/2607.03701"
pdf: "papers/2026_sable-an-nda-safe-closed-loop-llm-framework-for-analog-circuit-optimization_2607-03701.pdf"
sha256: "b146a4a18b079700fa76164ab603ed6a8508166659202464ad53ca679e69f3ab"
pages_extracted: 21
---

# SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

SABLE: An NDA-Safe Closed-Loop LLM Framework for
Analog Circuit Optimization in Industrial EDA Flows
XUNQI LI, University of Minnesota, Twin Cities, US
CHRIS H. KIM, University of Minnesota, Twin Cities, US
Large language models (LLMs) can propose circuit-sizing decisions, but industrial analog design flows cannot
simply expose foundry process-design-kit (PDK) content, proprietary schematic details, absolute simulation
paths, or license-bound tool state to a cloud endpoint. This paper presents SABLE (Safe Analog Boundary
for LLM-driven EDA), an NDA-safe closed-loop framework that lets LLMs optimize analog circuits through
Cadence Virtuoso, Maestro, and Spectre while returning only scrubbed topology intent, numeric simulation
metrics, operating-point summaries, and scoped writeback status. We use “NDA-safe” to mean enforcement
under a stated curious-but-passive cloud-provider threat model, not a formal non-interference proof. The
framework combines an explicit threat model for the cloud-LLM boundary, a whitelist of 28 scoped SKILL
entry points, PDK/path/model scrubbing on every return path, structured Maestro setup and writeback recipes,
a strict JSON action contract with six machine-checked stop conditions, and best-so-far state preservation.
We evaluate eleven LLM checkpoints from the same documented reset state on two real closed-loop tasks,
both run as process–voltage–temperature (PVT) sign-offs across the same three corners: a 20 GHz LC-VCO
tuning-curve optimization task and a differential two-stage op-amp optimization task. On the LC-VCO task, 7
of 11 models pass; on the harder op-amp task—where every metric must hold at the worst of three corners
and a phase-margin gate rejects unstable high-gain points—4 of 11 models pass within the 15-iteration budget.
Single-checkpoint feedback-path ablations show that removing individual sanitized feedback channels either
silently weakens the specification or measurably degrades the search. The results show that model quality
differs sharply once the loop requires tool discipline, bias reasoning, and specification repair, and that an
NDA-safe EDA boundary can still provide enough sanitized feedback for successful analog circuit optimization.
Additional Key Words and Phrases: LLM agents, analog circuit optimization, EDA automation, Cadence
Virtuoso, Maestro, Spectre, NDA-safe design automation, LLM4EDA
1 Introduction
LLM-driven EDA loops are useful only when the tool boundary is as carefully engineered as
the prompt. Recent digital, HLS, and full-flow EDA agents use compiler, synthesis, place-and-
route, or simulation feedback to refine LLM-generated artifacts [4, 5, 13, 14, 19, 20]. Analog circuit
optimization is a natural next target because SPICE simulation provides a direct numerical oracle.
However, the analog case introduces a deployment constraint that is not a minor implementation
detail: a real foundry-bound Virtuoso flow contains PDK model cards, proprietary cell names,
private schematic hierarchy, absolute host paths, and license-bound simulation state that cannot be
sent to a cloud LLM.
Existing analog LLM work has made rapid progress on code generation, topology synthesis,
sizing, testbench construction, and benchmark design [1, 3, 6–10, 12, 16–18, 22–24]. The missing
Authors’ Contact Information: Xunqi Li, University of Minnesota, Twin Cities, Minneapolis, US, li001674@umn.edu; Chris
H. Kim, University of Minnesota, Twin Cities, Minneapolis, US, chriskim@umn.edu.
Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee
provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the
full citation on the first page. Copyrights for components of this work owned by others than the author(s) must be honored.
Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires
prior specific permission and/or a fee. Request permissions from permissions@acm.org.
© 2026 Copyright held by the owner/author(s). Publication rights licensed to ACM.
ACM 1557-7309/2026/7-ART
https://doi.org/10.1145/nnnnnnn.nnnnnnn
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.
arXiv:2607.03701v1
[cs.AR]
4
Jul
2026

## Page 2

2 Li and Kim
Fig. 1. Conceptual overview of SABLE. The LLM sees JSON actions and scrubbed observations. Foundry PDK
content, Maestro state, simulator raw paths, and proprietary model details remain inside the isolated EDA
host.
systems question is how to let a frontier model use the feedback it needs without crossing an
NDA boundary. Open-PDK benchmarks, public netlist datasets, domain-adapted chip models, and
on-premise model deployments are valuable, but they leave two practical questions unanswered
for industrial users: can cloud LLMs be connected to a real Virtuoso/Maestro loop without leaking
PDK content, and how should models be compared when every run must start from the same
documented EDA state?
This paper argues that the right unit of contribution is not another standalone prompt or a
claim that an LLM is a circuit designer. The unit is a reproducible, safety-bounded analog circuit-
optimization substrate. SABLE wraps the Cadence SKILL and OCEAN interface with a Python agent,
a return-path scrubber, per-project design-variable whitelists, structured Maestro setup recipes,
simulation-result readback, and a strict JSON action contract (Figure 1). The LLM can propose
numeric design variables and reason over sanitized topology and operating-point summaries, but it
cannot browse arbitrary libraries, read raw PDK model files, or receive unsanitized simulator paths.
Crucially, the boundary is not an informal convention: it is specified as a threat model with named
protected assets, an enumerated attack ladder, and five invariants that are enforced on every bridge
call and exercised by red-team unit tests.
We make five contributions:
(1) We introduce an NDA-safe closed-loop architecture for LLM-driven analog circuit optimiza-
tion in Virtuoso/Maestro flows, together with an explicit threat model: a curious-but-passive
cloud adversary, a four-tier attack ladder with the enforcement status of each tier, and five
hard invariants checked on every call (Sections 3 and 4).
(2) We define a machine-checkable interaction protocol: a strict JSON action contract with an
engineering-suffix value grammar, six enumerated stop conditions, scoped Maestro writeback
with landing verification, and best-so-far state preservation (Section 4).
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 3

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 3
(3) We define a fair benchmark protocol that resets each model to the same documented initial
design-variable state, runs the same Maestro tests, and logs the terminal cause of every run
(Section 5).
(4) We report an eleven-model matched comparison on two circuit classes, both run as process–
voltage–temperature (PVT) sign-offs with worst-corner gating across the same three corners—
an LC-VCO tuning-curve task and a two-stage op-amp optimization task—with full per-run
artifacts (Section 6).
(5) We provide initial feedback-path ablation evidence and a failure taxonomy with observed
terminal-cause counts, showing where closed-loop analog agents break: contract violations,
missing tool discipline, wrong bias-scale reasoning, insufficient use of operating-point evi-
dence, and endpoint or harness failures (Sections 7 and 8).
The rest of the paper is organized as follows. Section 2 positions the work. Section 3 defines
the threat model. Section 4 describes the closed-loop framework and the interaction protocol.
Section 5 defines the benchmark tasks and the fair-start protocol. Section 6 reports the eleven-
model comparison. Section 7 reports feedback-path ablation evidence. Section 8 analyzes failures.
Section 9 describes the reproducibility package and security audit, and Sections 10 and 11 conclude.
2 Related Work
2.1 LLMs in EDA
LLM4EDA systems have been most visible in digital RTL, HLS, and code-oriented flows, where
the feedback loop can be closed with compiler and synthesis diagnostics. AutoChip, HLSRewriter,
C2HLSC, LHS, and ConfiBench demonstrate this pattern [4, 5, 13, 14, 20], while ChatEDA broadens
the same agentic idea toward an RTL-to-GDS tool-execution flow [19]. Domain adaptation work
such as ChipNeMo also shows that chip-design-specific context can improve LLM usefulness across
assistant, script-generation, and debugging tasks [11]. These works motivate the central premise
of this paper: LLMs become more useful when the EDA tool is in the loop and the feedback is
machine-checkable. What the digital results do not transfer, however, is the confidentiality posture:
a Verilog compile log can usually be shared with a cloud endpoint, whereas a Spectre run on a
foundry PDK cannot.
2.2 LLMs for analog design
Analog LLM work has moved from netlist/code generation toward sizing and workflow automation.
AnalogCoder and AnalogCoder-Pro focus on analog circuit generation and optimization [7, 8].
ADO-LLM, EEsizer, AnaFlow, LEDRO, AutoSizer, and AnalogSAGE explore LLM-aided sizing,
multi-agent search, memory, and optimization strategies [1, 6, 10, 18, 22, 23]. CircuitSynth and
AnalogXpert study LLM-based topology synthesis, while Circuit-Agent and AnalogAgent represent
broader agentic analog-design workflows [2, 17, 21, 24]. AnalogGym, AICircuit, AMSNet, AMSnet-
KG, and Masala-CHAI expand the benchmark and dataset side of the field [3, 9, 12, 15, 16]. These
efforts establish that LLMs can participate in analog design loops. They largely operate on open
PDKs, exported netlists, or simulator wrappers that assume the model may see the design database.
Our contribution is complementary: we target the safety and reproducibility boundary required by
foundry-bound Virtuoso/Maestro deployment, where the schematic, the PDK, and the simulator
host must remain opaque to the model. In particular, AMSnet-KG [15], a concurrent effort, performs
knowledge-graph-retrieval netlist auto-design on a foundry (28 nm) PDK with Cadence Spectre and
Bayesian-optimization sizing—the closest prior setting to ours—yet it defines no model-provider
trust boundary, so it is orthogonal to the scrubbed cloud-LLM deployment substrate studied here.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 4

4 Li and Kim
Table 1. Where SABLE sits relative to representative analog-design systems. Real Cad.: drives a licensed
Cadence (Virtuoso/Maestro/Spectre) flow rather than open-source or wrapped simulators; Safe bnd.: states
and enforces a model-provider trust boundary with return-path scrubbing; Wrt.bk.: iteratively writes sized
variables back and re-simulates; OP rdbk.: exposes sanitized per-device operating-point evidence; Multi-mdl.:
matched benchmark across multiple LLMs. Entries reflect a direct reading of each cited paper.
PDK
Real
Cad.
Safe
bnd.
Wrt.
bk.
OP
rdbk.
Multi
mdl.
AnalogCoder [7] Open × × ✓ ✓ ✓
AnalogAgent [2] — × × ✓ ✓ ✓
AMSnet-KG [15] Priv. ✓ × ✓ × ×
AnalogGym† [9] Both × × ✓ × ×
SABLE (ours) Priv. ✓ ✓ ✓ ✓ ✓
†AnalogGym is a sizing-algorithm benchmark (BO/RL/EA), not an LLM agent; listed as the closest open evaluation suite. AMSnet-KG’s Real
Cad. is batch Spectre, without the Virtuoso/Maestro ADE flow that SABLE targets.
Table 1 positions SABLE against representative analog-design systems along the axes that matter
for a foundry-bound, cloud-LLM deployment. Several prior systems already iterate over simulation
feedback, expose operating-point hints, or compare multiple models; one even runs a real foundry
PDK through Cadence Spectre. What none of them do is place that licensed Cadence flow behind an
enforced model-provider trust boundary. That boundary—not another optimizer—is our contribution.
2.3 NDA-constrained deployment
Two broad strategies can address NDA-constrained analog LLM use. One is to move more model
capability inside the trusted environment through domain-adapted, local, or self-improving agents
[2, 11]. The other is to keep cloud models outside the trusted environment and strictly control
what crosses the boundary. SABLE takes the second path. It does not claim to remove the need
for local EDA licenses or foundry access; instead, it defines a scrubbed bridge that lets external
models see enough evidence to optimize without seeing the confidential artifacts that produced the
evidence. To our knowledge, no prior analog LLM sizing work states an explicit threat model for
the model-provider boundary or audits its feedback path against one.
3 Threat Model and Safety Boundary
3.1 Adversary and protected assets
The framework assumes that the cloud LLM provider is curious-but-passive: it can observe every
prompt and returned tool result and may log, train on, or later mine that traffic offline, and the tool
arguments it returns may be adversarial; but it does not compromise the local operating system or
the Cadence process, and it does not mount active man-in-the-middle attacks on the EDA host.
“Passive” here refers to the absence of in-band host compromise, not to benign intent—adversarial
tool arguments are squarely in scope and are handled by the Tier-2 defenses below. Under this
model, the protected assets are:
• foundry device-model content (model-card parameter names and values, e.g., BSIM-class
threshold, mobility, and capacitance parameters);
• process back-end information (layer stack, pitches, parasitics);
• proprietary cell and library identifiers beyond the allowed project scope, and schematic
hierarchy that reveals them;
• host-environment identifiers: user names, host names, absolute paths, license servers, and
API keys; and
• raw simulator state and raw waveform payloads that could embed any of the above.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 5

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 5
Table 2. Attack ladder for the cloud-LLM boundary and current enforcement status. “Enforced” means blocked
by construction and covered by red-team unit tests; “out of scope” and “partial” tiers are stated limits of a
client-side boundary and are not claimed to be red-team covered.
Tier Attack class Status
1 Direct prompt-level requests for PDK/model content enforced
2 Injection via tool arguments (SKILL/identifier breakout) enforced
3a Reasoning-trace replay into later prompts enforced
3b Provider-side retention of first-emission traces out of scope
4 Timing and error-class side channels partial
The allowed feedback is intentionally narrower than what an interactive designer sees: scalar
metrics, bounded waveform summaries, anonymized topology intent, internal node voltages under
safe aliases, and selected device operating-point scalars drawn from a 20-key whitelist (Section 4.5).
3.2 Attack ladder and enforcement status
Rather than claiming blanket safety, the boundary is audited against an explicit ladder of attack
classes, summarized in Table 2. Tiers 1 and 2 cover direct requests for protected content and
injection through tool-call arguments (for example, SKILL expressions smuggled into identifier
fields); both are blocked by construction through the whitelist and grammar checks of Section 4.2
and are exercised by red-team unit tests. Tier 3 covers reasoning-trace channels: content that
enters a model’s reasoning trace and could be re-emitted later. Re-entry of prior reasoning text into
subsequent prompts is scrubbed at every LLM-client sink; what the provider retains server-side is
out of scope for a client-side framework and is mitigated only by provider policy or on-premise
inference. Tier 4 covers side channels such as timing and error-class differences, which are at best
partially mitigated. We state these limits explicitly because an NDA-safe claim is credible only
when its non-goals are named.
3.3 Hard invariants
Five invariants are checked on every bridge call, not at construction time:
(1) every string returned toward the LLM passes the PDK-token, path, and identifier scrubber;
(2) design-variable names proposed by the LLM must match the per-project whitelist declared
in the task specification;
(3) only whitelisted SKILL entry points may be invoked—28 at submission time: 25 scoped safe*
wrappers plus three backward-compatibility aliases—and nested calls inside arguments are
rejected except for a single data-constructor allow-list (list);
(4) file uploads toward the EDA host are confined to a dedicated skill directory; and
(5) operating-point readback returns only keys from a 20-field whitelist (bias voltages, currents,
small-signal parameters, threshold/overdrive, region, and terminal capacitances), never model-
card parameters.
These invariants are not all of one kind, and the distinction matters for what we claim. Invariants 2,
3, and 5 are structural allowlists: they bound what the boundary can emit—which design variables,
which SKILL entry points, which operating-point keys—by construction, so content outside the
list is unreachable rather than merely filtered. Invariant 1, the return-path scrubber, is different in
kind: it is a conservative, deployment-specific pattern filter that strips foundry tokens and absolute
paths from free-text fields (diagnostics, exception messages, reasoning echoes) on a best-effort
basis. An identifier that matches no seeded pattern can pass, so the scrubber is defense-in-depth on
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 6

6 Li and Kim
Fig. 2. SABLE data flow. The cloud LLM receives JSON actions and scrubbed observations only. PDK content
is referenced inside Cadence and Spectre but is not serialized into model-visible feedback.
the residual free-text surface, and the structural allowlists—not the scrubber—carry the primary
guarantee. We therefore make the pattern list part of the deployment: a foundry extends the seeded
families with its own naming scheme before use.
One structural channel also warrants an explicit caveat. The operating-point whitelist returns
instance operating points—per-device bias, transconductance, and terminal capacitance—never
model-card parameter names or values. A single reading is an instance quantity, not a model
coefficient; but a provider that aggregates these scalars across the benchmark’s bias points and
PVT corners could in principle regress device behaviour back toward model-card quantities (a
threshold trend, an overdrive-to-current slope). The small per-run point count bounds but does not
eliminate this inference channel; we treat it as out of scope for a client-side boundary (Section 10),
and stronger parameter confidentiality is a reason to prefer on-premise inference.
Together these invariants define what “NDA-safe” means in this paper: a practical, audited
scrubbing boundary under a stated adversary model, not a formal proof of non-interference.
4 The SABLE Framework
4.1 Closed-loop flow
SABLE is organized around a one-time setup pass followed by an iterative optimization loop. In the
setup pass, the agent first reads the target Virtuoso schematic through SafeBridge, which returns
an anonymized topology summary (Section 4.5)—device roles and connectivity with foundry cell
names replaced by neutral aliases—rather than the design database itself. This scrubbed topology is
what lets the LLM understand the circuit under test and ground the target specification, while the
confidential schematic never leaves the EDA host. Only then does the agent engage Maestro and
enter the loop. Each turn, the LLM proposes a JSON action containing design-variable updates; the
Python agent validates the action, writes allowed variables into Maestro, triggers simulation, reads
back numeric results, computes pass/fail status against the specification, and returns a scrubbed
feedback packet. The loop stops when all hard-pass specifications are met or when one of the stop
conditions of Section 4.3 fires. If the iteration budget is exhausted, the agent writes back the best
numeric point observed rather than blindly leaving the final point in Maestro. Figure 2 shows the
data flow and the trust boundary.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 7

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 7
4.2 SafeBridge enforcement layers
The bridge enforces the boundary of Section 3 in four code locations; together with the scoped-
readback filters of Section 4.5 they implement the five invariants of Section 3. First, project specifi-
cations define the design variables that the LLM may edit; variable names are validated against
this whitelist and against a conservative identifier grammar (letters, digits, underscore, bounded
length), and values must be single signed numeric atoms with engineering suffixes (f/p/n/u/m/k);
strings carrying physical units or expressions are rejected. Second, Python validates analysis names,
output expressions, and SKILL entry points before a command reaches Virtuoso: only 28 scoped
SKILL wrappers may be named (25 safe* functions plus three backward-compatibility aliases),
and the full argument expression is scanned for nested calls, which are rejected unless they belong
to a small data-constructor allow-list. Third, the SKILL wrappers on the Cadence side expose only
scoped read and write APIs (schematic readback, operating-point readback, parameter set, OCEAN
run/measure, Maestro setup and save), rather than arbitrary database access. Fourth, every return
path is scrubbed for absolute paths, foundry-like tokens, model identifiers, and unsafe strings before
the text is reinserted into the LLM conversation; the same scrubber is applied at every LLM-client
sink, including replayed reasoning content.
4.3 JSON action contract and stop conditions
Free-form model output is not accepted into the loop. Each turn, the model must return a single
JSON object of the following shape:
{
"iteration": 3,
"measurements": { "<metric_name>": <number> },
"pass_fail": { "<metric_name>":
"PASS | FAIL (...) | UNMEASURABLE (...)" },
"reasoning": "<one paragraph diagnosing the
pass/fail pattern>",
"design_vars": { "<var_name>":
"<value_with_engineering_suffix>" }
}
The design_vars keys must match the specification whitelist, and the values must use the
engineering-suffix grammar (for example 500u or 1.5f); unit-bearing strings such as mA or pF are
rejected. A malformed action triggers one structured repair prompt; a second violation terminates
the run with a logged contract violation rather than silently guessing the model’s intent. The agent
recomputes measurements and pass_fail authoritatively on the trusted side, so a model cannot
talk its way into a pass.
The loop has six machine-checked stop conditions: (i) success, all hard-pass metrics pass; (ii)
max_iter, the iteration budget is exhausted; (iii) safeguard, an oscillator-health guard fires (for
the LC-VCO task, an amplitude-hold ratio below 0.3 for three consecutive iterations); (iv) stuck,
identical design vectors with failing metrics for consecutive iterations; (v) contract_violation,
as above; and (vi) no_changes, a first iteration that produces neither a baseline simulation nor
a proposal. Logging the terminal cause of every run is what later makes the failure taxonomy of
Section 8 quantitative rather than anecdotal.
4.4 Maestro setup and scoped writeback
SABLE treats Maestro setup as part of the reproducible experiment, not as manual GUI state. A
setup recipe can enable analyses, create output metrics, define specs, set design variables, and verify
that the requested state actually landed under the target test row. This was necessary in practice
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 8

8 Li and Kim
because AC and DC outputs need different expression families, and because model comparison
is only meaningful when every run starts from the same documented state. The writeback path
records whether test-scoped variables were actually written; this matters for analog tasks because
a visually open Maestro session can still fail to accept writes if the wrong setup database handle is
used. At the end of a run, the best-so-far design point (by the evaluator’s score) is written back
and saved, so a failed search still leaves the database in the best observed state, and the writeback
status is part of the per-run record.
4.5 Sanitized readback
Three readback channels give the model evidence without exposing the design database. Topology
intent converts the schematic (including a hierarchical, same-library-only deep read) into an
anonymized device-role summary: input pair, tail source, mirrors, loads, common-mode feedback,
and output stage, with foundry cell names replaced by neutral aliases. Metric readback extracts
AC/DC/transient and swept metrics from Maestro/Spectre results, including multi-point sweeps
for tuning curves and per-corner scalar reads for PVT (process–voltage–temperature) sign-off. For
a PVT task the agent re-simulates the design at each named corner and returns one scalar set per
corner; the corner’s supply voltage is injected through the same trusted corner-control path used
for process and temperature, never through the LLM action space, so the model cannot edit a supply
rail even while the loop sweeps voltage. The evaluator then gates each metric on the worst corner
(worst-corner-must-pass). Operating-point readback returns a per-device DC table restricted to a
20-key whitelist—bias voltages (vgs, vds, vbs), currents (id, ids), small-signal parameters (gm, gds,
gmb), threshold and overdrive (vth, vdsat, vov), an operating-region label, terminal capacitances
(cgs, cgd, cdb, cdg, cgg), and per-element i/v/pwr for passive elements—so that the model can
reason about bias and compensation without ever seeing a model card.
Beyond these three channels, the framework exposes an optional, circuit-independent hook for a
deterministic, task-specific feedback module. When a task registers one, the framework appends a
bounded, ranked list of directional design-variable suggestions that a pure-Python routine computes
from data the agent already holds. Any such module intersects its proposals with the live design-
variable whitelist and asserts the absence of foundry tokens in its output, so it stays within the
same NDA boundary as the readback channels; it is a feedback channel, not an optimizer that
bypasses the LLM. A task that registers no module simply runs with the three channels above, and
the single instance used in this paper is described with its task in Section 5.
5 Benchmark Protocol
5.1 Tasks
The benchmark uses two closed-loop tasks on a proprietary 16 nm FinFET PDK, chosen to stress
different parts of analog circuit optimization. Because the process is FinFET, transistor sizing is set
by the number of fins rather than a width/length ratio, so the device-sizing variables in both tasks
are fin counts (𝑁•) rather than 𝑊 /𝐿 values.
LC-VCO tuning curve (PVT). The LC-VCO task uses a 20 GHz nominal operating point with
the control voltage fixed at 0.4 V (𝑉𝐷𝐷/2 at the nominal supply) for the single-point metrics and
swept over [0.1, 0.7] V (7 points) for the tuning curve. Like the op-amp task, it is run as a PVT
(process–voltage–temperature) sign-off: every iteration simulates the oscillator at three corners
and gates the single-point metrics on the worst corner (worst-corner-must-pass). The three corners
match the op-amp set—the typical center (tt, 25 ◦C, 0.80 V), a slow-hot corner (ss, 125 ◦C, 0.72 V),
and a fast-cold corner (ff, −40 ◦C, 0.88 V)—so that the two PVT tasks share one sign-off envelope.
The pass condition combines worst-corner oscillation frequency, differential swing, common-mode
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 9

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 9
Fig. 3. LC-VCO topology: a cross-coupled NMOS pair (𝑁xc) resonating with the LC tank (𝐿tank, 𝐶tank) and a
MOS varactor (𝑁var) tuned by 𝑉𝑐𝑡𝑟𝑙 , biased by a mirror tail current source (𝐼bias, 𝑁tail). Labels match Table 4.
Table 3. LC-VCO PVT machine-readable pass metrics. The seven single-point metrics are read back per corner
and gated on the worst of the three sign-off corners shared with the op-amp task; a run passes only if every
band holds at every corner. The four tuning metrics are evaluated over the 7-point 𝑉𝑐𝑡𝑟𝑙 sweep at the nominal
corner once the single-point worst-corner gate passes. The 𝑉𝑐𝑚 band is widened from the single-corner
[0.70, 0.81] to cover the 𝑉𝐷𝐷 rail at every corner, and the 𝐾𝑉𝐶𝑂 ceiling and linearity bound are relaxed for the
ideal-𝐿𝐶 concept as noted in the released spec.
Metric Pass band Unit
f_osc_GHz [19.5, 20.5] GHz
V_diff_pp_V ≥ 𝑉𝐷𝐷 /2 V
V_cm_V [0.65, 0.92] V (tracks 𝑉𝐷𝐷 )
duty_cycle_pct [48, 52] %
amp_hold_ratio ≥ 0.95 late/early RMS
t_startup_ns ≤ 10 ns
I_core_uA ≤ 800 𝜇A
tuning_range_GHz ≥ 0.8 GHz
Kvco_MHz_per_V [150, 3000] per segment MHz/V
Kvco_linearity ≤ 4.0 max/min ratio
monotonic true –
level, duty cycle, startup behavior, amplitude hold, and core current, plus four tuning-curve metrics
evaluated at the nominal corner after the single-point worst-corner gate passes (Table 3). The LLM
may edit eight design variables (Table 4; schematic in Figure 3); the supply 𝑉𝐷𝐷 is corner-driven
through the same trusted corner-injection path as the op-amp task and is not an LLM knob, and the
control voltage is not an optimization knob. This task stresses oscillator transient startup across
corners, swept metrics, and the need to keep design variables physically plausible. For this swept-
metric task we enable the optional deterministic feedback channel of Section 4.5: a curve-searcher
module that, after each tuning-curve sweep, ranks a bounded list of directional design-variable
suggestions targeting the worst-violating tuning metric. It is the only task-specific feedback module
in the benchmark; the op-amp task uses none.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 10

10 Li and Kim
Table 4. LC-VCO design variables exposed to the LLM, with their declared search ranges. Symbol names are
used here; the exact design_vars keys the LLM edits are listed in the released spec.
Variable Role Range
𝐼bias mirror reference current 100 𝜇A–2 mA
𝑁xc cross-coupled NMOS pair (fins) 4–32
𝑁var MOS varactor (fins) 4–40
𝑁mir diode-mirror device (fins) 4–32
𝑁tail tail device (fins) 8–32
𝑅bias bias isolation resistor 1 k–100 kΩ
𝐶tank tank fixed capacitor 10 f–200 fF
𝐿tank tank differential inductor 100 p–2 nH
Table 5. Op-amp PVT machine-readable pass metrics, evaluated on the differential AC testbench at 𝑉𝑖𝑐𝑚 =
𝑉𝐷𝐷 /2. Each metric is read back per corner and gated on the worst of the three sign-off corners; a run passes
only if every hard-pass metric holds at every corner. The phase-margin row was added for the PVT task to
gate stability; the common-mode and CMRR rows are diagnostic outputs configured in the Maestro setup.
Metric Worst-corner band Role
A0_diff_db ≥ 45 hard pass (diff. gain, dB)
UGB_Hz ≥ 150 MHz hard pass (bandwidth)
PM_deg ≥ 45 hard pass (phase margin, deg)
Vout_cm [0.35, 0.45] V DC output common mode
gain_cm_db minimize feeds CMRR
CMRR_db ≥ 40 𝐴0_𝑑𝑖𝑓 𝑓 _𝑑𝑏 − 𝑔𝑎𝑖𝑛_𝑐𝑚_𝑑𝑏 (dB)
Table 6. Op-amp design variables the LLM may edit (19 sizing knobs in four groups). Symbol names are used
here; the exact design_vars keys are listed in the released spec. The op-amp spec declares the whitelist and
the all-ones reset but not per-variable ranges; the model explores within the SafeBridge allowed_params
whitelist and physical plausibility. The four stimulus variables (input common mode, AC magnitude, two AC
phases) are held fixed and excluded.
Group (count) Variables (𝑁: fin counts, 𝐼: currents, 𝑅/𝐶)
Differential-stage sizing (7) 𝑁n1, 𝑁n2, 𝑁p1, 𝑁p2, 𝑁nb, 𝑁pb, 𝑁tail
Bias currents (2) 𝐼n, 𝐼p
CMFB (7) 𝑁cn1, 𝑁cn2, 𝑁cp1, 𝑁cp2, 𝑁cb, 𝑁cc, 𝐼c
Miller & CM sense (3) 𝑅m,𝐶m, 𝑅cm
Two-stage op-amp (PVT). The op-amp task uses a differential AC testbench with common-
mode input stimulus, output load capacitance, and a fully differential two-stage op-amp with
resistor-sensed common-mode feedback and Miller RC compensation (Figure 4). It is run as a
PVT (process–voltage–temperature) sign-off task: every iteration simulates the design at three
corners and gates each AC metric on the worst corner (worst-corner-must-pass). The three corners
are the typical center (tt, 25 ◦C, 0.80 V), a slow-hot corner (ss, 125 ◦C, 0.72 V), and a fast-cold
corner (ff, −40 ◦C, 0.88 V), spanning all three process labels, both temperature extremes, and the
±10% supply range. The two most extreme corners of a full five-corner set (slow/cold/low-supply
and fast/hot/high-supply) are excluded from sign-off based on exploratory five-corner runs: no
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 11

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 11
Fig. 4. Two-stage Miller-compensated op-amp (single-ended core shown): an NMOS input pair (𝑁n1) with a
PMOS mirror load, a common-source second stage (𝑁p1), and Miller compensation (𝑅m, 𝐶m). The benchmark
circuit is the fully differential version of this core, with a resistor-sensed common-mode feedback (𝑅cm) loop
setting the output common mode. Labels match the groups of Table 6.
sizing point within the declared variable ranges held gain at the slow/cold/0.72 V corner, where
the push-pull second stage falls out of its operating region, so retaining those corners would make
the task unsatisfiable by construction rather than discriminating. The three retained corners form
the feasible PVT envelope. The hard-pass targets are a worst-corner differential gain 𝐴0 ≥ 45 dB,
a worst-corner unity-gain bandwidth ≥ 150 MHz, and—added for the PVT task—a worst-corner
phase margin ≥ 45◦ that gates stability so a high-gain but under-compensated point cannot pass.
The worst-corner gain bound is 45 dB rather than the 50 dB single-corner target because the worst
of three corners naturally sits a few dB below the typical corner. The LLM may edit nineteen
sizing variables (stage and CMFB device sizes, bias currents, Miller 𝑅 and 𝐶, and the common-mode
sense resistor); the supply 𝑉𝐷𝐷 is not LLM-editable—it is driven per corner through a trusted
corner-injection path that keeps supply rails out of the LLM action space while still sweeping
voltage. The full per-metric bands are listed in Table 5. The four stimulus variables (input common
mode, AC magnitude, and the two AC phases) are held fixed by the harness so that a model cannot
pass by editing the testbench instead of the circuit.
5.2 Fair-start protocol
Each model is evaluated from the same reset state. For the op-amp, all non-stimulus sizing variables
are initialized to one—a deliberately unbiased, nearly pathological start—while stimulus variables
use the differential AC preset (common-mode input 𝑉𝐷𝐷/2, antiphase AC drive with 0/180 degree
phases). For the LC-VCO, all sizing variables likewise start from the documented all-ones reset, the
supply is driven per corner through the trusted corner path, and the control voltage is fixed by
the nominal and sweep definitions. The reset is applied programmatically through the Maestro
writeback path and verified before the first iteration; each per-run record stores the reset report.
The iteration budget is 15 unless otherwise specified. A run is counted as Pass only if all hard-pass
metrics are satisfied before the budget ends. Every run logs its terminal cause (success, metric miss
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 12

12 Li and Kim
at budget, contract violation, LLM/client error, or harness failure), wall-clock time, iteration count,
final and best metrics, final design vector, and writeback status, as CSV/JSONL artifacts.
The main tables report eleven matched model checkpoints spanning seven vendor families. The
same model set is used for the op-amp and LC-VCO tasks so that cross-task comparisons are not
confounded by different endpoint coverage. Each matched cell is a single audited run (seed 1); we
discuss the implications of single-seed snapshots in Section 10.
6 Results
6.1 Overall outcome
Both tasks are run under the same PVT (process–voltage–temperature) sign-off discipline, with
worst-corner gating across the same three corners, so their pass rates reflect the circuits’ intrinsic
closure difficulty rather than any asymmetry in the test regime. On the LC-VCO task, 7 of 11
models pass, with passing runs requiring four to fifteen iterations (Table 7). The op-amp task
is the harder of the two: only 4 of 11 models pass (Table 8). The failure signatures explain the
gap. LC-VCO failures are one-dimensional—the worst-corner oscillation frequency falls outside
the [19.5, 20.5] GHz band (an under-frequency miss for claude-sonnet-4-6, over-frequency for
gpt-5.4-mini, or no sustained oscillation at all for claude-haiku-4-5)—so a failing model misses on
frequency placement alone. The fourth LC-VCO failure is not a circuit miss at all: minimax-m2.7
terminated through an unrecoverable client error at iteration 7 (Section 8), so its row measures the
endpoint stack rather than the design. The op-amp failures are multi-dimensional: several models
never reach the worst-corner gain target (𝐴0 <45 dB at that corner) or collapse the design entirely,
while two models (claude-opus-4-8 and gpt-5.4-mini) clear the worst-corner gain and bandwidth
gates but with a negative worst-corner phase margin—a high-gain operating point that would
oscillate, which the phase-margin gate rejects as unstable. Holding gain, bandwidth, and stability
simultaneously across three corners is a substantially tighter closure condition than aligning a
single oscillation frequency, and it exposes that the loop can optimize a visible metric while missing
the coupled biasing and compensation tradeoffs required for full specification closure.
6.2 Tuning-curve quality of passing runs
Pass/fail alone hides how differently the models size the same oscillator. Figure 5 plots the per-
segment 𝐾𝑉𝐶𝑂 of each passing model’s final design point across the 𝑉𝑐𝑡𝑟𝑙 sweep at the nominal
corner, against the [150, 3000] MHz/V per-segment pass band. The seven passing models land
tuning curves of similar shape—monotonically decreasing gain toward the top of the varactor
range—but with visibly different flatness, which the Kvco_linearity column of Table 7 quantifies
(1.93 for minimax-m3 and 1.95 for kimi-k2.5 at the flat end versus 3.44 for deepseek-v4-flash at the
steep end). The four failing runs never reach this plot: because the tuning sweep runs only after the
single-point worst-corner gate passes, and all four miss the worst-corner 𝑓𝑜𝑠𝑐 band (or, for minimax-
m2.7, terminate on a client error first), none produced a tuning curve. The LC-VCO difficulty
under PVT is therefore concentrated in placing the worst-corner oscillation frequency inside the
band: every model that clears that gate also lands a tuning curve inside the task’s per-segment
bounds. That second observation should be read against the calibrated bands of Table 3—under
the base variant’s tighter 2000 MHz/V ceiling, three of the seven passing curves (minimax-m3,
deepseek-v4-flash, gemini-2.5-pro) would have exceeded the per-segment limit—so the relaxed
ceiling shifts the discriminating gate to frequency placement rather than eliminating tuning-curve
difficulty outright.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 13

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 13
Table 7. Eleven-model LC-VCO PVT benchmark. Each single-point metric is gated on the worst of the same
three sign-off corners as the op-amp task (tt/25 ◦C/0.80 V, ss/125 ◦C/0.72 V, ff/−40 ◦C/0.88 V); the 𝑓𝑜𝑠𝑐, 𝑉𝑝𝑝,
and 𝐼𝑐𝑜𝑟𝑒 columns report the worst-corner value, and the tuning-range and 𝐾𝑉𝐶𝑂 -linearity columns are read
at the nominal corner after the single-point gate passes (blank for runs that never reached the sweep phase).
All runs start from the same documented reset state and use a 15-iteration budget. The model set matches
the op-amp benchmark exactly.
Model Outcome Iter. Time (s) 𝑓𝑜𝑠𝑐 (GHz) 𝑉𝑝𝑝 (V) 𝐼𝑐𝑜𝑟𝑒 (𝜇A) Range (GHz) 𝐾𝑉𝐶𝑂 lin. Failure reason
claude-opus-4-8 Pass 5 2493 20.06 3.28 359.9 0.824 2.06 –
claude-sonnet-4-6 Fail 15 4093 19.33 3.14 473.0 – – max_iter (𝑓𝑜𝑠𝑐 <19.5)
claude-haiku-4-5 Fail 15 3709 – – – – – max_iter (no osc.)
gpt-5.5 Pass 6 3404 19.94 3.28 359.6 0.833 1.99 –
gpt-5.4-mini Fail 15 3912 20.52 3.09 415.6 – – max_iter (𝑓𝑜𝑠𝑐 >20.5)
kimi-k2.5 Pass 4 2165 20.00 3.26 365.3 1.005 1.95 –
minimax-m2.7 Fail 7 2523 20.86 3.18 372.4 – – llm_error
minimax-m3 Pass 6 3866 19.93 3.26 361.4 1.076 1.93 –
deepseek-v4-pro Pass 8 3483 20.15 3.28 361.7 0.845 2.11 –
deepseek-v4-flash Pass 6 2344 20.00 3.19 365.6 1.331 3.44 –
gemini-2.5-pro Pass 15 5912 20.07 3.26 365.8 1.165 2.85 –
The three LC-VCO metric-miss failures all miss the same hard metric—worst-corner 𝑓𝑜𝑠𝑐 outside [19.5, 20.5] GHz—and
terminate at the iteration budget: claude-sonnet-4-6 stalls under-frequency (19.33 GHz), gpt-5.4-mini over-frequency
(20.52 GHz), and claude-haiku-4-5 never sustains oscillation at the worst corner (no measurable 𝑓𝑜𝑠𝑐 ). minimax-m2.7
terminated early through an llm_error—an unrecoverable client error at iteration 7, not a metric miss. Because the
single-point worst-corner gate precedes the tuning sweep, none of the four failing runs reached the sweep phase, so their
tuning-range and 𝐾𝑉𝐶𝑂 columns are blank.
Table 8. Op-amp PVT benchmark. Each metric is gated on the worst of three sign-off corners—typical
(tt, 25 ◦C, 0.80 V), slow-hot (ss, 125 ◦C, 0.72 V), and fast-cold (ff, −40 ◦C, 0.88 V); the 𝐴0, UGB, and phase-
margin (PM) columns report the worst-corner value. A run is Pass only if all three hard gates (𝐴0 ≥45 dB,
UGB≥150 MHz, PM≥45◦) hold at every corner. All runs use the 15-iteration budget and the same model set
as the LC-VCO benchmark.
Model Outcome Iter. Time (s) 𝐴0 (dB) UGB (MHz) PM (deg) Failure reason
claude-opus-4-8 Fail 15 1663 51.76 695.0 −6.78 PM< 45 (unstable)‡
claude-sonnet-4-6 Pass 9 1144 50.35 257.0 150.33 –
claude-haiku-4-5 Fail 15 1785 39.88 1490.9 28.47 max_iter (𝐴0, PM)
gpt-5.5 Pass 3 397 52.53 212.9 101.51 –
gpt-5.4-mini Fail 15 1467 41.05 396.5 −36.12 PM< 45 (unstable)‡
kimi-k2.5 Fail 15 2645 −13.32 114.6 – max_iter
minimax-m2.7 Pass 9 1305 49.50 313.0 108.39 –
minimax-m3 Fail 2 1810 −9.45 2496.3 −85.96 contract_violation
deepseek-v4-pro Pass 9 1942 50.46 848.2 89.55 –
deepseek-v4-flash Fail 15 2194 37.10 710.9 137.11 max_iter (𝐴0)
gemini-2.5-pro Fail 15 1807 −74.82 – 70.13 max_iter
‡The design clears the worst-corner 𝐴0 and UGB gates but its worst-corner phase margin is negative—a high-gain
operating point that would oscillate. The phase-margin gate added for the PVT task rejects it as an unstable
(non-converged) design rather than a pass; without it, 𝐴0/UGB alone would have falsely counted these two runs as passes.
6.3 Cross-task observations
First, passing one topology does not imply passing the other, even under the same PVT gate. Several
strong checkpoints pass both tasks, but the mismatches remain informative and now run in both
directions. deepseek-v4-flash passes LC-VCO while failing the op-amp PVT loop with a worst-
corner gain miss at the iteration budget; conversely, claude-sonnet-4-6 and minimax-m2.7 pass the
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 14

14 Li and Kim
0.1 0.15 0.2 0.25 0.3 0.35 0.4 0.45 0.5 0.55 0.6 0.65 0.7
0
1,000
2,000
3,000
𝑉𝑐𝑡𝑟𝑙 segment midpoint (V)
𝐾𝑉
𝐶𝑂
(MHz/V)
pass-band ceiling (3000) pass-band floor (150) claude-opus-4-8
gpt-5.5 kimi-k2.5 minimax-m3
deepseek-v4-pro deepseek-v4-flash gemini-2.5-pro
Fig. 5. Per-segment 𝐾𝑉𝐶𝑂 of each passing model’s final LC-VCO design point over the 𝑉𝑐𝑡𝑟𝑙 sweep at the
nominal corner, from the benchmark artifacts. All seven passing models stay inside the [150, 3000] MHz/V
per-segment band (dashed/dotted red). The four failing runs are absent because they never cleared the
single-point worst-corner 𝑓𝑜𝑠𝑐 gate that precedes the tuning sweep.
op-amp PVT loop while missing LC-VCO—sonnet stalling under-frequency and minimax-m2.7 lost
to a client error. gpt-5.4-mini misses both tasks with different signatures: an unstable op-amp point
(negative worst-corner phase margin) and an over-frequency LC-VCO point at the budget. Claude-
haiku-4-5 also misses both, its LC-VCO row a worst-corner metric miss (no sustained oscillation)
rather than a tool failure. This bidirectional split supports treating analog agent evaluation as a
multi-topology benchmark rather than a single-circuit leaderboard.
Second, the op-amp failures are informative rather than random. Several failed runs produce very
high worst-corner UGB values but insufficient or negative low-frequency gain, or—the failure mode
the PVT phase-margin gate newly exposes—high gain and bandwidth but a negative worst-corner
phase margin. Both are recognizable analog-design failure modes: the model has found a fast or
misbiased point without the operating-region and compensation balance needed for gain, or it has
traded compensation away for gain and produced an unstable amplifier. The framework changes
these from an opaque model failure into a debuggable trace because the LLM-visible feedback
contains per-corner AC metrics (now including phase margin), DC readback, topology intent, and
operating-point scalars.
Third, tool-protocol reliability is part of model quality in an EDA agent. On the op-amp PVT
task, minimax-m3 terminated through a contract_violation after two iterations: it repeatedly
returned JSON that omitted required action keys, and the harness repair policy could not recover a
well-formed action. This is not an analog metric miss but a tool-contract failure, and it exposes an
important deployment risk: a model that can reason about circuits but cannot reliably emit the
contracted action loop is not deployable as a closed-loop EDA agent. A deployable agent must
therefore be evaluated at the level of the complete tool loop, including the provider endpoint,
response contract, parser, and repair policy; rankings must be read as dated snapshots of the served
model plus its client stack.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 15

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 15
Table 9. Single-corner base LC-VCO feedback-path ablations for mimo-v2.5-pro (single seed, 5-iteration evi-
dence budget; the 𝐾𝑉𝐶𝑂 ceiling here is the base variant’s 2000 MHz/V, not the PVT benchmark’s 3000 MHz/V).
“Nominal-only Pass” means all evaluated metrics passed but the tuning-curve metrics were never evaluated
because the swept-metric path was disabled.
Variant Outcome Iter. Note
full Pass 3 range 0.887 GHz, 𝐾𝑉𝐶𝑂 lin. 1.99
no swept metrics nominal-only Pass 2 tuning spec silently unevaluated
no curve searcher Fail 5 𝐾𝑉𝐶𝑂 segments above 2000 MHz/V
no writeback Pass 4 final state not persisted to Maestro
Fourth, iteration efficiency varies substantially among passing models. On LC-VCO, kimi-k2.5
passes in four iterations (2165 s wall clock), while gemini-2.5-pro needs the full fifteen (5912 s).
The snapshot also shows that a pass can arrive after the same iteration count at very different
wall-clock cost: gpt-5.5, minimax-m3, and deepseek-v4-flash all pass in six iterations, but at 3404 s,
3866 s, and 2344 s respectively. On the op-amp PVT task, gpt-5.5 passes in three iterations (397 s),
whereas claude-sonnet-4-6, minimax-m2.7, and deepseek-v4-pro each pass at nine iterations but at
1144 s, 1305 s, and 1942 s respectively—the same iteration count over three corners at very different
wall-clock cost. Since every iteration now costs a per-corner Spectre run at three corners plus
model tokens on both tasks, iteration count is a first-order proxy for deployment cost, and the
per-run wall-clock and iteration columns in Tables 7 and 8 are part of the published artifacts.
7 Feedback-Path Case Ablations
Each sanitized feedback channel is both optimization evidence and audit surface, so a natural
question is which channels actually change outcomes. As preliminary, design-motivating evidence—
a single mid-tier checkpoint (mimo-v2.5-pro), single seed, 5-iteration budget, from the same all-ones
reset on the single-corner base LC-VCO variant of the task (the feedback-channel mechanism under
test is identical to the PVT benchmark; only the corner set and the 𝐾𝑉𝐶𝑂 ceiling differ)—we disabled
feedback paths one at a time (Table 9). We report these because each directly justifies a framework
design decision; they are case observations, not statistical claims.
The first decision-relevant result is that disabling the swept-metric path does not degrade
gracefully: the run still reports Pass in fewer iterations, because the tuning-curve portion of the
specification silently drops out of the evaluation. In a safety-bounded loop a feedback ablation can
thus masquerade as an improvement—which is exactly why the framework records the evaluated
metric set, not just the pass verdict, as part of every run record. The second is that removing the
deterministic curve-searcher turns the same task into a failure within the budget: the model widens
the tuning range but leaves per-segment 𝐾𝑉𝐶𝑂 above the base variant’s 2000 MHz/V ceiling—direct
evidence that structured, PDK-safe directional hints are cheap to sanitize and measurably help hold
the tuning-curve specification. (Disabling best-so-far writeback did not change this run’s verdict,
consistent with its role as state preservation rather than search assistance; its value shows up in
failed runs, where the database would otherwise be left at an arbitrary point.)
A companion full-feedback sweep over seven other checkpoints at the same 5-iteration budget
broadly tracks the main table but is budget-sensitive: deepseek-v4-flash fails here yet passes under
the 15-iteration matched budget, confirming that the iteration budget materially affects rankings
even when the feedback path is unchanged. Ablations of the remaining channels—topology intent,
operating-point scalars, and the strict JSON protocol versus a loose natural-language variant—across
all checkpoints and multiple seeds are ongoing work and are not claimed here.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 16

16 Li and Kim
Table 10. Terminal causes over the 22 matched benchmark runs of Section 6.
Terminal cause Runs Instances
all hard-pass metrics met 11 7 LC-VCO, 4 op-amp
metric miss at budget 9 3 LC-VCO, 6 op-amp
LLM/client error 1 1 LC-VCO
contract violation 1 1 op-amp
More broadly, because model quality varies sharply across the matched set (seven of eleven
pass the LC-VCO PVT task, with the failures showing distinct signatures) and removing individual
feedback channels changes outcomes as shown above, the loop’s result is attributable to the model’s
reasoning rather than to the fixed scaffolding—the simulator, the deterministic curve-searcher, or
the all-ones reset—acting alone.
8 Failure Analysis
8.1 Observed terminal causes
Because every run logs a machine-readable terminal cause, the failure analysis can start from
counts rather than anecdotes. Table 10 aggregates the 22 matched runs reported in Section 6 (11
LC-VCO, 11 op-amp).
8.2 Failure taxonomy
The taxonomy below classifies how closed-loop analog agents break. Buckets 1 and 2 were observed
as terminal causes or failure signatures in the matched runs; bucket 4 was the terminal cause of
one matched op-amp run (minimax-m3), and bucket 5 the terminal cause of one matched LC-VCO
run (minimax-m2.7) in addition to a transient provider outage that the Anthropic checkpoints
recovered from before completion; buckets 3, 6, 7, and 8 are boundary- or robustness-guarded
behaviors, exercised by repair prompts and red-team unit tests rather than occurring as terminal
causes in this snapshot.
(1) Metric miss at budget: simulation completes but a hard-pass metric is never satisfied at the
worst corner, such as op-amp worst-corner gain stalling below 45 dB (claude-haiku-4-5 at
39.88 dB, deepseek-v4-flash at 37.10 dB) or LC-VCO worst-corner oscillation frequency stuck
outside the [19.5, 20.5] GHz band at the iteration budget—under-frequency for claude-sonnet-
4-6 (19.33 GHz), over-frequency for gpt-5.4-mini (20.52 GHz), or no sustained oscillation at all
for claude-haiku-4-5. A new sub-case under PVT is a stability miss: claude-opus-4-8 and gpt-
5.4-mini reach the worst-corner gain and bandwidth gates but with a negative worst-corner
phase margin, so the phase-margin gate fails them as unstable high-gain points.
(2) Bias or operating-region miss: a sub-class of the above with a recognizable signature—high
worst-corner bandwidth with low or negative worst-corner gain (for example, gemini-2.5-pro
collapses to a negative worst-corner gain and kimi-k2.5 reaches only −13.3 dB worst-corner
gain), indicating a misbiased operating point rather than a near-miss.
(3) Identical-vector stall: the model repeatedly proposes the same failing design vector after
receiving valid measurement feedback. The identical-vector stall guard (Section 4.3) exists
to terminate such runs, but it was not the terminal cause of any run in this matched PVT
snapshot; each metric-miss run instead ran to the iteration budget.
(4) Contract failure (protocol-enforced): the model cannot hold the JSON action contract; after
the structured repair attempts, the run terminates with a logged violation (Section 4.3). On
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 17

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 17
the op-amp PVT task this was the terminal cause for minimax-m3, which repeatedly returned
actions missing required keys and could not be repaired into a well-formed action.
(5) LLM/client loop error: the model API, streaming client, parser, or repair loop returns
an unrecoverable error before a metric-based terminal condition is reached. This was the
terminal cause of one matched LC-VCO run: minimax-m2.7 aborted with an llm_error at
iteration 7 after producing a valid but out-of-band point (20.86 GHz worst corner), its client
returning an unrecoverable error before the budget. The same class also appears as a transient:
an earlier provider outage during the PVT campaign caused the three Anthropic checkpoints
to abort with a client-side error before producing any metric, and they only completed the
matched run after the endpoint recovered. This bucket is a property of the served model plus
client stack at benchmark time, not of the circuit alone.
(6) Harness failure: the benchmark runner terminates before a final result is parsed—a framework-
level failure rather than a model failure. The framework’s data-integrity safeguards (Section 9)
keep this class out of the matched terminal causes.
(7) PDK-content probes (enforced): tool calls that request foundry content by name or via
SKILL injection are blocked by the entry-point whitelist and scrubber (Tiers 1–2 of Table 2).
(8) Reasoning-trace replay (enforced): prior reasoning content re-entering the prompt passes
through the same scrubber as tool results (Tier 3a of Table 2).
The remaining degenerate-search guards—oscillator-death safeguard and unmeasurable-metric
saturation—exist in the protocol (Section 4.3) but were not terminal causes in this snapshot.
8.3 What the failures say about feedback design
The dominant real failure mode is not protocol chaos but partial optimization: models reliably
improve the metrics they can see moving (bandwidth, range) and stall on metrics that require
coupled, multi-variable reasoning (gain at a healthy bias point, per-segment gain flatness). This
is precisely where sanitized operating-point evidence and structured directional hints earn their
audit cost, as the ablation of Section 7 indicates. It also explains why the same model family can
split across tasks: the LC-VCO loop exposes a curve-level summary after every sweep, while the
op-amp loop exposes per-corner scalar AC metrics (now including phase margin) plus the DC table,
leaving more of the cross-corner compensation reasoning to the model.
9 Reproducibility and Security Audit
The reproducibility package has two layers. The public layer is released as an open-source repos-
itory1 and contains the agent code, project specifications, Maestro setup recipes, scrubbed tran-
scripts, benchmark summaries (CSV/JSONL, including every value in Tables 7–10 and Figure 5), and
leak-check scripts. The private layer is supplied by the user: a licensed Virtuoso/Maestro/Spectre
installation, an opened Maestro session for the target testbench, and a foundry PDK that remains
local. This split is intentional: a reviewer can inspect the safety mechanism and replay scrubbed
benchmark evidence without receiving the PDK, and a licensed user can re-run the full loop without
any artifact from us touching their NDA boundary.
The security audit checks three classes of leakage. First, source-level leak checks scan the
repository for private paths, API keys, remote host names, and foundry-shaped tokens; the gate
run accompanying this draft passes with no sensitive patterns in the PC-side source workspace.
Second, bridge-level tests verify that schematic readback, operating-point readback, OCEAN result
generation, and Maestro writeback return only whitelisted fields, and that non-whitelisted SKILL
entry points and nested-call injections are rejected. Third, transcript-level checks verify that the
1https://github.com/lixunqi12/virtuoso-agent
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 18

18 Li and Kim
Table 11. Adversarial extraction results. A trial leaks if the oracle (the production scrub-pattern gate plus a
planted synthetic-canary registry) matches a protected token in the final text returned to the LLM. Fixed
probes are counted per-probe; the adaptive LLM attacker (deepseek-v4-pro, eight turns per session) per-
session.
Attacker Surface / tier Trials Leaks ASR
Fixed probe T1 direct readback 4 0 0%
Fixed probe T2 tool-arg injection 2 0 0%
Fixed probe T3a reasoning replay 2 0 0%
Live readback real LC-VCO + op-amp 2 0 0%
Adaptive LLM all surfaces, 10 sessions 10 0 0%
Table 12. Evidence trace: each framework guarantee maps to a concrete artifact in the reproducibility package.
Guarantee Evidence artifact
Strict JSON contract contract-repair / violation unit tests
Maestro writeback writeback_status=ok in matched runs
Sweep isolation per-cell sweep-dir copy; no duplicated model rows
Leak gate P0 gate pass: no path, PDK token, or API key
Active leak resistance 0% ASR over fixed/live/adaptive attacks (Table 11)
OP readback whitelisted operating-point scalar table, scrubbed
exact text sent back to the LLM does not include PDK model names, absolute paths, or replayed
reasoning content that bypasses the scrubber. The benchmark artifacts published with the paper
are the same files the gate scans, so the audit covers the evidence, not only the code.
Beyond these static checks, we measure leak resistance under active attack. A leak oracle scans
the final text returned to the LLM—the assembled feedback prompt, not the bridge return object—
against the production scrub-pattern gate plus a registry of planted synthetic canaries, one per
protected-asset class. Three attacker modalities exercise it (Table 11): a fixed, deterministic probe
suite that drives the real readback and injection paths with canary-laden input; a live readback of
the two real benchmark schematics through the deployed bridge; and an adaptive LLM red-teamer
(deepseek-v4-pro) that proposes a fresh injection each turn over eight-turn sessions, adapting
to what the bridge returns. Across all three—eight fixed probes, two live cell readbacks, and ten
adaptive sessions—no protected token reached the model (attack-success rate 0). The measurement
is token/sentinel-level under the stated threat model, not a non-interference proof; consistent
with Section 3, path redaction is keyed to a known-root allowlist that covers the deployment’s
/project-rooted paths.
The harness includes data-integrity safeguards so that every reported row is independently
reproducible. Each cell runs against an isolated copy of the Maestro sweep directory, so a model’s
tuning-curve readback reflects only its own simulation and cannot be contaminated by a neigh-
bouring cell. Subprocess result capture is locale-robust, enforcing UTF-8 end to end so that remote
simulation output is read back correctly regardless of host encoding. Together these guarantee that
every value in Tables 7–10 and Figure 5 is attributable to a single, re-runnable cell.
Because the contribution is a safety-bounded engineering system rather than a single algorithm,
each claimed guarantee maps to a concrete, inspectable artifact in the released package (Table 12).
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 19

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 19
10 Discussion
SABLE deliberately optimizes a conservative surface: sizing variables, analysis setup, outputs, and
specs. It is not a topology-synthesis system, and it does not claim that cloud LLMs should see raw
netlists or model cards. This limited scope is a strength for industrial adoption because it matches a
common designer workflow: keep the schematic and PDK local, let an assistant propose numeric
sizing moves, and use the simulator as the source of truth.
The main limitation is statistical breadth. The matched comparison is one audited run per model
per task, and the evidence runs already show that both budget and endpoint state can flip individual
outcomes (Sections 7 and 8); multi-seed replication and at least one additional topology class are
the clear next step, and the protocol— documented reset, fixed Maestro tests, logged terminal
causes—was designed so that such replication is mechanical. The second limitation is that safety
and usefulness trade off against each other. Too little feedback leaves the model guessing; too much
feedback risks leaking design or PDK information. The current evidence suggests that sanitized
topology intent plus numeric operating-point scalars plus curve-level summaries is a useful middle
ground, but channel-by-channel ablations across models remain to be quantified. Third, the threat
model bounds what we claim: provider-side retention of first-emission reasoning traces, timing
side channels, aggregate parameter inference from operating-point scalars, and re-identification of
a design from its anonymized topology graph are explicitly out of scope for a client-side framework
(Table 2). The boundary constrains what each response contains, not what a provider can infer by
correlating many responses over a run; closing that gap is what on-premise inference, and not a
client-side scrubber, is for.
Finally, model comparisons should not be interpreted as permanent rankings. Provider end-
points, context behavior, tool-call discipline, and pricing change over time—our own data contains
one checkpoint whose outcome differs across five days for endpoint reasons alone. The durable
contribution is the benchmark method: fixed reset, fixed Maestro setup, fixed spec, fixed feedback
schema, and logged failure reason. The reported tables are a snapshot generated under that method,
and the method is what we expect to outlive any particular row.
11 Conclusion
This paper presented SABLE, an NDA-safe closed-loop framework for LLM-driven analog circuit
optimization through industrial Virtuoso/Maestro flows. The framework combines an explicit threat
model with audited enforcement status, scoped SKILL access through 28 whitelisted entry points,
PDK/path/model scrubbing on every return path, Maestro setup recipes with landing verification,
AC/DC and swept-metric readback, whitelisted operating-point summaries, a strict JSON contract
with six stop conditions, and best-so-far writeback. In matched eleven-model benchmarks from a
documented reset state, both tasks run under the same three-corner process–voltage–temperature
sign-off with worst-corner gating: 7 of 11 models pass the LC-VCO tuning-curve task, and 4 of
11 pass the harder op-amp task, whose phase-margin gate rejects high-gain but unstable designs.
Feedback-path ablations show that individual sanitized channels are load-bearing, and the failure
taxonomy shows that the dominant failure is partial optimization of visible metrics (worst-corner
frequency placement for the LC-VCO, coupled gain and stability for the op-amp) rather than
protocol collapse. These results support the paper’s central claim: cloud LLMs can be evaluated
and used in real analog EDA loops without exposing raw PDK content, but their reliability must be
measured at the level of complete tool-loop behavior—contract discipline, endpoint stability, and
metric-driven repair—rather than standalone text reasoning.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 20

20 Li and Kim
References
[1] Mohsen Ahmadzadeh, Kaichang Chen, and Georges Gielen. 2025. AnaFlow: Agentic LLM-based Workflow for
Reasoning-Driven Explainable and Sample-Efficient Analog Circuit Sizing. In Proc. ACM/IEEE Int. Conf. on Computer-
Aided Design (ICCAD). IEEE, Munich, Germany, 1–7. arXiv:2511.03697 doi:10.1109/ICCAD66269.2025.11240818 Invited
paper.
[2] Zhixuan Bao, Zhuoyi Lin, Jiageng Wang, Jinhai Hu, Yuan Gao, Yaoxin Wu, Xiaoli Li, and Xun Xu. 2026. AnalogA-
gent: Self-Improving Analog Circuit Design Automation with LLM Agents. arXiv preprint arXiv:2603.23910 (2026).
arXiv:2603.23910 https://arxiv.org/abs/2603.23910
[3] Jitendra Bhandari, Vineet Bhat, Yuheng He, Hamed Rahmani, Siddharth Garg, and Ramesh Karri. 2024. Masala-CHAI:
A Large-Scale SPICE Netlist Dataset for Analog Circuits by Harnessing AI. arXiv preprint arXiv:2411.14299 (2024).
arXiv:2411.14299 https://arxiv.org/abs/2411.14299
[4] Jason Blocklove, Shailja Thakur, Benjamin Tan, Hammond Pearce, Siddharth Garg, and Ramesh Karri. 2025. Automati-
cally Improving LLM-based Verilog Generation using EDA Tool Feedback. ACM Transactions on Design Automation of
Electronic Systems 30, 6 (2025), 1–26. doi:10.1145/3723876
[5] Luca Collini, Siddharth Garg, and Ramesh Karri. 2025. C2HLSC: Leveraging Large Language Models to Bridge the
Software-to-Hardware Design Gap. ACM Transactions on Design Automation of Electronic Systems 30, 6 (2025), 1–24.
doi:10.1145/3734524
[6] Dimple Vijay Kochar, Hanrui Wang, Anantha P. Chandrakasan, and Xin Zhang. 2025. LEDRO: LLM-Enhanced Design
Space Reduction and Optimization for Analog Circuits. In Proc. IEEE Int. Conf. on LLM-Aided Design (ICLAD). IEEE,
Stanford, CA, USA, 141–148. arXiv:2411.12930 doi:10.1109/ICLAD65226.2025.00011
[7] Yao Lai, Sungyoung Lee, Guojin Chen, Souradip Poddar, Mengkang Hu, David Z. Pan, and Ping Luo. 2025. AnalogCoder:
Analog Circuit Design via Training-Free Code Generation. In Proc. AAAI Conference on Artificial Intelligence. Association
for the Advancement of Artificial Intelligence, Washington, DC, USA, 379–387. arXiv:2405.14918 doi:10.1609/aaai.
v39i1.32016
[8] Yao Lai, Souradip Poddar, Sungyoung Lee, Guojin Chen, Mengkang Hu, Bei Yu, Ping Luo, and David Z. Pan. 2026.
AnalogCoder-Pro: Unifying Analog Circuit Generation and Optimization via Multi-modal LLMs. IEEE Transactions on
Computer-Aided Design of Integrated Circuits and Systems (2026), 1–1. arXiv:2508.02518 [cs.AR] doi:10.1109/TCAD.
2026.3673493 Early Access.
[9] Jintao Li, Haochang Zhi, Ruiyu Lyu, Wangzhen Li, Zhaori Bi, Keren Zhu, Yanhan Zeng, Weiwei Shan, Changhao Yan,
Fan Yang, Yun Li, and Xuan Zeng. 2024. AnalogGym: An Open and Practical Testing Suite for Analog Circuit Synthesis.
In Proc. ACM/IEEE Int. Conf. on Computer-Aided Design (ICCAD). ACM, New York, NY, USA, 1–9. arXiv:2409.08534
doi:10.1145/3676536.3697117
[10] Chang Liu and Danial Chitnis. 2026. EEsizer: LLM-Based AI Agent for Sizing of Analog and Mixed Signal Circuit.
IEEE Transactions on Circuits and Systems I: Regular Papers 73, 6 (2026), 3825–3834. arXiv:2509.25510 doi:10.1109/TCSI.
2025.3646359
[11] Mingjie Liu, Teodor-Dumitru Ene, Robert Kirby, Chris Cheng, Nathaniel Pinckney, Rongjian Liang, Jonah Alben,
Himyanshu Anand, Sanmitra Banerjee, Ismet Bayraktaroglu, et al. 2023. ChipNeMo: Domain-Adapted LLMs for Chip
Design. arXiv preprint arXiv:2311.00176 (2023). arXiv:2311.00176 https://arxiv.org/abs/2311.00176
[12] Asal Mehradfar, Xuzhe Zhao, Yue Niu, Sara Babakniya, Mahdi Alesheikh, Hamidreza Aghasi, and Salman Avestimehr.
2024. AICircuit: A Multi-Level Dataset and Benchmark for AI-Driven Analog Integrated Circuit Design. arXiv preprint
arXiv:2407.18272 (2024). arXiv:2407.18272 https://arxiv.org/abs/2407.18272
[13] Ruidi Qiu, Grace Li Zhang, Rolf Drechsler, Tsungyi Ho, Ulf Schlichtmann, and Bing Li. 2025. ConfiBench: Automatic
Testbench Generation with Confidence-Based Scenario Mask and Testbench Ensemble using LLMs for HDL Design.
ACM Transactions on Design Automation of Electronic Systems (2025). doi:10.1145/3773087 Just Accepted, article
№ 3773087.
[14] E Bhawani Eswar Reddy, Sutirtha Bhattacharyya, Ankur Sarmah, Fedrick Nongpoh, Karthik Maddala, and Chandan
Karfa. 2025. LHS: LLM Assisted Efficient High-level Synthesis of Deep Learning Tasks. ACM Transactions on Design
Automation of Electronic Systems 30, 6 (2025), 1–27. doi:10.1145/3734523
[15] Yichen Shi, Zhuofu Tao, YuHao Gao, Tianjia Zhou, Cheng Chang, Yaxin Wang, Bingyu Chen, Genhao Zhang, Alvin Liu,
Zhiping Yu, Ting-Jung Lin, and Lei He. 2025. AMSnet-KG: A Netlist Dataset for LLM-based AMS Circuit Auto-design
Using Knowledge Graph RAG. ACM Transactions on Design Automation of Electronic Systems 30, 6 (2025), 1–37.
doi:10.1145/3736166
[16] Zhuofu Tao, Yichen Shi, Yiru Huo, Rui Ye, Zonghang Li, Li Huang, Chen Wu, Na Bai, Zhiping Yu, Ting-Jung Lin, and
Lei He. 2024. AMSNet: Netlist Dataset for AMS Circuits. In Proc. IEEE LLM Aided Design Workshop (LAD). IEEE, San
Jose, CA, USA, 1–5. arXiv:2405.09045 doi:10.1109/LAD62341.2024.10691781
[17] Prashanth Vijayaraghavan, Luyao Shi, Ehsan Degan, and Xin Zhang. 2024. CircuitSynth: Leveraging Large Language
Models for Circuit Topology Synthesis. In Proc. IEEE LLM Aided Design Workshop (LAD). IEEE, San Jose, CA, USA, 1–6.
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

## Page 21

SABLE: An NDA-Safe Closed-Loop LLM Framework for Analog Circuit Optimization in Industrial EDA Flows 21
arXiv:2407.10977 doi:10.1109/LAD62341.2024.10691716
[18] Zining Wang, Jian Gao, Weimin Fu, Xiaolong Guo, and Xuan Zhang. 2025. AnalogSAGE: Self-evolving Analog
Design Multi-Agents with Stratified Memory and Grounded Experience. arXiv preprint arXiv:2512.22435 (2025).
arXiv:2512.22435 https://arxiv.org/abs/2512.22435
[19] Haoyuan Wu, Zhuolun He, Xinyun Zhang, Xufeng Yao, Su Zheng, Haisheng Zheng, and Bei Yu. 2024. ChatEDA:
A Large Language Model Powered Autonomous Agent for EDA. IEEE Transactions on Computer-Aided Design of
Integrated Circuits and Systems 43, 10 (2024), 3184–3197. arXiv:2308.10204 doi:10.1109/TCAD.2024.3383347
[20] Kangwei Xu, Grace Li Zhang, Xunzhao Yin, Cheng Zhuo, Ulf Schlichtmann, and Bing Li. 2026. HLSRewriter: Efficient
Refactoring and Optimization of C/C++ Code with LLMs for High-Level Synthesis. ACM Transactions on Design
Automation of Electronic Systems 31, 4 (2026), 1–21. doi:10.1145/3749986
[21] Tingjie Yang, Bo Li, Yongfu Li, Wei Mao, and Genquan Han. 2025. Circuit-Agent: A Large Language Model Based
Circuit Agent Framework for Analog/Mixed-signal Circuit Design Automation. In Proc. IEEE Int. Conf. on Artificial
Intelligence Circuits and Systems (AICAS). IEEE, Bordeaux, France, 1–5. doi:10.1109/AICAS64808.2025.11173131
[22] Yuxuan Yin, Yu Wang, Boxun Xu, and Peng Li. 2024. ADO-LLM: Analog Design Bayesian Optimization with In-Context
Learning of Large Language Models. In Proc. ACM/IEEE Int. Conf. on Computer-Aided Design (ICCAD). ACM, New York,
NY, USA, 1–9. arXiv:2406.18770 doi:10.1145/3676536.3676816
[23] Xi Yu, Dmitrii Torbunov, Soumyajit Mandal, and Yihui Ren. 2026. AutoSizer: Automatic Sizing of Analog and Mixed-
Signal Circuits via Large Language Model (LLM) Agents. arXiv preprint arXiv:2602.02849 (2026). arXiv:2602.02849
https://arxiv.org/abs/2602.02849
[24] Haoyi Zhang, Shizhao Sun, Yibo Lin, Runsheng Wang, and Jiang Bian. 2025. AnalogXpert: Automating Analog Topology
Synthesis by Incorporating Circuit Design Expertise into Large Language Models. In Proc. Int. Symp. of Electronics
Design Automation (ISEDA). IEEE, Hong Kong, China, 772–777. arXiv:2412.19824 doi:10.1109/ISEDa65950.2025.11100627
ACM Trans. Des. Autom. Electron. Syst., Vol. 1, No. 1, Article . Publication date: July 2026.

