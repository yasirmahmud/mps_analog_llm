# LLMs for analog circuit design: literature survey and MPS agent proposal

**Cutoff:** 28 September 2026. **Audience:** MPS leadership with software and AI expertise. **Evidence base:** 63 locally available PDFs in the [catalog](README.md), their [page-marked, LLM-readable text](llm_corpus/README.md), and three supplemental works linked below. This is a curated survey, not a claim that every publication has been found.

## Executive readout

The literature shows a clear shift from asking an LLM to output a circuit directly toward **tool-driven design loops**: the model interprets specifications, chooses bounded actions, reads simulator feedback, and revises a design. The strongest evidence concerns transistor sizing and topology generation for small or medium benchmark circuits. Industrial sign-off, layout effects, power-management-specific workflows, and reproducible comparisons across technology nodes remain much thinner. Published success rates are **not comparable across papers** because circuits, process models, specifications, iteration budgets, and pass rules differ.

For MPS, the first useful product is a **private, simulation-grounded design assistant for one selected power-management block**, with a benchmark and audit trail. A local open-weight model can propose actions and explain trade-offs; deterministic parsers, simulators, and sign-off checks determine whether a design passes. Benchmark the model before fine-tuning it. Use proprietary MPS design history as retrieval material and, only after rights and quality review, as training data.

### Featured paper for an MPS pilot

**[AgenticSizing (Hao et al., 2026)](https://arxiv.org/abs/2609.25873)** is the strongest single paper to feature for the proposed fixed-topology LDO sizing pilot. It tags functional blocks in a netlist, coordinates specialized sizing agents, and uses Spectre feedback. The [released code](https://github.com/aprilaihub/agentic-analog-sizing) provides a concrete workflow to evaluate. In the paper's standalone LDO benchmark, AgenticSizing and Bayesian optimization each found feasible designs in 4/5 trials; successful AgenticSizing runs averaged 79 simulations versus 524 for Bayesian optimization. On an integrated LDO with a bandgap reference and folded-cascode amplifier (55 transistors, 60 variables), AgenticSizing succeeded in 3/5 trials, while each of three tested baselines succeeded in 0/5 within six hours. These are small, paper-specific comparisons and do not establish production readiness.

The published experiments use **GPT-5.2**, so the result does not yet satisfy MPS's private open-weight requirement. Reproduce the same workflow with a locally deployed model, then test held-out MPS LDO designs against the required process, voltage, and temperature corners. Track all-spec pass rate, simulator calls, elapsed time, and engineer corrections against a conventional optimizer. **AnalogAgent** remains the complementary open-weight proof point because it tests Qwen 8B inside a feedback-and-memory workflow, but its reported tasks are less directly aligned with an LDO sizing pilot.

## Scope, method, and counts

The collection includes work on circuit topology (which devices connect), sizing (device values), schematics, physical layout, testbench creation, knowledge/data, and evaluation. Each local PDF is assigned **one primary category** based on its main contribution. The counts therefore add up without double-counting multi-stage systems. The original collection contained one digital gate-level paper, [CircuitMind](https://arxiv.org/abs/2504.14625); it has been moved to the adjacent group because its TuringComplete logic-gate tasks do not test analog IC design. The [two identified works without local PDFs](not_downloaded.md) are excluded from counts. The supplemental works below are also excluded so the denominator stays auditable.

| Primary category | Local papers | What the papers mainly test |
|---|---:|---|
| Sizing and optimization | 18 | Choosing component/device values to meet specifications |
| Topology and schematics | 14 | Generating or interpreting connections and schematics |
| End-to-end workflows | 9 | Linking topology, sizing, and sometimes layout |
| Data, retrieval, and knowledge | 6 | Building reusable circuit data or design memory |
| Models and benchmarks | 4 | Training representations or testing model reliability |
| Layout and physical design | 3 | Layout constraints, I/O design, or tool interaction |
| Verification and testbenches | 1 | Creating simulation tests |
| **Analog-focused research total** | **55** |  |
| Surveys/background | 5 | Context, not primary experiments in this count |
| Adjacent applications | 3 | General circuits, analog in-memory systems, or digital gates |
| **All local PDFs** | **63** |  |

The 55 core papers break down by first catalog date as **14 in 2024, 22 in 2025, and 19 in 2026 through 28 September**. Publication counts reflect this collection's search and access rules; they are not a field-wide bibliometric census. The full title, authors, date, URL, file, checksum, and category for every local item are in [index.csv](index.csv).

### Important work outside the local denominator

- [AMSbench](https://arxiv.org/abs/2505.24138) adds approximately 8,000 multimodal questions across schematic perception, analysis, and design. It reports difficulty on complex reasoning and is essential when designing an MPS evaluation set.
- [SPICEPilot](https://arxiv.org/abs/2410.20553) provides a SPICE code dataset, generation framework, and executable evaluation metrics.
- [Atelier](https://doi.org/10.1109/TCAD.2025.3573228) uses a compact knowledge base and specialized agents for op-amp topology and sizing. The local collection records it as identified but without a local PDF. It is a distinct work from **Artisan**, despite a likely wording error in Artisan's abstract.

## What the research actually establishes

### 1. Topology and schematic generation: promising inside constrained domains

[AnalogCoder](https://arxiv.org/abs/2405.14918) generates Python-based circuit descriptions, then uses checks and simulation feedback; its circuit library makes useful subcircuits reusable. [AnalogXpert](https://arxiv.org/abs/2412.19824) adds a subcircuit library and proofreading, but reports only **23% success on its 30 real examples**, a reminder that a convincing netlist is not automatically a working one. [LaMAGIC](https://arxiv.org/abs/2407.18269), [LaMAGIC2](https://arxiv.org/abs/2506.10235), [AUTOCIRCUIT-RL](https://arxiv.org/abs/2506.03122), and [EXPLORE](https://arxiv.org/abs/2607.13416) learn or search converter topologies in defined component spaces. Their relative gains are meaningful within their benchmarks; they do not demonstrate arbitrary industrial topology synthesis.

[Schemato](https://arxiv.org/abs/2411.13899) and [EEschematic](https://arxiv.org/abs/2510.17002) address human-readable schematic rendering from netlists. [CircuitFormer](https://arxiv.org/abs/2605.05773) studies a circuit-specific representation and smaller model. The evidence favors structured representations, reusable blocks, and executable validation over unconstrained prose-to-netlist generation. For MPS, a library of approved power-management subcircuits is a practical starting point.

### 2. Sizing and optimization: the largest category, with the clearest near-term use

[ADO-LLM](https://arxiv.org/abs/2406.18770) and [LEDRO](https://arxiv.org/abs/2411.12930) combine LLM guidance with Bayesian/trust-region optimization. LEDRO evaluates **22 op-amp topologies across four FinFET nodes** using Llama 3 70B; its result is stronger evidence for transfer than a single-circuit demonstration, although the target family remains op-amps. [AmpAgent](https://arxiv.org/abs/2409.14739) extracts amplifier knowledge from literature before sizing. [TopoSizing](https://arxiv.org/abs/2509.14169), [HeaRT](https://arxiv.org/abs/2511.19669), and [AgenticSizing](https://arxiv.org/abs/2609.25873) explicitly identify functional subblocks before changing parameters. AgenticSizing includes an LDO and reports a largest case of **55 transistors and 60 sizing variables**, but uses GPT-5.2 as its agent backbone; source-code availability alone does not make the method private by default.

[AutoSizer](https://arxiv.org/abs/2602.02849) contributes an open SKY130 sizing benchmark. [AnaFlow](https://arxiv.org/abs/2511.03697) and [EEsizer](https://arxiv.org/abs/2509.25510) automate simulator feedback loops, largely with cloud models. [A Self-Calibrating Framework](https://arxiv.org/abs/2604.07387) generates interpretable analytical equations and calibrates them with simulation. [RFAmpDesigner](https://arxiv.org/abs/2605.10093) extends agent design to RF amplifiers. [ORACLE](https://arxiv.org/abs/2608.04999) keeps multiple objectives explicit instead of compressing them to one score. These are useful patterns for an MPS pilot: circuit-aware parameter grouping, bounded search, explicit trade-offs, and saved best-feasible designs.

### 3. Multi-stage design: coordination helps, but full sign-off is rare

[LADAC](https://doi.org/10.36227/techrxiv.170473941.10097233) and [Artisan](https://doi.org/10.1145/3649329.3655903) are early knowledge-guided systems. [AnalogCoder-Pro](https://arxiv.org/abs/2508.02518), [AnalogSAGE](https://arxiv.org/abs/2512.22435), [AnalogAgent](https://arxiv.org/abs/2603.23910), and [AaLLM](https://arxiv.org/abs/2608.13472) integrate generation, feedback, optimization, and memory. AnalogAgent reports **72.1% aggregate Pass@1 with a Qwen 8B backbone** in its tasks, compared with higher results from closed models within that paper; this supports testing small local models with a stronger workflow, not assuming equivalent performance on MPS designs. [AnalogMaster](https://arxiv.org/abs/2604.20916) and [PANDA](https://arxiv.org/abs/2606.15052) connect front-end design to layout, but published examples remain narrower than an MPS production flow. [An LLM-assisted Analog IC Design Tool](https://doi.org/10.1109/SMACD65553.2025.11091899) uses Llama 3.1 8B to select from a circuit library and an ANN for sizing.

### 4. Data, benchmarks, layout, and verification: essential but understudied

[Masala-CHAI](https://arxiv.org/abs/2411.14299), [AMSnet-KG](https://arxiv.org/abs/2411.13560), and [Dataset Construction for Training LLM to Learn Analog Circuit Knowledge](https://arxiv.org/abs/2508.10409) show why structured, quality-checked training examples matter. [DocEDA](https://arxiv.org/abs/2412.05301), [MuaLLM](https://arxiv.org/abs/2508.08137), [ADO-KT](https://doi.org/10.1109/ICCAD66269.2025.11240770), and [AnalogRetriever](https://arxiv.org/abs/2604.23195) mine documents, transfer knowledge, or retrieve prior designs. Their value depends on provenance and access controls when internal MPS designs are included.

[ACDC](https://arxiv.org/abs/2512.09199), [ACR-Bench/Agent](https://doi.org/10.48130/ker-0026-0012), and [NetlistBench](https://arxiv.org/abs/2608.12197) reveal representation sensitivity and unreliable SPICE operations even when high-level explanations sound plausible. [LayoutCopilot](https://arxiv.org/abs/2406.18873), [LLANA](https://arxiv.org/abs/2406.05250), and [AMS-IO-Bench/Agent](https://arxiv.org/abs/2512.21613) cover parts of physical design. [AnalogTester](https://arxiv.org/abs/2507.09965) is the sole primary testbench paper in the local core set. This low count should not be read as verification being unimportant; it is a research gap.

[SABLE](https://arxiv.org/abs/2607.03701) is unusually relevant to industrial privacy and tool boundaries. It uses an explicit threat model, sanitized outputs, allowlisted Cadence operations, machine-checked stop rules, and PVT checks. Its cloud-provider boundary is **not a formal non-interference proof** and differs from MPS's proposed self-hosted privacy requirement. In two tasks and 11 model checkpoints, 4/11 passed the harder op-amp task within 15 iterations. This demonstrates both the value of tool discipline and the current reliability limit.

## Open-weight models and privacy

“Open-source framework,” “open dataset,” and “open-weight model” describe different things. A published agent may release code while depending on a closed API. Likewise, calling a hosted API for an open-weight model does not keep MPS inputs private. MPS needs a **locally deployed checkpoint**, local retrieval, and local simulation/log storage. Model and dataset licenses still need individual review.

| Evidence in this literature | Role of the open model | Practical reading for MPS |
|---|---|---|
| [LEDRO](https://arxiv.org/abs/2411.12930) | Llama 3 70B guides design-space reduction | Open weights can guide a conventional optimizer; test on MPS circuits and hardware budget. |
| [Schemato](https://arxiv.org/abs/2411.13899) | Fine-tuned Llama 3.1 8B converts netlists to schematics | A narrow task can justify a smaller specialist model. |
| [AnalogCoder](https://arxiv.org/abs/2405.14918), [Masala-CHAI](https://arxiv.org/abs/2411.14299) | Llama, CodeLlama, DeepSeek variants evaluated beside GPT | Simulator checks remain necessary regardless of model family. |
| [AnalogAgent](https://arxiv.org/abs/2603.23910) | Qwen 8B tested inside a memory-and-feedback workflow | Agent design can materially improve a compact model; paper metrics do not transfer directly. |
| [ACDC](https://arxiv.org/abs/2512.09199), [NetlistBench](https://arxiv.org/abs/2608.12197) | Mistral/GPT-oss and Qwen evaluated for structured circuit tasks | Include format robustness and netlist manipulation in model selection. |
| [AgenticSizing](https://arxiv.org/abs/2609.25873), [AnaFlow](https://arxiv.org/abs/2511.03697) | Released workflow or benchmark; main agent uses GPT/Gemini | Reproduce the workflow with a local backbone before citing it as private-ready. |

**Candidate checkpoints for a controlled MPS bake-off:** [Qwen3-32B](https://huggingface.co/Qwen/Qwen3-32B) and [gpt-oss-20b](https://openai.com/index/gpt-oss-model-card/) are downloadable under Apache 2.0; [DeepSeek-R1-Distill-Qwen-32B](https://huggingface.co/deepseek-ai/DeepSeek-R1) is another reasoning candidate whose base-model lineage and terms should be checked. [Llama 3.3 70B](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct) is self-hostable but has a **custom community license**, so it should not be grouped with Apache/MIT-licensed models. These are test candidates, not a declaration of a winner or a hardware sizing recommendation. Compare exact checkpoint IDs, quantization, prompts, tool budgets, and inference costs on the same holdout tasks. A vision model can be added only if raw schematic images provide value beyond parsed netlists.

## Gaps that matter most to MPS

1. **Industrial benchmark gap.** Public suites test different tasks and pass rules. Few include a complete MPS sequence from requirements through simulation, layout effects, PVT corners, and engineer sign-off. Even AMSbench's broad question coverage does not substitute for executable design acceptance.
2. **Power-management transfer gap.** Converter topology papers, a PCB SMPS workflow, and LDO sizing papers touch relevant functions, but they differ in abstraction, fabrication context, and acceptance criteria. Their scores cannot establish readiness for MPS products. [PE-GPT](https://vbn.aau.dk/en/publications/pe-gpt-a-new-paradigm-for-power-electronics-design/) is useful adjacent power-electronics evidence outside the local count.
3. **Reliability gap.** Valid syntax, simulator convergence, all-spec feasibility, corner robustness, and post-layout sign-off are distinct. Many reports stop before the last two. SABLE is a useful exception, but spans only two tasks.
4. **Data and leakage gap.** Circuit families, process nodes, repeated topologies, and related layouts can leak from train to test. Papers rarely provide a split that proves transfer to unseen MPS blocks or nodes. Public PDF text also does not automatically confer training rights.
5. **Privacy and integration gap.** Many high-performing systems use closed APIs. Few document the full data path, tool permissions, licensing, audit logs, rollback, and safe failure behavior needed for a private industrial agent.

## Proposed MPS agent and evaluation plan

**Pilot scope:** choose one MPS-approved circuit family with repeatable SPICE testbenches, preferably an LDO or a buck-converter control block. Keep topology fixed in the first sizing pilot; add topology proposals after the agent proves reliable. An MPS analog engineer should define the target specifications and acceptance tests.

| Stage | Build | Exit gate |
|---|---|---|
| 1. Data and benchmark | Ingest approved specifications, netlists, testbenches, simulation traces, and design notes into an access-controlled store. Create a canonical spec schema and blinded held-out tasks split by circuit family and, where possible, process node. | Reproducible baseline results; every task has a deterministic pass/fail test and provenance. |
| 2. Private baseline | Run local Qwen3-32B and gpt-oss-20b with the same prompts and tool budget. Compare with a conventional optimizer and a human-assisted baseline. | Report valid-action rate, all-spec pass rate, simulation calls, elapsed time, human corrections, and PVT/post-layout checks where available. |
| 3. Bounded agent | Add retrieval of approved prior designs; parse netlists into functional blocks; use typed, allowlisted actions to change parameters; run simulator/testbench tools; preserve best feasible state and explain each step. | Agent beats the strongest baseline on predeclared holdout metrics, with zero unauthorized tool actions. |
| 4. Specialization and shadow use | Use failure analysis to decide between better retrieval, parameter-efficient fine-tuning, or a circuit-specific parser/model. Test under engineers' review before any design writeback. | Measured gain on unseen tasks, stable corner performance, complete audit trail, and engineer acceptance. |

The agent loop should be: **specification → validated circuit representation → retrieved MPS examples → local planning model → typed EDA action → simulation/verification → state and evidence log → engineer review**. The model proposes; code checks syntax and scope; simulation establishes electrical behavior. Tool access should be least-privilege, with sandboxed execution, explicit budgets, error handling, and rollback. The private deployment boundary includes model inference, vector search, prompts, outputs, PDK references, netlists, and logs.

**Evaluation contract:** keep a frozen test set and publish both per-task outcomes and aggregate metrics internally. At minimum record netlist parse/compile rate, successful simulation rate, all-spec pass rate, worst-corner pass rate, number of simulator calls to first feasible solution, wall-clock time, and designer edits. Use identical starting states and budgets for each model/agent. Report failures, not only successful examples. Separate pre-layout from post-layout scores. Treat a design as passing only when every required hard constraint passes under the agreed MPS testbench.

## Reuse and evidence limits

The [LLM corpus](llm_corpus/README.md) provides Markdown text with page markers, a provenance manifest, and retrieval chunks for all 63 PDFs. Its text extraction is useful for search and summarization, but paper figures, equations, and two-column reading order can remain imperfect. Check consequential numbers against the PDF and original source. Keep the PDFs and extracted text inside MPS's authorized environment; review document and dataset rights before model training or redistribution. The survey's recommendations are inferences from the papers, not validated MPS results.
