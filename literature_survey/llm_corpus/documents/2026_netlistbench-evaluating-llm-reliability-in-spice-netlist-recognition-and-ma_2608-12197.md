---
title: "NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition and Manipulation"
authors: "Ma, Jiarui; Wang, Jianghan; Ma, Yuheng; Zhuang, Ziyi; Liu, Xiaoguang"
date: "2026/08/12"
category: "model_benchmark"
source_url: "https://arxiv.org/abs/2608.12197"
pdf: "papers/2026_netlistbench-evaluating-llm-reliability-in-spice-netlist-recognition-and-ma_2608-12197.pdf"
sha256: "1dcc9c7c322098a8c3090d96afc9695fbcc95d9fa2f662325e9627dc92722590"
pages_extracted: 8
---

# NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition and Manipulation

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

NetlistBench: Evaluating LLM Reliability in SPICE Netlist
Recognition and Manipulation
Jiarui Ma
Southern University of Science and
Technology
School of Microelectronics
Shenzhen, China
12312626@mail.sustech.edu.cn
Jianghan Wang
Southern University of Science and
Technology
School of Microelectronics
Shenzhen, China
12311107@mail.sustech.edu.cn
Yuheng Ma
Southern University of Science and
Technology
School of Microelectronics
Shenzhen, China
12412108@mail.sustech.edu.cn
Ziyi Zhuang
Southern University of Science and
Technology
School of Microelectronics
Shenzhen, China
12412728@mail.sustech.edu.cn
Xiaoguang Liu
Southern University of Science and
Technology
School of Microelectronics
Shenzhen, China
liuxg@sustech.edu.cn
Abstract
Large Language Models (LLMs) are increasingly used in circuit
design workflows, yet their reliability on simulator-facing SPICE
netlist recognition and manipulation remains poorly understood
and is rarely separated from high-level design reasoning. Although
netlists are textual, they encode structured circuit objects through
topology and parameters. We present NetlistBench, a structure-
verified benchmark for SPICE netlist recognition and manipulation.
NetlistBench contains 2,342 cases across 24 task families, cover-
ing parameter and connectivity recognition and edits, hierarchical
operations, equivalence judgment, and long-horizon compound
editing. Model outputs are evaluated by a deterministic structure-
aware oracle. Across six non-thinking LLMs, performance varies
substantially with operation-level structural complexity. Simple
local edits reach 96%–100% accuracy, while device addition drops to
41%–83% and equivalence judgment to 49%–90%. Enabling reason-
ing substantially improves weaker models but does not eliminate
structure-preservation failures, with performance still degrading
sharply as the edit horizon increases. NetlistBench identifies netlist
reliability as a distinct bottleneck for trustworthy LLM-based circuit
design automation.
CCS Concepts
• Hardware → Electronic design automation; • Computing
methodologies → Natural language processing.
Keywords
Large language models, circuit representation, SPICE netlists, netlist
recognition and manipulation
Permission to make digital or hard copies of all or part of this work for personal or
classroom use is granted without fee provided that copies are not made or distributed
for profit or commercial advantage and that copies bear this notice and the full citation
on the first page. Copyrights for components of this work owned by others than the
author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or
republish, to post on servers or to redistribute to lists, requires prior specific permission
and/or a fee. Request permissions from permissions@acm.org.
,
© 2026 Copyright held by the owner/author(s). Publication rights licensed to ACM.
ACM Reference Format:
Jiarui Ma, Jianghan Wang, Yuheng Ma, Ziyi Zhuang, and Xiaoguang Liu.
2026. NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition
and Manipulation. In Proceedings of . ACM, New York, NY, USA, 8 pages.
1 Introduction
Large language models (LLMs) are increasingly explored across the
lifecycle of integrated circuit (IC) design, including domain-adapted
chip-design assistance, analog circuit generation, simulation-driven
optimization, and multimodal netlist extraction [10, 15, 16, 20, 21,
29, 31]. By treating hardware artifacts as structured code, LLM-
based systems hold the potential to accelerate electronic design
automation (EDA) by translating specifications, editing topologies,
and driving simulator- or layout-facing tools.
Despite differences in their inputs, objectives, and tool interfaces,
many of these workflows share SPICE netlists as a recurring rep-
resentation layer (Figure 1). In generation-oriented settings, LLMs
may synthesize netlists from specifications, schematics, or circuit
images [1, 10, 15, 31]. In simulation-driven optimization loops, they
often revise existing netlists according to performance or simulator
feedback [20, 26, 29]. In netlist-to-schematic or netlist-to-layout
workflows, they may interpret connectivity, hierarchy, and device
relationships to guide downstream visualization or physical de-
sign [5, 8, 13, 14, 17]. Across these settings, key model actions fre-
quently take the form of reading, generating, editing, or reasoning
over netlist text.
Errors at this representation layer can directly corrupt down-
stream simulation, optimization, or layout reasoning. Consequently,
failures in LLM-based circuit workflows may originate either from
high-level design reasoning or from low-level netlist corruption,
yet existing evaluations rarely separate these two sources. Reli-
able netlist operation is therefore a prerequisite for trustworthy
LLM-based circuit design workflows.
However, this prerequisite has not been directly quantified. Gen-
eral LLM-for-code benchmarks commonly evaluate executable func-
tional correctness through unit tests or repository test suites [3, 12],
but do not capture the device-specific terminal semantics, shared-
node connectivity, and ordered subcircuit interfaces of SPICE netlists.
arXiv:2608.12197v1
[eess.SY]
12
Aug
2026

## Page 2

, , Ma et al.
Figure 1: SPICE netlists as a common representation layer in
LLM-based circuit design workflows.
Existing evaluations of LLM-based circuit design typically focus
on end-to-end outcomes, such as syntactic validity, simulation
success, specification improvement, or downstream task comple-
tion [15, 20, 26]. Recent circuit-oriented benchmarks mainly assess
domain-level capabilities, including circuit interpretation, topol-
ogy reasoning, schematic understanding, AMS-domain multimodal
reasoning, or graph-structured reasoning [23, 25, 27]. While these
evaluations reveal important limitations, they do not isolate the ele-
mentary operations required to interpret and modify SPICE netlists
correctly. Accordingly, the reliability of LLMs in performing
core SPICE netlist operations remains unclear.
To address this gap, we introduce NetlistBench, a structure-
verified benchmark for evaluating whether LLMs can reliably rec-
ognize and manipulate analog SPICE netlists as structured circuit
representations. NetlistBench focuses on structure-level operations
that test a model’s ability to recover circuit structure or apply
explicit modifications without introducing unintended changes.
Model outputs are evaluated through a deterministic canonical cir-
cuit representation that captures devices, ordered terminal bindings,
node identities, parameters, directives, and hierarchy.
The main contributions of this work are:
• We formulate SPICE netlist reliability as a representation-
level evaluation problem, focusing on whether LLMs can
correctly recognize and manipulate netlists as structured
circuit artifacts.
• We introduce NetlistBench, a structure-verified benchmark
covering netlist structural-property recognition and natural-
language-guided netlist manipulation. Using NetlistBench,
we evaluate representative frontier, flash-class, and open-
weight LLMs and show that reliability varies sharply across
operation type and task horizon.
• We develop a structure-aware evaluation pipeline based on
canonical circuit representations, enabling manipulation out-
puts to be verified beyond raw text matching or final simu-
lation outcomes.
2 Background
2.1 SPICE Netlists as Structured Circuit
Representations
SPICE netlists are simulator-facing circuit descriptions that encode
devices, terminals, nodes, parameters, models, ports, and subcir-
cuit hierarchies in a compact, positional textual format [18, 19].
Although a netlist appears as a sequence of text lines, its underly-
ing semantics correspond to a structured circuit object. Each device
statement typically begins with an instance prefix, followed by
an ordered sequence of node names connected to specific device
terminals, a model reference, and optional parameter assignments.
For hierarchical circuits, subcircuit definitions (.subckt) establish
ordered port interfaces, and each subcircuit instance binds external
nodes to internal ports strictly according to their positional order
in the instance statement.
A defining characteristic of this representation is that electri-
cal connectivity is encoded implicitly through node-name sharing
rather than explicit terminal-to-terminal links. For circuit simula-
tion, node labels are sufficient because devices contribute equations
to the modified nodal analysis (MNA) system according to the nodes
attached to their terminals [9]. Terminals sharing the same node
name are treated as electrically connected. For structural analysis
and manipulation, however, these connections are not represented
as explicit terminal-to-terminal links; the circuit topology must
be reconstructed from terminal–node bindings across the entire
netlist.
These properties make netlist operations different from ordinary
text editing. A correct edit must preserve terminal-role bindings,
maintain consistent node identities, and avoid unintended changes
to unrelated devices or subcircuit interfaces. NetlistBench there-
fore evaluates netlist outputs through structural equivalence to a
canonical circuit representation rather than through surface-string
similarity.
2.2 Reported Limitations of LLMs on SPICE
Netlists
Existing studies have reported several limitations of LLMs in pro-
cessing circuit representations and SPICE-like netlists. At the circuit-
reasoning level, benchmarks such as CIRCUIT and AMSbench show
that LLMs can struggle with topology-heavy circuit interpreta-
tion and multi-step circuit reasoning [23, 25]. At the generation
and adaptation level, systems such as SPICEPilot, SPICEAssistant,
AnalogCoder, and Spice Wizard rely on simulation feedback, syn-
tax checks, or tool-assisted repair loops to improve SPICE code or
netlist generation, indicating that unvalidated LLM outputs may not
provide sufficient reliability for direct downstream use [6, 15, 20, 26].
At the netlist-analysis level, SPICED studies LLM-aided detection
and localization of syntactical bugs and analog Trojans in A/MS
netlists, further showing that node, parameter, subcircuit, and con-
nectivity errors are meaningful failure classes in SPICE-like repre-
sentations [2].
Complementary representation-oriented work suggests why
these failures are difficult to avoid with ordinary text modeling
alone. CircuitFormer highlights the mismatch between standard

## Page 3

NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition and Manipulation , ,
language tokenization and the graph-structured semantics of cir-
cuits, while Image2Net evaluates diagram-to-netlist conversion
using graph-structured netlist comparison rather than raw string
matching [11, 31]. Taken together, these findings suggest that LLM
failures on SPICE netlists are not merely surface-level syntax issues,
but are closely related to the difficulty of preserving structured
circuit semantics in a positional textual representation.
However, these studies evaluate netlists within broader gener-
ation, simulation, conversion, or reasoning pipelines rather than
isolating representation-level netlist operations.
3 Benchmark Design
NetlistBench evaluates representation-level netlist reliability through
two modalities: recognition, which extracts or compares circuit
structure, and manipulation, which applies explicit natural-language
edits to SPICE netlists. This design separates structure interpre-
tation and structure-preserving transformation from high-level
design reasoning, simulator behavior, and optimization.
Table 1: Composition of the NetlistBench source corpus.
Source subset Count Structural summary
AnalogGenie–Simple 492 Median: 5 devices
AnalogGenie–Medium 1,752 Median: 20 devices
AnalogGenie–Complex 594 Median: 36 devices;
maximum: 69
Flat subtotal 2,838 Approximately 58,000
device instances
ALIGN hierarchical 931 21 topology families; 3–4
subcircuits per netlist
Total 3,769
3.1 Source Corpus
NetlistBench uses two complementary SPICE netlist sources, sum-
marized in Table 1. AnalogGenie provides flat CMOS analog netlists
originally developed for topology discovery [7]; after normalization,
these circuits are used to construct flat recognition and manipu-
lation tasks. ALIGN provides hierarchical analog netlists from a
layout automation flow [5]; its circuits contain multiple .subckt
definitions and top-level instance calls and are used for hierarchical
tasks.
3.2 Instance Construction Pipeline
NetlistBench constructs evaluation instances through a determin-
istic, template-driven pipeline rather than stochastic or uncon-
strained generation. Starting from source SPICE netlists, the pipeline
applies family-specific transformation rules, syntax perturbations,
and prompt templates to produce reproducible benchmark cases.
Each instance is represented as a self-contained triplet:
I
𝑖 =

N(𝑖)
src, T(𝑖)
inst, Y(𝑖)
target

, (1)
where N(𝑖)
src denotes the source SPICE netlist, T(𝑖)
inst denotes the ex-
plicit task instruction, and Y(𝑖)
target denotes the task-specific target
Figure 2: NetlistBench pipeline for generating benchmark
cases and evaluating model outputs with a structure-aware
oracle.
used for evaluation. Both T(𝑖)
inst and Y(𝑖)
target are produced by deter-
ministic, family-specific templates, ensuring that each instance has
an unambiguous instruction and reproducible ground truth.
The form of Y(𝑖)
target depends on the task type. For manipulation
tasks, it is the uniquely determined target SPICE netlist that realizes
the requested structural transformation. For recognition tasks, it
is the canonical JSON answer derived from the source netlist. For
equivalence judgment tasks, it is the binary structural-equivalence
verdict.
All instances undergo automated construction-time validation
before inclusion. The checker ensures that manipulation targets
implement exactly the specified structural changes without un-
intended edits, and that recognition and equivalence targets are
consistent with the canonical IR of the corresponding input netlist
or netlist pair.
3.3 Structure-Aware Evaluation Oracle
This structure-aware evaluation follows the broader principle that
circuit artifacts should be compared through their underlying con-
nectivity and device structure rather than by surface text. Simi-
lar graph-structured evaluation ideas have been used in diagram-
to-netlist conversion, where generated and reference netlists are
compared through heterogeneous circuit graphs rather than raw
strings [31].
Concretely, each model output and the reference target are
parsed into the canonical IR —a normalized structure that lists
every device by instance name with its device kind, ordered termi-
nal nodes, and parameters, together with top-level directives and,
for hierarchical circuits, each subcircuit’s port interface and inter-
nal devices. An output passes only if its IR matches the reference IR
under a fixed set of semantics-preserving normalizations: the two
must contain the same set of named devices, with no missing or ex-
tra device, identical terminal-node bindings, parameters equal up to
numeric normalization (e.g., 1k equals 1000), and identical top-level

## Page 4

, , Ma et al.
Figure 3: Overview of the NetlistBench benchmark, showing the distribution of cases across task families and representative
task examples.
directives; symmetric two-terminal passives (𝑅/𝐶/𝐿) are compared
with unordered terminals, and subcircuit definitions must agree
on their port interface (port order treated as semantic except for
extraction tasks), internal devices, and directives. This exact-match-
up-to-normalization rule directly encodes the name-preservation
and locality constraints of edit tasks: renaming an untouched node,
dropping or duplicating a device, or perturbing an unrelated param-
eter each surfaces as an IR mismatch and fails the case. The same
canonical-IR comparator scores SPICE and PySpice [22] outputs
through a uniform interface.
For the equivalence-judgment family, the task is instead to decide
whether two netlists denote the same circuit up to consistent node
and instance renaming. To validate the generated ground-truth
labels for this family, we projected the IR into a labeled bipartite
device–net graph—device nodes labeled by type, model, and normal-
ized parameter signatures, and device–net edges labeled by terminal
roles—and ran a VF2 graph-isomorphism check [4] confirming that
positive pairs are isomorphic and negative pairs are not. This iso-
morphism check audits equivalence-case labels only and is not part
of scoring model outputs: scoring must instead preserve device
and node names, whereas VF2 equates circuits up to renaming and
would therefore mask the very name- and locality-violations that
the edit tasks are designed to detect.
3.4 Task Families
NetlistBench contains 24 task families across recognition and ma-
nipulation, as summarized in Figure 3. The recognition modality
contains 800 cases across eight families. Seven evaluate structured
extraction of device parameters, terminal connectivity, node in-
cidence, subcircuit interfaces, and instance mappings, while the
eighth evaluates structural equivalence between netlist pairs.
The manipulation modality contains 1,542 cases across 16 fami-
lies. Six single-edit families cover connectivity editing, device ad-
dition, removal and replacement, parameter editing, and rename
propagation. Five flat compound families combine 3, 6, 9, 12, or 15
dependent operations, and five hierarchical families evaluate sub-
circuit expansion, interface modification, and multi-step internal
editing.
4 Evaluation
4.1 Experimental Setup and Protocol
We evaluate six single-shot non-thinking models spanning fron-
tier, flash-class, and open-weight tiers: Claude Sonnet 4.6, GPT-4.1,
Gemini 2.5 Flash, DeepSeek-V4-Flash, Qwen3.6-Flash, and Qwen3-
30B-A3B. All are queried through official provider APIs with explicit
reasoning modes disabled, so the main comparison measures base
netlist-operation reliability rather than reasoning elicitation. As a
reasoning reference, we additionally evaluate the same DeepSeek-
V4-Flash with its native thinking mode enabled; this column is
reported separately and excluded from the non-thinking compar-
ison. We also run two controlled secondary analyses on a paired
stratified subset: SPICE versus PySpice output representation, and
direct prompting versus native thinking and CoT prompting [28].

## Page 5

NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition and Manipulation , ,
Table 2: Per-family NetlistBench pass rates (%).
C/G/Ge/QF/Q30/DS denote Claude-S4.6, GPT-4.1, Gemini-
2.5-F, Qwen3.6-F, Qwen3-30B, and DeepSeek-V4-F. DS+R† is
the reasoning reference, excluded from bolding; underlines
exceed all non-thinking models.
Task 𝑛 C G Ge QF Q30 DS DS+R†
Conn. edit 100 97 91 93 59 47 85 99
Dev. add 100 83 72 57 54 43 41 55
Dev. remove 100 100 100 100 98 97 100 100
Dev. replace 100 95 93 86 75 61 78 91
Param. edit 100 100 99 98 99 96 98 99
Rename prop. 100 99 98 97 85 89 92 100
Comp. 3 100 80 71 70 34 28 44 74
Comp. 6 100 57 58 33 6 1 18 63
Comp. 9 100 56 51 21 2 1 6 50
Comp. 12 100 41 39 17 0 0 1 33
Comp. 15 97 26 34 6 0 0 0 31
Subckt inline 45 56 62 33 16 2 42 98
Port swap 100 88 80 83 33 19 65 97
Subckt comp. 3 100 79 69 55 39 21 32 66
Subckt comp. 6 100 67 55 36 5 4 8 54
Subckt comp. 9 100 42 43 41 1 1 3 47
Edit subtotal 1542 74 70 59 39 33 45 71
Dev. param. 100 99 100 100 100 99 100 100
Sem. term. conn. 100 99 73 74 25 6 82 100
Ord. term. conn. 100 100 100 99 99 100 100 100
Node inc. 100 98 42 59 21 15 53 96
Subckt ports 100 100 100 100 100 99 100 100
Inst. port map 100 100 96 87 89 65 96 100
Term. neigh. inc. 100 97 13 20 4 2 12 93
Equiv. judge 100 90 67 61 55 56 49 97
Recog. subtotal 800 98 74 75 62 55 74 98
Overall 2342 82 71 64 47 41 55 81
Each model is queried once per case with deterministic decoding,
and retries are used only for transport failures. Responses are graded
by the structure-aware oracle in Section 3.3 and reduced to binary
pass/fail outcomes: manipulation outputs must match the reference
structure, recognition outputs must match the canonical JSON an-
swer, and equivalence judgments must match the reference verdict.
The parser tolerates incidental code fences, but empty, unparsable,
or structurally invalid outputs fail. We report pass rates with Wil-
son 95% confidence intervals [30]; Table 2 gives the per-family
case count 𝑛, and aggregate ablation intervals are stated explicitly.
Subtotals and overall scores are case-weighted. Reasoning-mode
results are single samples and may carry run-to-run variance.
Availability. NetlistBench is publicly available at https://github.
com/WoshiMayou/NetlistBench. The repository contains all 2,342
benchmark cases across the 24 task families, the deterministic
structure-aware oracle, seeded case-generation scripts, evaluation
runners, and the per-family prompt templates required to repro-
duce the benchmark evaluation. The code is released under the
Apache-2.0 license, while the benchmark cases and prompts are
released under CC BY 4.0.
Figure 4: Pass rates under compound editing with increasing
numbers of dependent edits for (a) flat netlists and (b) hier-
archical subcircuit netlists. Lines show the mean pass rates,
and shaded regions indicate the corresponding variation.
4.2 Performance Across Task Families
Table 2 shows substantial variation across operation types. Local
operations that primarily modify explicit text are the most reliable:
device removal and parameter editing reach 96%–100% across mod-
els. Reliability decreases for operations that require maintaining
connectivity or introducing new structure, including connectiv-
ity editing, device replacement, device addition, subcircuit port
swapping, and inline expansion.
Recognition exhibits a similar distinction. Explicit attributes such
as device parameters, ordered terminal lists, and subcircuit ports
are extracted with high accuracy, whereas relational queries vary
substantially across models. Node incidence ranges from 15% to
98%, semantic terminal connectivity from 6% to 99%, and terminal-
neighbor incidence from 2% to 97%. Structural equivalence judg-
ment also remains challenging, with pass rates from 49% to 90%.
Overall non-thinking pass rates range from 41% to 82%. The re-
sults indicate that current models are considerably more reliable on
localized attribute extraction and substitution than on recovering
or preserving the implicit connectivity graph. Enabling reasoning
raises DeepSeek-V4-Flash from 55% to 81%, but does not consis-
tently surpass the strongest non-thinking model.
4.3 Long-Horizon Compound Editing
The compound editing families chain 3, 6, 9, 12, and 15 mutually
dependent edits into a single instruction, revealing a substantial
reliability degradation in NetlistBench. Accuracy declines consis-
tently as the edit horizon increases (Figure 4). Even models with
strong short-horizon performance degrade substantially: Claude
drops from 80% at 3 steps to 26% at 15 steps, while GPT-4.1 drops
from 71% to 34%. The remaining models decline to near-zero ac-
curacy at longer horizons, with Gemini decreasing from 70% to
6%, DeepSeek-V4-Flash from 44% to 0%, and both Qwen3.6-Flash
and the open-weight Qwen3-30B from 34%/28% to 0%. Crucially,
reasoning does not eliminate this trend: DeepSeek-V4-Flash with
reasoning enabled, although far stronger at short horizons (74% at
3 steps), still falls to 31% at 15 steps.
The degradation is not simply a consequence of weak atomic
editing. Long-horizon compound tasks require models to track mul-
tiple dependent edit intents, update intermediate circuit state, and
preserve edit locality across an extended instruction sequence. Be-
cause the edits are mutually dependent, errors compound across

## Page 6

, , Ma et al.
Table 3: Mitigation results on the paired subset (𝑛=718), with
Wilson 95% confidence intervals.
Model Direct Native CoT PySpice
DeepSeek-V4-F 52 81 75 58
[48, 55] [78, 83] [72, 78] [55, 62]
Qwen3.6-F 45 85 78 45
[41, 48] [82, 88] [75, 81] [42, 49]
the sequence: even a high per-edit success rate yields a low joint
success probability once many edits must all be correct. The same
downward trend appears in the hierarchical compound family, in-
dicating that this effect is not limited to flat netlists. These results
show that high reliability on isolated edits does not translate into
reliable multi-step netlist transformation. This pattern is consis-
tent with broader observations that small per-step error rates can
compound sharply over long execution horizons [24].
4.4 Representation and Reasoning
We study two mitigations on DeepSeek-V4-Flash and Qwen3.6-
Flash: changing the circuit representation (SPICE→PySpice) and en-
abling explicit reasoning (native thinking and CoT prompting [28]).
Both analyses use the same paired, stratified subset (30 cases per
family, 28 for Comp. 15, 𝑛=718); Table 3 reports overall pass rates
with Wilson 95% intervals, and paired arms are compared with
McNemar’s test.
On this subset, reasoning gives the larger aggregate gains: native
thinking raises both models by roughly 30–40 points (𝑝 < 10−6),
while CoT also improves performance but less strongly. The task-
level results in Table 5 and Figure 6 show similar gains across
several structure-heavy families. By contrast, PySpice has a smaller
and less consistent effect: it improves DeepSeek-V4-Flash overall
(𝑝 < 0.001), but not Qwen3.6-Flash (𝑝 = 0.51), with task-level trends
shown in Table 4 and Figure 5.
Because each per-family cell has only 30 cases, we treat task-level
patterns as descriptive. Overall, reasoning is the stronger mitigation
here, but errors still concentrate on long-horizon compound edits,
hierarchical operations, and relational structural queries; neither
mitigation makes current LLMs sufficiently reliable for unverified
netlist editing.
5 Discussion
5.1 Implications for LLM-Based Netlist Editing
NetlistBench shows that netlist reliability cannot be reduced to
general circuit knowledge or output-format compliance. Models
tend to perform better on localized edits, such as parameter changes
and device removal, while showing reduced reliability on tasks in-
volving structural attachment, ordered port handling, equivalence
judgment, or multi-step edits. The observed failures are frequently
structural rather than purely procedural: recognition outputs usu-
ally follow the required JSON schema but contain incorrect circuit
facts, while manipulation failures involve omitted edits, duplicated
edits, loss of locality, unintended terminal rebinding, and topology
drift.
This error pattern follows directly from the representation prop-
erties described in Section 2.1. Local substitutions and deletions
often require only limited changes to already explicit text, while
attachment, hierarchy, equivalence, and compound editing require
the model to maintain an implicit circuit graph across terminal
roles, node identities, and subcircuit interfaces. The observed fail-
ures therefore indicate a structure-preservation bottleneck: models
can often produce syntactically plausible netlists, but still lose edit
locality, perturb unrelated bindings, or fail to maintain consistent
topology across multiple dependent operations.
These results suggest that current LLMs should not be treated
as standalone, unverified netlist editors. Reasoning modes and al-
ternative surface representations can mitigate some failures, but
neither fully resolves the structure-preservation problem. More
robust workflows may need to decompose complex edits, verify
each intermediate netlist structurally, and provide feedback when
unintended changes are detected. Future work may also explore
graph-based or other structured circuit representations that expose
connectivity more directly than raw SPICE text.
5.2 Limitations
NetlistBench evaluates bounded, circuit-block-level netlists rather
than industrial-scale post-layout decks. Although the compound
tasks increase the number of dependent edits, all evaluated netlists
fit within the tested models’ context windows. The benchmark
therefore does not assess long-context retrieval, hierarchical parti-
tioning, or direct processing of extracted netlists containing millions
of device and parasitic statements.
The current release also covers a restricted circuit and syn-
tax domain, primarily flat and hierarchical analog CMOS blocks
from AnalogGenie and ALIGN. It does not comprehensively cover
symbolic .param expressions, complex .model cards, behavioral
or controlled sources, include hierarchies, extracted parasitics, or
simulator- and PDK-specific syntax. The reported results therefore
should not be assumed to transfer unchanged to RF, power, digital,
or mixed-signal netlists.
Finally, task instructions are generated from deterministic tem-
plates to isolate structural capabilities and enable unambiguous
grading. They do not capture the full linguistic variability or de-
sign intent of real designer–assistant interactions. Each model–case
pair is evaluated once, so the results characterize the tested API
snapshots rather than complete output distributions.
6 Conclusion
NetlistBench shows that netlist reliability is a distinct bottleneck for
LLM-based circuit design. Current models often handle local substi-
tutions and simple extraction, but remain fragile on connectivity-
sensitive edits, hierarchy manipulation, structural equivalence, and
long-horizon compound transformations. Reasoning improves per-
formance, yet does not make LLMs reliable unverified editors of
simulator-facing netlists. These findings motivate decomposed edit-
ing workflows, structure-aware verification after each edit, and
circuit representations that expose topology more directly than raw
SPICE text.

## Page 7

NetlistBench: Evaluating LLM Reliability in SPICE Netlist Recognition and Manipulation , ,
References
[1] Jitendra Bhandari, Vineet Bhat, Yuheng He, Siddharth Garg, Hamed Rahmani,
and Ramesh Karri. 2024. Masala-CHAI: A Large-Scale SPICE Netlist Dataset for
Analog Circuits by Harnessing AI. arXiv:2411.14299 [cs.AR]
[2] Jayeeta Chaudhuri, Dhruv Thapar, Arjun Chaudhuri, Farshad Firouzi, and Krish-
nendu Chakrabarty. 2024. SPICED: Syntactical Bug and Trojan Pattern Identifica-
tion in A/MS Circuits using LLM-Enhanced Detection. arXiv:2408.16018 [cs.AR]
[3] Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira
Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman,
et al. 2021. Evaluating Large Language Models Trained on Code. arXiv preprint
arXiv:2107.03374 (2021).
[4] Luigi P. Cordella, Pasquale Foggia, Carlo Sansone, and Mario Vento. 2004. A
(Sub)Graph Isomorphism Algorithm for Matching Large Graphs. IEEE Trans-
actions on Pattern Analysis and Machine Intelligence 26, 10 (2004), 1367–1372.
doi:10.1109/TPAMI.2004.75
[5] Tonmoy Dhar, Kishor Kunal, Yaguang Li, Meghna Madhusudan, Jitesh Poojary,
Arvind K. Sharma, Wenbin Xu, Steven M. Burns, Ramesh Harjani, Jiang Hu,
Desmond A. Kirkpatrick, Parijat Mukherjee, Soner Yaldiz, and Sachin S. Sapat-
nekar. 2021. ALIGN: A System for Automating Analog Layout. IEEE Design &
Test 38, 2 (2021), 8–18. doi:10.1109/MDAT.2020.3042177
[6] Aakash Divakar, Aditya Anekar, and Manas Kulkarni. 2026. Spice Wizard: A Uni-
fied AI Agent for Netlist Generation. doi:10.36227/techrxiv.177162431.10627206
Preprint.
[7] Jian Gao, Weidong Cao, Junyi Yang, and Xuan Zhang. 2025. AnalogGenie: A
Generative Engine for Automatic Discovery of Analog Circuit Topologies. In The
Thirteenth International Conference on Learning Representations.
[8] Helmut Graeb and Markus Leibl. 2023. Learning from the Implicit Functional
Hierarchy in an Analog Netlist. In Proceedings of the 2023 ACM International
Symposium on Physical Design (ISPD). Association for Computing Machinery,
93–100. doi:10.1145/3569052.3578921
[9] Chung-Wen Ho, Albert E. Ruehli, and Pierce A. Brennan. 1975. The Modified
Nodal Approach to Network Analysis. IEEE Transactions on Circuits and Systems
22, 6 (1975), 504–509. doi:10.1109/TCS.1975.1084079
[10] Chun-Yen Huang, Hsuan-I Chen, Hao-Wen Ho, Pei-Hsin Kang, Mark Po-Hung
Lin, Wen-Hao Liu, and Haoxing Ren. 2025. Netlistify: Transforming Circuit
Schematics into Netlists with Deep Learning. In Proceedings of the 2025 ACM/IEEE
7th Symposium on Machine Learning for CAD (MLCAD). IEEE, 1–8. doi:10.1109/
MLCAD65511.2025.11189145
[11] Md Touhidul Islam, Sujan Kumar Saha, Farimah Farahmandi, and Mark Tehra-
nipoor. 2026. CircuitFormer: A Circuit Language Model for Analog Topology
Design from Natural Language Prompt. arXiv:2605.05773 [cs.AR]
[12] Carlos E. Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press,
and Karthik Narasimhan. 2024. SWE-bench: Can Language Models Resolve Real-
World GitHub Issues?. In International Conference on Learning Representations.
[13] Kishor Kunal, Tonmoy Dhar, Meghna Madhusudan, Jitesh Poojary, Arvind K.
Sharma, Wenbin Xu, Steven M. Burns, Jiang Hu, Ramesh Harjani, and Sachin S.
Sapatnekar. 2020. GANA: Graph Convolutional Network Based Automated Netlist
Annotation for Analog Circuits. In Proceedings of the 2020 Design, Automation
& Test in Europe Conference & Exhibition (DATE). IEEE, 55–60. doi:10.23919/
DATE48585.2020.9116329
[14] Kishor Kunal, Tonmoy Dhar, Meghna Madhusudan, Jitesh Poojary, Arvind K.
Sharma, Wenbin Xu, Steven M. Burns, Jiang Hu, Ramesh Harjani, and Sachin S.
Sapatnekar. 2023. GNN-Based Hierarchical Annotation for Analog Circuits. IEEE
Transactions on Computer-Aided Design of Integrated Circuits and Systems 42, 9
(2023), 2801–2814. doi:10.1109/TCAD.2023.3236269
[15] Yao Lai, Sungyoung Lee, Guojin Chen, Souradip Poddar, Mengkang Hu, David Z.
Pan, and Ping Luo. 2025. AnalogCoder: Analog Circuit Design via Training-Free
Code Generation. In Proceedings of the AAAI Conference on Artificial Intelligence,
Vol. 39. 379–387. doi:10.1609/aaai.v39i1.32016
[16] Mingjie Liu, Teodor-Dumitru Ene, Robert Kirby, Chris Cheng, Nathaniel Pinckney,
Rongjian Liang, Jonah Alben, Himyanshu Anand, Sanmitra Banerjee, Ismet
Bayraktaroglu, et al. 2023. ChipNeMo: Domain-Adapted LLMs for Chip Design.
arXiv:2311.00176 [cs.CL]
[17] Ryoga Matsuo, Stefan Uhlich, Arun Venkitaraman, Andrea Bonetti, Chia-Yu
Hsieh, Ali Momeni, Lukas Mauch, Augusto Capone, Eisaku Ohbuchi, and
Lorenzo Servadei. 2024. Schemato: An LLM for Netlist-to-Schematic Conversion.
arXiv:2411.13899 [cs.LG]
[18] Laurence W. Nagel. 1975. SPICE2: A Computer Program to Simulate Semiconductor
Circuits. Technical Report UCB/ERL M520. Electronics Research Laboratory, Uni-
versity of California, Berkeley. https://www2.eecs.berkeley.edu/Pubs/TechRpts/
1975/9602.html
[19] Laurence W. Nagel and Donald O. Pederson. 1973. SPICE (Simulation Program
with Integrated Circuit Emphasis). Technical Report UCB/ERL M382. Electronics
Research Laboratory, University of California, Berkeley. https://www2.eecs.
berkeley.edu/Pubs/TechRpts/1973/22871.html
[20] Simon Nau, Jan Krummenauer, and André Zimmermann. 2025. Eval-
uating LLM-based Workflows for Switched-Mode Power Supply Design.
arXiv:2507.10639 [cs.AR]
[21] Jingyu Pan, Guanglei Zhou, Chen-Chia Chang, Isaac Jacobson, Jiang Hu, and
Yiran Chen. 2025. A Survey of Research in Large Language Models for Electronic
Design Automation. ACM Transactions on Design Automation of Electronic Systems
30, 3, Article 34 (2025), 21 pages. doi:10.1145/3715324
[22] Fabrice Salvaire. 2021. PySpice: Simulate Electronic Circuit using Python and the
Ngspice/Xyce Simulators. Software. https://pyspice.fabrice-salvaire.fr/ Accessed
2026-07-28.
[23] Yichen Shi, Ze Zhang, Hongyang Wang, Zhuofu Tao, Zhongyi Li, Bingyu Chen,
Yaxin Wang, Zhiping Yu, Ting-Jung Lin, and Lei He. 2025. AMSbench: A
Comprehensive Benchmark for Evaluating MLLM Capabilities in AMS Circuits.
arXiv:2505.24138 [cs.LG]
[24] Akshit Sinha, Arvindh Arun, Shashwat Goel, Steffen Staab, and Jonas Geiping.
2025. The Illusion of Diminishing Returns: Measuring Long Horizon Execution
in LLMs. arXiv:2509.09677 [cs.AI]
[25] Lejla Skelic, Yan Xu, Matthew Cox, Wenjie Lu, Tao Yu, and Ruonan Han. 2025.
CIRCUIT: A Benchmark for Circuit Interpretation and Reasoning Capabilities of
LLMs. arXiv:2502.07980 [cs.LG]
[26] Deepak Vungarala, Sakila Alam, Arnob Ghosh, and Shaahin Angizi. 2024. SPI-
CEPilot: Navigating SPICE Code Generation and Simulation with AI Guidance.
arXiv:2410.20553 [cs.AR]
[27] Heng Wang, Shangbin Feng, Tianxing He, Zhaoxuan Tan, Xiaochuang Han, and
Yulia Tsvetkov. 2023. Can Language Models Solve Graph Problems in Natural
Language?. In Advances in Neural Information Processing Systems, Vol. 36. 30840–
30861.
[28] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei
Xia, Ed H. Chi, Quoc V. Le, and Denny Zhou. 2022. Chain-of-Thought Prompting
Elicits Reasoning in Large Language Models. In Advances in Neural Information
Processing Systems, Vol. 35. 24824–24837.
[29] Ziming Wei, Zichen Kong, Yuan Wang, David Z. Pan, and Xiyuan Tang. 2025.
TopoSizing: An LLM-aided Framework of Topology-based Understanding and
Sizing for AMS Circuits. arXiv:2509.14169 [cs.LG]
[30] Edwin B. Wilson. 1927. Probable Inference, the Law of Succession, and Statistical
Inference. J. Amer. Statist. Assoc. 22, 158 (1927), 209–212. doi:10.1080/01621459.
1927.10502953
[31] Haohang Xu, Chengjie Liu, Qihang Wang, Wenhao Huang, Yongjian Xu, Weiyu
Chen, Anlan Peng, Zhijun Li, Bo Li, Lei Qi, Jun Yang, Yuan Du, and Li Du. 2025.
Image2Net: Datasets, Benchmark and Hybrid Framework to Convert Analog
Circuit Diagrams into Netlists. arXiv:2508.13157 [cs.AR]

## Page 8

, , Ma et al.
A Additional Results
Table 4: Task-level pass rates for SPICE and PySpice.
Task DS-S DS-Py Qwen-S Qwen-Py
Connectivity edit 0.80 1.00 0.60 0.57
Device add 0.50 0.80 0.50 0.50
Device remove 1.00 1.00 1.00 0.97
Device replace 0.70 0.80 0.80 0.73
Parameter edit 0.93 0.97 0.97 0.93
Rename propagation 0.90 0.93 0.93 0.77
Compound 3-step 0.13 0.50 0.20 0.23
Compound 6-step 0.13 0.23 0.07 0.20
Compound 9-step 0.03 0.23 0.07 0.03
Compound 12-step 0.00 0.10 0.00 0.03
Compound 15-step 0.00 0.07 0.00 0.00
Subckt inline-expand 0.37 0.47 0.13 0.37
Subckt port-swap 0.60 0.73 0.27 0.33
Subckt compound 3-step 0.33 0.10 0.37 0.10
Subckt compound 6-step 0.07 0.03 0.00 0.03
Subckt compound 9-step 0.03 0.00 0.03 0.00
Edit subtotal 0.41 0.50 0.37 0.36
Equivalence judgment 0.50 0.53 0.50 0.60
Device parameter 1.00 1.00 1.00 0.90
Semantic terminal conn. 0.80 0.83 0.23 0.43
Ordered terminal conn. 1.00 1.00 1.00 0.97
Node incidence 0.53 0.60 0.23 0.23
Subckt port list 1.00 1.00 1.00 1.00
Instance port map 0.90 0.90 0.80 0.93
Terminal neighbor inc. 0.07 0.13 0.03 0.00
Recognition subtotal 0.76 0.78 0.61 0.64
Overall 0.52 0.58 0.45 0.45
DS-S: DeepSeek-V4 SPICE; DS-Py: DeepSeek-V4 PySpice; Qwen-S: Qwen-3.6 SPICE; Qwen-Py: Qwen-3.6 PySpice.
Table 5: Task-level pass rates under different prompting
modes.
Task
DeepSeek-V4 Qwen-3.6
Dir. Think CoT Dir. Think CoT
Connectivity edit 0.80 1.00 1.00 0.60 0.93 0.87
Device add 0.50 0.47 0.57 0.50 0.83 0.67
Device remove 1.00 1.00 1.00 1.00 1.00 1.00
Device replace 0.70 0.90 0.83 0.80 0.93 0.93
Parameter edit 0.93 0.97 1.00 0.97 0.97 0.97
Rename propagation 0.90 1.00 1.00 0.93 0.97 1.00
Compound 3-step 0.13 0.57 0.50 0.20 0.73 0.73
Compound 6-step 0.13 0.50 0.53 0.07 0.67 0.63
Compound 9-step 0.03 0.47 0.57 0.07 0.73 0.70
Compound 12-step 0.00 0.37 0.27 0.00 0.63 0.33
Compound 15-step 0.00 0.18 0.14 0.00 0.54 0.32
Subckt inline-expand 0.37 0.97 1.00 0.13 0.83 0.77
Subckt port-swap 0.60 1.00 1.00 0.27 0.93 0.93
Subckt compound 3-step 0.33 0.73 0.63 0.37 0.90 0.63
Subckt compound 6-step 0.07 0.60 0.53 0.00 0.77 0.70
Subckt compound 9-step 0.03 0.63 0.40 0.03 0.63 0.40
Edit subtotal 0.41 0.71 0.69 0.37 0.81 0.73
Equivalence judgment 0.50 0.97 0.93 0.50 0.73 0.70
Device parameter 1.00 1.00 1.00 1.00 1.00 1.00
Semantic terminal conn. 0.80 1.00 0.93 0.23 0.90 0.87
Ordered terminal conn. 1.00 1.00 1.00 1.00 1.00 1.00
Node incidence 0.53 1.00 0.73 0.23 0.93 0.83
Subckt port list 1.00 1.00 1.00 1.00 1.00 0.97
Instance port map 0.90 1.00 0.97 0.80 1.00 1.00
Terminal neighbor inc. 0.07 1.00 0.47 0.03 0.83 0.73
Recognition subtotal 0.76 1.00 0.87 0.61 0.95 0.91
Overall 0.52 0.81 0.75 0.45 0.85 0.78
Dir.: direct prompting; Think: think-mode prompting; CoT: chain-of-thought prompting.
(a) DeepSeek (b) Qwen
(c) DeepSeek, Compound (d) Qwen, Compound
Figure 5: Task-level performance under SPICE and PySpice
representations.
(a) DeepSeek (b) Qwen
(c) DeepSeek, Compound (d) Qwen, Compound
Figure 6: Task-level performance under non-thinking, think-
ing, and CoT prompting.

