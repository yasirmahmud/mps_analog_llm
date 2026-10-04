---
title: "CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt"
authors: "Islam, Md Touhidul; Saha, Sujan Kumar; Farahmandi, Farimah; Tehranipoor, Mark"
date: "2026/05/07"
category: "topology_schematic"
source_url: "https://arxiv.org/abs/2605.05773"
pdf: "papers/2026_circuitformer-a-circuit-language-model-for-analog-topology-design-from-natu_2605-05773.pdf"
sha256: "a60ba5f7e5a0fe9020eef26a3d5b290d15a217bf5edc17e70082b360f210c4e6"
pages_extracted: 22
---

# CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

CircuitFormer: A Circuit Language Model for Analog Topology Design from
Natural Language Prompt
Md Touhidul Islam 1
Sujan Kumar Saha 1
Farimah Farahmandi 1
Mark Tehranipoor 1
1
University of Florida
mdtouhidul.islam@ufl.edu, sujansaha@ufl.edu
farimah@ece.ufl.edu, tehranipoor@ufl.edu
Abstract
Automating analog circuit design remains a
longstanding challenge in Electronic Design
Automation (EDA). While Transformer-based
Large Language Models (LLMs) have revolution-
ized software code generation, their application
to analog hardware design is hindered by two
critical limitations: (i) the scarcity of analog
design datasets containing natural language
description of a design and its corresponding
netlist, and (ii) the inefficiency of general-purpose
tokenizers (e.g., Byte Pair Encoding (BPE)) in
capturing the inherent graph structure of circuits.
To bridge this gap, first, we curate the largest
annotated dataset of analog circuit netlists to
date, comprising 31,341 netlist-natural language
description pairs across all major circuit classes.
Furthermore, we propose Circuit Tokenizer
(CKT), a novel circuit graph tokenizer designed
to encode netlist connectivity by explicitly mining
frequent subcircuits. In terms of scalability,
CKT overcomes the bottleneck of prior circuit
graph serialization methods where vocabulary
size scales linearly with maximum number of
components in the dataset, nmax, (O(nmax));
instead, CKT decouples vocabulary growth from
circuit complexity, achieving a constant O(1)
complexity. Empirically, CKT outperforms
standard BPE on circuit topology representation,
reducing sequence length by 57% and achieving a
2.3× superior compression ratio using a compact,
fixed vocabulary of size 512. Leveraging this
optimized tokenization, we train a circuit-specific
language model, CircuitFormer, a 511M
parameter encoder-decoder transformer. Our
model achieves 100% syntactic correctness
and an 83% functional success rate across all
major analog circuit categories, outperforming
state-of-the-art open-source LLMs by 10% and
14%, respectively, while requiring 240× fewer
parameters. The dataset is publicly available at
https://huggingface.co/datasets/
touhid314/cktformer-dataset.
1. Introduction
The recent success of Large Language Models (LLMs) has
transformed software development. Yet analog circuit de-
sign remains largely manual and driven by expert heuristics.
This analog automation gap persists even though analog
circuits are essential, serving as the interface between the
physical world and digital computation.
Progress in applying generative AI to analog design is lim-
ited by two fundamental challenges. First, high-quality
analog circuit data is scarce: unlike software code, most
analog designs are buried in textbooks, research papers, or
proprietary PDKs. Existing public datasets(Bhandari et al.,
2025)(Shi et al., 2025) are small, lack natural language
descriptions, or cover only narrow classes of circuits. Sec-
ond, there is a severe text–graph mismatch. While standard
LLMs operate on linear text, a circuit netlist is simply a
textual serialization of an underlying graph. Conventional
tokenizers such as BPE are sensitive to component order,
even though circuit behavior is invariant to such ordering,
leading models to learn superficial text patterns rather than
true circuit structure.
Prior work has attempted to address this problem using
LLMs(Lai et al., 2025a)(Liu et al., 2024), graph neural
networks(Dong et al., 2023), or domain-specific genera-
tors(Gao et al., 2025a)(Chang et al., 2024), but each ap-
proach has clear limitations: LLMs hallucinate invalid con-
nectivity, graph-based models struggle to align with free-
form language, and existing circuit-specific transformers
lack scale, control, or natural language conditioning. These
limitations motivate a new modeling paradigm.
In this work, we treat circuits as a distinct modality, analo-
gous to images or video in multimodal learning. We intro-
1
arXiv:2605.05773v1
[cs.AI]
7
May
2026

## Page 2

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
Table 1. Comparison of CircuitFormer with Existing Datasets and Domain-Specific Transformer Models for Analog Design
DATASET STATISTICS MODEL ARCHITECTURE
METHOD SIZE
DESC.
ANNOT.
SIM.
VALID.
COMP.
TYPES
*
AUTO-
MATED INPUT PARAMS
MASALACHAI (BHANDARI ET AL., 2025) 7,500 YES NO 9 YES — Dataset Only —
AMSNET 2.0 (SHI ET AL., 2025) 2,686 NO YES † NO — Dataset Only —
ANALOGGENIE (GAO ET AL., 2025A) 3,350 NO YES 8 NO NONE 11.8M
ANALOGGENIE-LITE (GAO ET AL., 2025B) 3,350 NO YES 8 NO NONE 11.8M
LAMAGIC (CHANG ET AL., 2024) 2,000 NO YES 6 YES VECTOR 248M
LAMAGIC 2 (CHANG ET AL., 2025) 2,000 NO YES 6 YES VECTOR 248M
CIRCUITFORMER (OURS) 31,341 YES YES 20 YES
NATURAL
LANGUAGE 511M
*
Comp. Types: Fundamental SPICE elements (e.g., V, R, L, etc.). †Analog Mixed Signal specific
duce CircuitFormer, a 511M-parameter encoder–decoder
Transformer designed for translating natural language spec-
ifications directly into analog circuit topologies. Circuit-
Former is powered by a novel graph-mining Circuit Tok-
enizer (CKT) that operates on topological subgraphs rather
than text strings, allowing the model to focus on structural
validity instead of serialization artifacts.
Our contributions are threefold:
1. We release the largest publicly available annotated ana-
log circuit dataset, consisting of 31,341 verified SPICE
netlists across all major analog circuit classes.
2. We propose CKT, a domain-specific tokenizer for cir-
cuit graphs that exploits graph permutation invariance,
reduces sequence length by 57% compared to general-
purpose tokenizers, and achieves O(1) vocabulary scal-
ing.
3. We train a circuit-specific transformer, CircuitFormer,
which achieves 100% syntactic validity and an 83% func-
tional success rate on the CircuitBench-100 benchmark,
while being 240× smaller than open-source SOTA LLMs
such as GPT-OSS.
We will release the full dataset and source code upon accep-
tance.
2. Preliminaries and Related Work
Recent advancements in AI-driven analog circuit design
can be broadly categorized into three distinct paradigms: 1)
LLM-Based Approaches (Prompt Engineering & LoRA), 2)
Graph Neural Networks, 3) Domain-Specific Transformers.
1. LLM-Based Approaches (Prompt Engineering &
LoRA): The first paradigm applies off-the-shelf LLMs
(e.g., GPT-4, Qwen, DeepSeek) to netlist generation via
prompt engineering or parameter-efficient fine-tuning, as in
AnalogCoder(Lai et al., 2025a), AnalogCoder-Pro(Lai et al.,
2025b), Artisan(Chen et al., 2024), AnalogXpert(Zhang
et al., 2025) and AmpAgent(Liu et al., 2024). Despite strong
reasoning ability, these models treat netlists as linear text
rather than graphs, leading to a fundamental text–graph mis-
match. This results in connectivity hallucinations, such as
invalid or floating nodes that violate circuit laws and fail
simulation, and (ii) inefficient tokenization, where order-
sensitive tokenizers like BPE(Sennrich et al., 2016) learn
redundant representations for permutation-invariant circuits,
wasting model capacity on serialization artifacts instead of
structure.
2. Graph Neural Networks (GNNs): Graph-based ap-
proaches such as CktGNN(Dong et al., 2023) operate di-
rectly on circuit topology, naturally enforcing permutation
invariance and structural validity. However, they struggle
with multimodal alignment: mapping free-form natural lan-
guage prompts to structured graph embeddings remains
challenging, limiting their use as flexible, language-driven
design agents.
3. Domain-Specific Transformers: Recent Transformers
trained specifically on circuit netlists, including AnalogGe-
nie(Gao et al., 2025a) and LaMagic(Chang et al., 2024),
move closer to structure-aware modeling but face key limi-
tations. AnalogGenie is an unconditional generator without
user-driven control, while LaMagic relies on rigid, non-
natural conditioning formats. Moreover, both are trained
on limited-scale datasets, restricting their ability to gener-
alize to complex, real-world analog topologies. AnalogGe-
nie introduces a circuit-specific graph serialization scheme;
however, its vocabulary size grows with the maximum num-
ber of components in a circuit, which limits scalability and
efficiency at larger design scales.
Table 1 provides a comprehensive comparison of recent ana-
log design datasets and domain-specific transformer models.
2

## Page 3

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
V #2
R
D
0
1 2
3
4
V #2
0
1 [V, 0, 1,
#2, 1, 0]
[ V, 0, 1,
#2, 1, 2, 4, 0,
D, 2, 3,
R, 3, 4]
V
R
L
C
R
D
R
V L
C
Normalized
Circuit Dataset
V
C
R
L
Circuit Dataset
as Graph
V
C
R
L
R
D
Subcircuit Mining (gRePair) Compressed
Graph
CKT
Tokens
R
L
C
R L
rule #1
arity=2
rule #2
arity=4
(#1 + C)
C
V
R
L
V
C
R
L
R
D
Figure 1. Subcircuit mining and tokenization process.
2.1. Circuit as a Distinct Modality
We argue that circuit generation is fundamentally different
from code generation: while code is sequential and order-
dependent, circuits are physical graphs where component or-
dering is irrelevant. Accordingly, circuits should be treated
as a distinct modality within multimodal learning, analo-
gous to images or video. CircuitFormer operationalizes this
view by decoupling topological representation from textual
serialization, enabling translation from high-level natural
language intent to physically valid, graph-structured circuit
designs.
3. Circuit Tokenizer (CKT)
The CKT tokenizer is a domain-specific tokenization frame-
work designed to represent electronic circuit topologies as
compact, invariant sequences suitable for transformer-based
architectures. Unlike standard text-based methods such
as Byte Pair Encoding (BPE)(Sennrich et al., 2016), and
SentencePiece(Kudo & Richardson, 2018) which process
netlists as raw strings, CKT operates directly on the graphi-
cal structure of the circuit. It leverages the Graph Re-Pair
(gRePair) algorithm (Maneth & Peternek, 2018) to identify
and compress recurring topological substructures.
3.1. The Tokenization Algorithm
The CKT algorithm follows a two-stage pipeline: (1) Nor-
malization, (2) Subcircuit Mining.
3.1.1. NORMALIZATION AND INSTANCE
ANONYMIZATION
The objective of normalization is to distill the netlist into a
canonical “topology-only” representation that retains full
electrical connectivity while discarding variable metadata.
Canonicalization: The raw SPICE netlist is first sanitized
by removing comments, simulator directives (e.g., .model,
.tran), and merging continuation lines. Node identifiers
are remapped to a contiguous integer sequence starting from
0, with the global ground (e.g., 0 or gnd) always mapped
to node 0 to preserve topological consistency.
Instance Anonymization: We discard instance names and
retain only the device type prefix. For example, instances
R1, R2, and RCC are all mapped to a single token type ⟨R⟩.
This reduces the base vocabulary to the sum of the number
of supported SPICE primitives and the maximum number
of nodes in a circuit in the dataset. However, a critical chal-
lenge of representing a circuit netlist without element names
arises with components that reference other devices by name.
In SPICE, Current-Controlled Voltage Sources (CCVS, pre-
fix H), Current-Controlled Current Sources (CCCS, prefix
F), and Mutual Inductors (prefix K) define their dependen-
cies by referencing the name of a controlling element (typ-
ically a voltage source, e.g., F1 n1 n2 Vsens). Since
our normalization process strips instance names (reducing
Vsens to ⟨V ⟩), the unique reference to the controlling
branch is lost. Furthermore, a branch cannot be uniquely
identified solely by its two endpoint nodes if multiple com-
ponents are connected in parallel between them. However a
branch can be uniquely identified with 3 nodes in all cases.
Hence, we introduce a Dummy Node Insertion strategy:
1. We identify any branch that serves as a controlling refer-
ence.
2. We insert a virtual “dummy” node into that branch, split-
ting the original edge.
3. The name-based reference in the dependent source is
replaced with a topological reference to this new 3-node
sequence (start node, dummy node, end node).
This transformation allows the controlling relationship to
be encoded purely through graph connectivity, allowing
CKT to represent a circuit without component names and
hence reducing the vocabulary space. The overall process
after normalization until CKT tokens creation is depicted in
Figure 1.
3

## Page 4

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
3.1.2. SUBCIRCUIT MINING VIA GRAPH RE-PAIR
CKT employs an iterative graph compression algorithm
adapted from Graph Re-Pair (gRePair) (Maneth & Peternek,
2018) to induce a context-free graph grammar by repeatedly
collapsing frequent topological substructures into single
macro tokens.
Training begins by aggregating the entire netlist corpus
into a single disjoint union graph, Gtrain =
F
c∈D Gc, with
unique node prefixes preserving circuit boundaries; this
enables global frequency estimation of recurring patterns
across the dataset. The algorithm then enumerates all valid
digrams, defined as pairs of adjacent circuit elements shar-
ing at least one node, while explicitly encoding their connec-
tion topology (shared-node mask) to distinguish structures
such as series and parallel configurations.
Selection via Minimum Description Length (MDL)
Gain: Unlike frequency-based tokenizers such as BPE
or AnalogGenie (Gao et al., 2025a), CKT selects merge
candidates using a gain-based criterion grounded in the
Minimum Description Length (MDL) principle. The util-
ity of merging a digram d is defined by its net reduction
in representation complexity rather than raw occurrence
count. Specifically, the compression gain is computed as
Gain(d) = (Sd − 1) × Fd − Sd, where Sd is the total num-
ber of primitive devices spanned by the digram (including
primitives within existing macros), and Fd is the number
of non-overlapping occurrences in the global graph. The
first term captures the savings from removing primitives,
while the second accounts for the cost of introducing a new
grammar rule. A merge is accepted only if Gain(d) > 0,
preventing the vocabulary from being dominated by high-
frequency but low-impact patterns.
To ensure efficiency and learnability, we impose two con-
straints during mining: (i) a minimum frequency threshold
Fmin to filter rare, non-generalizable patterns, and (ii) a
maximum arity Amax that limits the number of external
connections of any macro, preventing the creation of overly
complex tokens that are difficult to predict or reconstruct.
Rule Creation and Graph Rewriting: After selecting
the optimal digram d∗
, CKT induces a new grammar rule
by introducing a non-terminal macro token (e.g., #1, #2,
...) that represents the corresponding substructure. The
algorithm identifies the external nodes of the digram, those
connecting it to the rest of the graph, and defines them as the
ports of the macro, preserving the subcircuit’s interface. All
subgraphs isomorphic to d∗
in Gtrain are then replaced with
this single macro edge, compressing the graph and hierarchi-
cally encoding the topology. This process iterates through
digram collection, MDL-based selection, and replacement
until the target vocabulary size |V| is reached or no further
positive compression gain is possible.
Algorithm 1 illustrates the complete procedure described
above.
3.2. Complexity Analysis of Vocabulary Size
A critical advantage of CKT is that, given a maximum num-
ber of nodes allowed, it has O(1) vocabulary complexity
with respect to circuit size in the dataset, resolving a major
scalability bottleneck present in prior work. Approaches
like AnalogGenie (Gao et al., 2025a) and AnalogGenie-Lite
(Gao et al., 2025b) rely on instance-specific tokenization,
where the vocabulary must explicitly include distinct tokens
for every unique identifier (e.g., R1, . . . , Rnmax
).
Algorithm 1 Circuit Tokenizer (CKT) Training Procedure
Input: Netlist Corpus D, Target Vocabulary Size Vmax,
Min Frequency Fmin, Max Arity Amax, Min Gain Gmin
Output: Grammar Rules R, Tokenized Graphs
{Phase 1: Normalization & Graph Construction}
Gtrain ← ∅
for all netlist N ∈ D do
N′
← NORMALIZE(N) {Remove comments, stan-
dardize names}
N′′
← ADDDUMMYNODES(N′
) {Handle branch ref-
erences (Sec 3)}
Gtrain ← Gtrain ∪ TOGRAPH(N′′
) {Disjoint union}
end for
{Phase 2: Subcircuit Mining (gRePair)}
R ← Initial Primitive Tokens
while |R| < Vmax do
M ← ENUMERATEDIGRAMS(Gtrain)
d∗
← None, MaxGain ← 0
for all digram d ∈ M do
Sd ← GETSPAN(d)
Fd ← COUNTFREQUENCY(d, Gtrain)
Ad ← GETARITY(d) {Number of external ports}
if Fd < Fmin or Ad > Amax then
continue {Skip invalid candidates}
end if
Gain ← (Sd − 1) × Fd − Sd {MDL Objective}
if Gain > MaxGain then
d∗
← d, MaxGain ← Gain
end if
end for
if MaxGain < Gmin then
break {Gain threshold not met}
end if
rnew ← CREATERULE(d∗
)
Gtrain ← REPLACEOCCURRENCES(Gtrain, d∗
, rnew)
R ← R ∪ {rnew}
end while
return R
4

## Page 5

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
YOLO
No
Circuit
Image
VLM
Netlist
Description &
Names
SPICE
Simulator
Simulated
Without
Error?
Add (Netlist,
Description, Names)
to Dataset
PDF (Book,
Paper etc.)
Source
Page
Yes
No
Is this a circuit
schematic?
Small
VLM
Is this a circuit
schematic?
Small
VLM
Yes
Drop
Drop
Figure
PDF Page
as Image
Extracted
Figures from
Image
Figure 2. The fully automated dataset making pipeline.
Consequently, their vocabulary size |V| scales linearly as
O(nmax), where nmax is the maximum component count
in the dataset, leading to vocabulary explosion and an in-
ability to generalize to circuits larger than those seen during
training. In contrast, by decoupling topology from instance
naming (mapping all {R1, . . . , Rk} → ⟨R⟩) and defining a
fixed limit for learned macro-rules, CKT maintains a con-
stant vocabulary size for a maximum number of nodes al-
lowed.
3.3. Detokenization
The detokenization process reconstructs the full circuit
topology by recursively expanding CKT macro tokens into
their underlying primitive components and resolving node
connectivity. Because grammar rules explicitly encode
shared-node masks and external ports, this inverse trans-
formation is lossless. A detailed description of the detok-
enization algorithm is provided in Appendix B.
3.4. Line Shuffle Augmentation
Because CKT identifies components based on their graph-
domain connections rather than their position in the text,
we can randomly shuffle the lines without affecting how
well the tokenizer compresses the circuit. This allows us
to effectively do line shuffle augmentation during training -
we randomly shuffle the lines every time the model sees an
example. A circuit with n lines has n! valid permutations,
meaning there are millions of ways to write the exact same
circuit. This prevents the model from memorizing a specific
text order and forces it to learn the actual circuit topology.
4. The CircuitFormer Dataset
To enable the training of large-scale generative models for
analog design, we curated the largest dataset of analog cir-
cuit topologies to date. Our dataset comprises 31,341 unique
SPICE netlists. Each netlist is paired with a natural lan-
guage description, a list of possible names, and the original
schematic image. The data was sourced from 62 prominent
textbooks and reference volumes in analog integrated circuit
design. Every netlist in the dataset has been verified via
Ngspice simulation to ensure syntactical validity.
Unlike most previous methods, our approach is fully au-
tomated. It accurately extracts netlists from diverse PDF
sources, including textbooks and research papers. This scal-
able design allows to easily expand the dataset. Furthermore,
the system automatically assigns circuits with their correct
names and descriptions, removing the need for manual an-
notation.
4.1. Automated Dataset Creation Pipeline
Due to the scarcity of open-source, high-quality SPICE
netlists online, we identified textbooks as the optimal source
for high-fidelity circuit data. However, converting static
PDF pages into simulatable code requires a robust extraction
pipeline. Our automated workflow consists of three stages:
1. Visual Extraction and Filtering: We first rasterize
the textbook PDFs into high-resolution images. To iso-
late circuit diagrams from text and other figures, we em-
ploy the DocLayout-YOLO(Zhao et al., 2024) object de-
tection model. Extracted figures are then passed through a
lightweight Vision-Language Model (VLM) filter to discard
non-circuit images (e.g., block diagrams, signal plots).
2. Context-Aware Netlist Parsing: We utilize Gemini 3 to
parse the identified schematic images into SPICE netlists.
Along with the cropped schematic image, we provide the
VLM with the full source page containing the figure. This
contextual grounding allows the model to accurately extract
the circuit’s functional description and canonical name from
the caption and surrounding text, and to resolve ambiguous
component values or model parameters that are defined in
the text but omitted from the schematic.
3. Simulation-Based Verification: The raw netlists gener-
ated by the VLM are sanitized and executed in the Ngspice
simulator. Any netlist that fails to converge or produces
syntax errors is automatically discarded. This rigorous filter-
ing ensures that our dataset contains only electrically valid,
syntactically correct circuits.
Figure 2 shows the full dataset making process.
5

## Page 6

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
4.2. Can Vision Language Models accurately Parse
Netlists from Circuit Schematic Images?
Our adoption of VLMs for netlist extraction is grounded in
the observation that very recent state-of-the-art models ex-
hibit remarkable proficiency in translating circuit schematic
images into valid SPICE netlists. To validate the feasibility
of this approach, we assessed the capabilities of these mod-
els in converting circuit schematic images into structured
netlists. Appendix C.1 provides detailed experimental evi-
dence demonstrating that Gemini 3 performs this task with
notably high accuracy and reliability.
To rigorously verify the quality of the dataset generated via
this approach, we performed a manual validity audit on the
final dataset. We randomly sampled 50 instances from the
final dataset and manually inspected the VLM-generated
netlists against the source images. As shown in Table 2,
CircuitFormer (utilizing the Gemini 3-generated ground
truth) achieved a 95% perfect image-to-netlist performance.
The single failure case involved a minor connectivity error
affecting only one element, resulting in an effective element-
wise accuracy of 99.25%.
4.3. Dataset Statistics
The dataset covers a comprehensive range of 20 major
SPICE element types, excluding only the rare A, O, P, U,
W, and Y elements. In total, it comprises 328,565 SPICE
components. The complexity of the circuits varies widely,
ranging from single-component primitives to large systems
with up to 139 components. Detailed statistical distributions
of the dataset are provided in Appendix C.4.
Table 2. Manual validity audit of random samples from the dataset
(N = 50).
METHOD NETLIST ACCURACY ELEMENT ACCURACY
MASALA CHAI 6.0% 36.92%
CIRCUITFORMER (OURS) 95.0% 99.25%
5. Results
5.1. Experimental Setup
The “CircuitBench-100” Test Set: Due to the absence
of standardized benchmarks for text-to-netlist generation,
we curated CircuitBench-100, a novel test set consisting
of 100 natural language prompts. The set spans diverse
topologies including single and multi-stage amplifiers, ac-
tive/passive filters, oscillators, and power management ICs
(PMICs). Each entry includes a high-level design prompt
and a ground-truth netlist for reference. Detailed statistics
on the topological diversity of this benchmark are provided
in Appendix D.
Dataset and Preprocessing: From the curated corpus of
31,341 netlists, we filtered the training set to exclude netlists
Decoder with
Cross
Attention
Prompt
Design a current
mirror with
MOSFET
CKT Tokens
CKT Detokenizer
M1 2 2 0 0 NMOS\n V1 1 0 5 ...
Circuit Topology Netlist
Encoder
(sBERT)
Figure 3. The Overall Transformer architecture used.
with subcircuit (‘X’) calls. This ensures that the model
learns fundamental device physics rather than hierarchical
abstractions. The resulting dataset of 18,011 valid netlists
was split into a 94% training set and a 6% validation set
(≈ 1, 000 samples). To prevent overfitting and enforce topo-
logical invariance, we applied the line-shuffling augmenta-
tion technique described in Section 3.4 during training.
Training Setup: CircuitFormer is a hybrid en-
coder–decoder Transformer that uses a 110M parameter
MPNet encoder (Song et al., 2020) to capture the seman-
tic meaning of natural language prompts and a GPT-2-
based decoder(Radford et al., 2019) extended with cross-
attention layers. We use 24-layers with dmodel = 1020 and
nheads = 12 for the decoder. The encoder uses a wordpiece
tokenizer while the decoder uses the CKT tokenizer. The
overall model architecture is presented in Figure 3. Training
is done in three stages: (1) pre-train the decoder on netlists
to learn circuit syntax(100 epochs, 5 × 10−5
learning rate),
(2) align the encoder and decoder by training only the cross-
attention layers for 25 epochs, and (3) fine-tune the full
model end-to-end to jointly optimize semantic understand-
ing and topology generation (75 epochs, 2 × 10−5
learning
rate). AdamW optimizer with a 10% linear warmup was
used. For inference, we employed beam search with a beam
width of k = 4 to ensure generation consistency. All experi-
ments were conducted on a single NVIDIA H100 GPU with
96GB VRAM.
We apply line-shuffle augmentation (Section 3.4) during
training to encourage the model to learn the underlying
circuit graph structure rather than relying on a fixed textual
ordering.
Appendix E contains details about the model architecture
and the training strategy.
Baselines: We compare CircuitFormer against state-of-the-
art Open-Source Large Language Models (LLMs). Domain-
specific transformers like AnalogGenie were excluded from
direct comparison as they lack natural language conditioning
capabilities. Consequently, open-source LLMs represent the
6

## Page 7

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
only viable baseline for diverse, prompt-based analog gener-
ation. We evaluate: GPT-OSS (120B)(OpenAI et al., 2025),
Llama 3.3 (70B)(Grattafiori et al., 2024), Mistral Large 2
(123B)(Mistral AI Team, 2024), Gemma 3 (27B)(Gemma
Team et al., 2025), and DeepSeek-V3.2 (685B)(DeepSeek-
AI et al., 2025).
Circuit Generation Evaluation Metrics:
1. Success Rate: Since valid analog circuit designs are
non-unique, traditional metrics like exact string match-
ing or graph isomorphism are insufficient for evaluating
functional correctness. Consequently, we rely on manual
human expert evaluation to determine whether a gen-
erated topology satisfies the design prompt, using the
ground-truth netlist as a functional reference. While
manual expert inspection provides the highest fidelity,
it is prohibitively expensive and difficult to scale or re-
produce. To address this, we also implemented an au-
tomated evaluation pipeline using Gemini 3 as an LLM
Judge. Appendix F contains detail about the judge LLM.
2. Validity Rate: The percentage of generated netlists that
pass Ngspice simulation without syntax errors or conver-
gence failures.
3. Maximum Mean Discrepancy (MMD): We calculate
the MMD between the graph of the generated topologies
and the ground truth topologies in the test set.
Tokenizer Evaluation Metrics:
1. Compression Ratio (CR): Defined as the ratio of char-
acters in the raw topology to the number of tokens
produced:
CR =
Total Characters (Topology-Only)
Total Tokens
(1)
Note: To ensure a fair comparison, “Topology-Only”
inputs strip variable values and model names for both
CKT and baseline tokenizers.
2. Sequence Length Reduction (SLR): The percentage
reduction in sequence length achieved by CKT com-
pared to other baseline tokenizer:
SLR =

1 −
Avg TokensCKT
Avg TokensBaseline

× 100 (2)
5.2. Evaluation of CKT Tokenizer
Compression Efficiency: We first analyze the impact of
vocabulary size on compression efficiency. As shown in
Figure 4, increasing the vocabulary size via subgraph min-
ing directly correlates with higher compression. Table 3
benchmarks CKT against standard text tokenizers. Even
with the minimal vocabulary of 92 tokens (representing
the base primitive set with no grammar rules added), CKT
achieves a 43.8% reduction in sequence length compared
to GPT-4’s ‘tiktoken’. When applying subcircuit mining
under the constraints of Fmin = 2, Gainmin = 1, and
Amax = 4, the tokenizer reaches a maximum vocabu-
lary size of 5139. At this maximum compression level,
CKT achieves a massive 65.5% reduction, demonstrat-
ing that topological macro-tokens are significantly more
information-dense than character-based BPE tokens.























9RFDEXODU\6L]H







&RPSUHVVL
R
Q5DWL
R
&RPSUHVVLRQ5DWLR
Figure 4. Impact of vocabulary size on compression ratio. The
“Min Vocab” (94) represents the base primitive set, while 5139
represents the maximal rule set mined with Fmin = 2. Higher
vocabulary sizes yield denser latent representations.
5.3. Evaluation of Circuit Generation
Comparison with State-of-the-Art: Table 4 presents the
comparative results on CircuitBench-100. CircuitFormer
outperforms massive general-purpose models by a wide
margin. While DeepSeek-V3.2 (685B) achieves a compet-
itive 94% validity rate, it only reaches a 63% success rate
in functional design. In contrast, CircuitFormer achieves
100% validity and an 83% success rate, demonstrating that
domain-specific tokenization and training allows a model
1000× smaller to master circuit topology.
Impact of Vocabulary Size on Generation: We analyzed
how vocabulary size affects model performance (Figure 5)
and observed an inverted-U trend. With a small vocabulary
(under 100 tokens), the success rate is limited to 70% be-
cause the resulting sequences are too long for the model to
handle effectively. Performance improves as we add more
tokens, reaching a peak of 83% success at around 512 to-
kens. This is the “sweet spot” where we balance shorter
sequences with common, easy-to-learn tokens. However,
beyond 1024 tokens, performance starts to drop. This hap-
pens because many subcircuit tokens become too rare in the
training data, making it difficult for the model to learn them
well.
Generalization vs. Memorization: CircuitFormer exhibits
a slightly higher MMD (0.232) compared to baselines like
7

## Page 8

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
Table 3. Comparison of Tokenizer Compression and Sequence Length Reduction (Topology-Only Inputs)
TOKENIZER IMPLEMENTATION
COMPRESSION
RATIO
VOCAB SIZE
REDUCTION VS
BPE (%)
REDUCTION VS
SENTENCEPIECE (%)
BPE TIKTOKEN (GPT-4) 1.09 200K — —
SENTENCEPIECE GEMMA 3 1.04 262K — —
CKT (OURS) MIN VOCAB 1.94 92 43.8% 46.4%
CKT (OURS) OPTIMAL VOCAB 2.54 512 57.1% 59.1%
CKT (OURS) MAX VOCAB 3.16 5139 65.5% 67.1%
Table 4. Comparison of CircuitFormer with SOTA Open Source LLMs on CircuitBench-100.
MODEL PARAMS SUCCESS RATE (↑) VALID RATE (↑) MMD
GPT-OSS 120B 69% 90% 0.180
LLAMA 3.3 70B 31% 60% 0.224
MISTRAL LARGE 2 123B 32% 85% 0.219
GEMMA 3 27B 23% 64% 0.186
DEEPSEEK V3.2 685B 63% 94% 0.202
CIRCUITFORMER (OURS) 0.5B 83% 100% 0.232
    
9RFDEXODU\6L]H/RJ6FDOH

## Page 9








6XFFHVV5DWH

## Page 10

6XFFHVV5DWH
9DOLGLW\5DWH







9DO
L
G
L
W
\5DWH

## Page 11

Figure 5. Impact of vocabulary size on generation quality. While
validity remains robust at 100%, success rate peaks at |V| =
512, highlighting the trade-off between sequence compression and
token sparsity.
GPT-OSS. In the context of our 100% validity rate, this is
a positive indicator. A low MMD often suggests the model
is reproducing “average” circuits close to the training dis-
tribution. The higher MMD, coupled with high functional
success, suggests that CircuitFormer has generalized the un-
derlying design rules and is synthesizing novel topological
variations that differ distributionally from the training set
while maintaining electrical correctness.
5.4. Ablation Studies
We conducted an ablation study on the CKT tokenizer and
the line shuffle augmentation strategy to understand their
overall impact. Table 5 summarizes the results of our abla-
tion studies.
Impact of Graph-Aware Tokenization (CKT vs. BPE):
To quantify the benefit of the CKT tokenizer over a general-
purpose tokenizer in the overall system, we trained a variant
of CircuitFormer using a standard BPE tokenizer instead of
CKT. The BPE tokenizer was trained on the same training
dataset with a vocabulary size of 2000, while keeping all
other settings identical. As shown in Table 5, replacing
CKT with BPE results in a sharp decline in performance.
The BPE-based model achieves only a 71% success rate,
compared to 83% for the CKT-based model, even though
CKT uses a 74.4% smaller vocabulary.
Impact of Line Shuffle Augmentation: We also evaluated
the importance of our line-shuffling augmentation strategy.
We can see in table 5 that just turning off the line shuffle
augmentation, drops the success rate to 23% while main-
taining a validity score of 98%. These results highlight
the critical role of line-shuffle augmentation in improving
training effectiveness and overall success.
Table 5. Ablation Study Results. Both the CKT tokenizer and Line
Shuffle augmentation are critical for achieving high functional
success.
MODEL
VARIANT VALIDITY (%)
SUCCESS
RATE (%)
CIRCUITFORMER (FULL) 100% 83%
W/O CKT (STANDARD BPE) 98% 70%
W/O LINE SHUFFLE 98% 23%
6. Conclusion and Future Scope
We introduced CircuitFormer and the Circuit Tokenizer
(CKT), establishing the first framework capable of generat-
ing valid analog topologies directly from natural language
by resolving the fundamental text-graph mismatch. Beyond
analog design, CKT’s ability to decouple topology from seri-
alization offers a scalable, permutation-invariant solution for
8

## Page 12

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
other graph-structured domains, such as molecular discovery
and network analysis. Furthermore, this methodology holds
significant promise for automating direct transistor-level
digital synthesis, paving the way for a unified, end-to-end
generative design flow across the entire hardware spectrum.
7. Impact Statement
This work addresses the critical data scarcity bottleneck
in Electronic Design Automation (EDA) by releasing the
largest annotated analog circuit dataset to date, a contribu-
tion likely to catalyze a new wave of data-driven research
in analog automation. Beyond hardware, the proposed Cir-
cuit Tokenizer (CKT) establishes a scalable, permutation-
invariant framework for graph serialization, offering a trans-
ferable methodology with significant potential for advancing
generative modeling in other graph-intensive domains such
as chemoinformatics, social network analysis, and knowl-
edge representation.
References
Bhandari, J., Bhat, V., He, Y., Rahmani, H., Garg, S., and
Karri, R. Masala-chai: A large-scale spice netlist dataset
for analog circuits by harnessing ai, 2025. URL https:
//arxiv.org/abs/2411.14299.
Chang, C.-C., Shen, Y., Fan, S., Li, J., Zhang, S., Cao, N.,
Chen, Y., and Zhang, X. LaMAGIC: Language-model-
based topology generation for analog integrated circuits.
In Salakhutdinov, R., Kolter, Z., Heller, K., Weller, A.,
Oliver, N., Scarlett, J., and Berkenkamp, F. (eds.), Pro-
ceedings of the 41st International Conference on Ma-
chine Learning, volume 235 of Proceedings of Machine
Learning Research, pp. 6253–6262. PMLR, 21–27 Jul
2024. URL https://proceedings.mlr.press/
v235/chang24c.html.
Chang, C.-C., Lin, W.-H., Shen, Y., Chen, Y., and Zhang,
X. LaMAGIC2: Advanced circuit formulations for lan-
guage model-based analog topology generation. In Forty-
second International Conference on Machine Learning,
2025. URL https://openreview.net/forum?
id=Y0zXGw0GUk.
Chen, Z., Huang, J., Liu, Y., Yang, F., Shang, L., Zhou,
D., and Zeng, X. Artisan: Automated operational ampli-
fier design via domain-specific large language model. In
Proceedings of the 61st ACM/IEEE Design Automation
Conference, DAC ’24, New York, NY, USA, 2024. Asso-
ciation for Computing Machinery. ISBN 9798400706011.
doi: 10.1145/3649329.3655903. URL https://doi.
org/10.1145/3649329.3655903.
DeepSeek-AI, Liu, A., Mei, A., et al. Deepseek-v3.2: Push-
ing the frontier of open large language models, 2025.
URL https://arxiv.org/abs/2512.02556.
Dong, Z., Cao, W., Zhang, M., Tao, D., Chen, Y., and Zhang,
X. CktGNN: Circuit graph neural network for electronic
design automation. In The Eleventh International Confer-
ence on Learning Representations, 2023. URL https:
//openreview.net/forum?id=NE2911Kq1sp.
Gao, J., Cao, W., Yang, J., and Zhang, X. Analoggenie:
A generative engine for automatic discovery of analog
circuit topologies. In The Thirteenth International Confer-
ence on Learning Representations, 2025a. URL https:
//openreview.net/forum?id=jCPak79Kev.
Gao, J., Cao, W., and Zhang, X. Analoggenie-lite: En-
hancing scalability and precision in circuit topology dis-
covery through lightweight graph modeling. In Forty-
second International Conference on Machine Learning,
2025b. URL https://openreview.net/forum?
id=KRk0WTII0I.
Gemma Team, Kamath, A., Ferret, J., et al. Gemma 3
technical report, 2025. URL https://arxiv.org/
abs/2503.19786.
Grattafiori, A., Dubey, A., Jauhri, A., et al. The llama 3
herd of models, 2024. URL https://arxiv.org/
abs/2407.21783.
Kudo, T. and Richardson, J. SentencePiece: A simple and
language independent subword tokenizer and detokenizer
for neural text processing. In Proceedings of the 2018
Conference on Empirical Methods in Natural Language
Processing: System Demonstrations, pp. 66–71, Brus-
sels, Belgium, November 2018. Association for Compu-
tational Linguistics. doi: 10.18653/v1/D18-2012. URL
https://aclanthology.org/D18-2012/.
Lai, Y., Lee, S., Chen, G., Poddar, S., Hu, M., Pan,
D. Z., and Luo, P. Analogcoder: analog circuit de-
sign via training-free code generation. In Proceedings
of the Thirty-Ninth AAAI Conference on Artificial In-
telligence and Thirty-Seventh Conference on Innova-
tive Applications of Artificial Intelligence and Fifteenth
Symposium on Educational Advances in Artificial In-
telligence, AAAI’25/IAAI’25/EAAI’25. AAAI Press,
2025a. ISBN 978-1-57735-897-8. doi: 10.1609/aaai.
v39i1.32016. URL https://doi.org/10.1609/
aaai.v39i1.32016.
Lai, Y., Poddar, S., Lee, S., Chen, G., Hu, M., Yu, B., Luo, P.,
and Pan, D. Z. Analogcoder-pro: Unifying analog circuit
generation and optimization via multi-modal llms, 2025b.
URL https://arxiv.org/abs/2508.02518.
9

## Page 13

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
Liu, C., Chen, W., Peng, A., Du, Y., Du, L., and Yang,
J. Ampagent: An llm-based multi-agent system for
multi-stage amplifier schematic design from literature for
process and performance porting, 2024. URL https:
//arxiv.org/abs/2409.14739.
Maneth, S. and Peternek, F. Grammar-based graph com-
pression. Information Systems, 76:19–45, 2018. ISSN
0306-4379. doi: https://doi.org/10.1016/j.is.2018.03.
002. URL https://www.sciencedirect.com/
science/article/pii/S0306437917301680.
Mistral AI Team. Large enough: Mistral large 2. https:
//mistral.ai/news/mistral-large-2407/,
July 2024. Accessed: 2026-01-28.
OpenAI, Agarwal, S., Ahmad, L., et al. gpt-oss-120b &
gpt-oss-20b model card, 2025. URL https://arxiv.
org/abs/2508.10925.
Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., and
Sutskever, I. Language models are unsupervised multitask
learners. 2019.
Reimers, N. and Gurevych, I. Sentence-BERT: Sentence
embeddings using Siamese BERT-networks. In Inui,
K., Jiang, J., Ng, V., and Wan, X. (eds.), Proceed-
ings of the 2019 Conference on Empirical Methods
in Natural Language Processing and the 9th Interna-
tional Joint Conference on Natural Language Processing
(EMNLP-IJCNLP), pp. 3982–3992, Hong Kong, China,
November 2019. Association for Computational Lin-
guistics. doi: 10.18653/v1/D19-1410. URL https:
//aclanthology.org/D19-1410/.
Sennrich, R., Haddow, B., and Birch, A. Neural machine
translation of rare words with subword units. In Erk,
K. and Smith, N. A. (eds.), Proceedings of the 54th
Annual Meeting of the Association for Computational
Linguistics (Volume 1: Long Papers), pp. 1715–1725,
Berlin, Germany, August 2016. Association for Compu-
tational Linguistics. doi: 10.18653/v1/P16-1162. URL
https://aclanthology.org/P16-1162/.
Shi, Y., Tao, Z., Gao, Y., Huang, L., Wang, H., Yu, Z., Lin,
T.-J., and He, L. Amsnet 2.0: A large ams database with
ai segmentation for net detection, 2025. URL https:
//arxiv.org/abs/2505.09155.
Song, K., Tan, X., Qin, T., Lu, J., and Liu, T.-Y. Mpnet:
Masked and permuted pre-training for language under-
standing, 2020. URL https://arxiv.org/abs/
2004.09297.
Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones,
L., Gomez, A. N., Kaiser, L. u., and Polosukhin, I.
Attention is all you need. In Guyon, I., Luxburg, U. V.,
Bengio, S., Wallach, H., Fergus, R., Vishwanathan, S.,
and Garnett, R. (eds.), Advances in Neural Information
Processing Systems, volume 30. Curran Associates, Inc.,
2017. URL https://proceedings.neurips.
cc/paper_files/paper/2017/file/
3f5ee243547dee91fbd053c1c4a845aa-Paper.
pdf.
Zhang, H., Sun, S., Lin, Y., Wang, R., and Bian, J. Analogx-
pert: Automating analog topology synthesis by incorpo-
rating circuit design expertise into large language mod-
els. In 2025 International Symposium of Electronics
Design Automation (ISEDA), pp. 772–777, 2025. doi:
10.1109/ISEDA65950.2025.11100627.
Zhao, Z., Kang, H., Wang, B., and He, C. Doclayout-yolo:
Enhancing document layout analysis through diverse syn-
thetic data and global-to-local adaptive perception, 2024.
URL https://arxiv.org/abs/2410.12628.
10

## Page 14

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
A. Example of Circuits Designed by CircuitFormer
Prompt:
”Create a 3-stage ring oscillator using CMOS in-
verters”.
(a) Input Prompt (b) Generated Circuit Topology
Figure 6. CircuitFormer generating a ring oscillator from a natural language prompt. (a) The input user prompt. (b) The corresponding
valid CMOS topology generated by CircuitFormer.
Prompt:
”design a bjt differential amplifier with an active
load current mirror and a tail current source”.
(a) Input Prompt (b) Generated Circuit Topology
Figure 7. CircuitFormer generating a differential amplifier from a natural language prompt. (a) The input user prompt. (b) The
corresponding valid topology generated by CircuitFormer.
11

## Page 15

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
B. CKT Detokenization and Reconstruction Algorithm
The detokenization process serves as the inverse operation to CKT tokenization, strictly designed to be a lossless transforma-
tion with respect to circuit topology. The algorithm reconstructs a valid, simulatable SPICE netlist from a flat sequence of
CKT tokens through a two-stage pipeline: (1) Recursive Graph Expansion, which recovers the primitive graph topology, and
(2) Topological Reconstruction, which resolves semantic dependencies and SPICE syntax requirements.
B.1. Stage 1: Recursive Macro Expansion (Graph-Level)
The objective of this stage is to ”unroll” the hierarchical grammar rules learned during training to recover the fundamental
graph of primitive components.
Queue-Based Expansion: The algorithm initializes a First-In-First-Out (FIFO) queue with the input token sequence. It
operates iteratively:
1. Dequeue: An edge (token) e is popped from the queue.
2. Check Type:
• If e is a Primitive (e.g., ⟨R⟩, ⟨M⟩): It is added to the final set of resolved components.
• If e is a Macro (e.g., ⟨#25⟩): The algorithm retrieves the corresponding Grammar Rule r that generated this
macro.
3. Pattern Mapping: The rule r defines the connectivity between the macro’s external ports (p0, p1, . . . ) and its two
constituent children (Left L and Right R). The algorithm maps the concrete node IDs of the current edge e to the rule’s
internal placeholders to generate two new concrete edges, eL and eR.
4. Recurse: The new edges eL and eR are pushed back into the queue for further processing. This allows for deep
hierarchical structures to be unrolled level-by-level until only primitives remain.
B.2. Stage 2: Topological Reconstruction (SPICE-Level)
Once the primitive graph is recovered, the node identifiers are normalized integers. The second stage transforms this
raw graph into valid SPICE syntax, reversing the ”Topological Branch Identification” transformations applied during
normalization.
1. Branch Re-merging (Controlling Sources): During normalization, branch-defining elements (Voltage Sources V ,
Inductors L) were split into 3-node sequences using a ”dummy” node to allow for topological referencing. The reconstruction
algorithm scans for these specific topological signatures (e.g., a pair of edges identifying as partial split-branches). It merges
the 3-node branch back into a single 2-node component, effectively removing the dummy node and restoring the original
branch.
2. Reference Resolution (F, H, K Elements): Dependent sources (CCCS F, CCVS H) and mutual inductors (K) rely
on named references to controlling branches. In the raw graph, these dependencies exist as direct connections to the
(now-resolved) dummy nodes. The algorithm traverses the graph to identify components connected to these resolved
branches. It replaces the explicit node connection with the name of the restored controlling component (e.g., converting a
topological connection at node ndummy into the text string "Vsense").
3. Canonical Naming and Value Injection: Since instance names (R1, R2) are abstracted during tokenization, the recon-
structor assigns stable, canonical instance names based on component type and traversal order (e.g., R 0, R 1, M 0 . . . ).
The complete detokenization procedure, integrating both the recursive graph expansion and the topological reconstruction,
is summarized in Algorithm 2.
12

## Page 16

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
Algorithm 2 CKT Detokenization Procedure
Input: Token Sequence T, Grammar R
Output: SPICE Netlist Nout
{Stage 1: Recursive Expansion}
Q ← InitializeQueue(T)
Eprimitives ← ∅
while Q is not empty do
e ← Q.pop()
if IsPrimitive(e) then
Eprimitives.add(e)
else
Rule ← R[GetID(e)]
eL, eR ← MapNodes(e, Rule)
Q.push(eL, eR)
end if
end while
{Stage 2: Reconstruction}
G ← BuildGraph(Eprimitives)
G′
← MergeSplitBranches(G) {Resolves V/L dummy nodes}
G′′
← ResolveReferences(G′
) {Resolves F/H/K dependencies}
Nout ← AssignNamesAndValues(G′′
)
return Nout
13

## Page 17

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
C. Dataset
C.1. Can Vision Language Models accurately Parse Netlists from Circuit Schematic Images?
To assess SOTA Vision Language Models’ ability to parse valid SPICE netlists from circuit schematic images, we sampled
25 schematic images randomly from our large corpus of extracted circuit schematic and tasked four prominent VLMs with
generating the corresponding netlists and descriptions.
Table 6 summarizes the performance of these baselines. Our empirical analysis reveals that Gemini 3 significantly
outperforms other candidates in netlist extraction accuracy. Notably, Gemini 3 was the only model capable of consistently
generating accurate netlists while simultaneously producing high-fidelity descriptions of the circuit. Consequently, we
selected Gemini 3 as the backbone for our dataset generation pipeline.
Table 6. Performance comparison of VLM baselines on a random subset of circuit schematic images.
MODEL SAMPLE SIZE % CORRECT
GEMINI 3 38 97.4%
GPT-5.2 37 86.5%
GEMINI 2.5 38 76.3%
GPT-4O 37 51.4%
C.2. VLM Prompt for Generating Netlist and Description Pairs from Schematic Images
VLM Prompt for Schematic Extraction
You get two images: (1) a circuit crop and (2) the full source PDF page. Use
the source page for labels/context; extract every circuit in the crop. Topology
only; no simulation commands. Use standard SPICE components; X for subcircuits.
First line of the netlist MUST BE title. Nodes numbered sequentially from 0, only
numerical node names. If node names are given in the image, add the mapping in
comments of the netlist.
Output a JSON list of objects with: "netlist" (string), "name" (list of possible
circuit names), "desc" (brief description). If no circuit is present, return null
for all fields. Ensure valid JSON.
Output MUST NOT include source page specific identifiers (figure numbers, book
names, etc.). So, no figure names/numbers in the name. If a component value is
missing, use a sensible placeholder (e.g., 10k resistors, 10u capacitors, 1N4148
diodes, 2N2222 BJTs, TL081 opamps). No need to include .model statements, they will
be added later. Include all components in the netlist, even implicit ones (e.g.,
voltage sources not shown but written as Vdd, Vin etc.).
VERY IMPORTANT: The netlist should only be for the circuit(s) present in the circuit
crop image. The source page image is only for fetching description/labels, NOT for
extracting additional circuits. ONLY look at the circuit crop image for the actual
circuit topology.
Make netlist according to standard NGSPICE syntax. Ngspice element guide: A:XSPICE
B:Behavioral C:Cap D:Diode E:VCVS F:CCCS G:VCCS H:CCVS I:Current Source J:JFET
K:MutualL L:Inductor M:MOSFET N:VerilogA O:LossyTL P:CPL Q:BJT R:R S:VSW T:TL
U:RC/XSPICE V:V W:ISW X:Subckt Y:TXL Z:MESFET
When wires overlap, carefully check whether they are electrically connected or
merely crossing. An overlap alone does not imply a connection; typically, a dot
indicates a junction, while no dot means the wires simply pass over each other.
Although there can be exceptions, so pay close attention.
Figure 8. The system prompt used for the VLM to extract netlists from schematic images.
14

## Page 18

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
C.3. Dataset Preparation
Figure 9 shows how the dataset construction pipeline successfully generated a correct netlist and description from a given
circuit schematic image.
(a) Circuit Schematic (b) Source page for Context
Netlist for the circuit:
Differential amplifier with current mirror biasing
.model NMOS NMOS
.model PMOS PMOS
Vdd 1 0 1.8
Iref 1 2 100u
* NMOS bias circuit
M0 2 2 0 0 NMOS W=10u L=1u
MB1 3 2 0 0 NMOS W=20u L=1u
MB2 4 2 0 0 NMOS W=50u L=1u
* PMOS bias circuit
MrefP 3 3 1 1 PMOS W=10u L=1u
* Differential amplifier stage
M1 5 7 4 0 NMOS W=20u L=1u
M2 6 8 4 0 NMOS W=20u L=1u
* Load and bias current sources
M3 5 5 1 1 PMOS W=10u L=1u
M5 5 3 1 1 PMOS W=10u L=1u
M4 6 6 1 1 PMOS W=10u L=1u
M6 6 3 1 1 PMOS W=10u L=1u
* Inputs
Vin1 7 0 0.9
Vin2 8 0 0.9
* Node mapping: VDD=1, X=2, bias_P=3, P=4,
* Vout_L=5, Vout_R=6, Vin1=7, Vin2=8
(c) Generated SPICE Netlist
Description for the circuit:
”A differential amplifier stage where the tail current and the load currents are set
by current mirrors from a single reference current source IREF. NMOS M0 and
MB1-MB2 mirror the reference current to the tail node (P) and to a PMOS bias
branch. PMOS MrefP then mirrors this current to M5 and M6, which act as current
source loads in parallel with diode-connected PMOS loads M3 and M4.”
Generated Names:
• Current mirrors used to bias a differential amplifier
• Biased differential pair with active loads
• Differential amplifier with auxiliary PMOS current sources
(d) Generated Description and Names
Figure 9. Netlist and description pair correctly generated by VLM for circuit ID ckt 33298469 in the dataset. (a) The extracted circuit
schematic. (b) The original source page for VLM context. (c) The Ngspice compatible netlist generated for the schematic circuit. (d) The
natural language description and names generated from the schematic.
15

## Page 19

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
C.4. Dataset Statistics
Figure 10 illustrates the distribution of circuit elements across the full dataset. Figure 11 presents the histogram of netlist
sizes. Table 7 summarizes the categories of all books processed to construct the dataset.
R
V
C
X
M
D
Q
L
I
S
E
K
G
B
T
J
F
H
Z
N
Element prefix
0
20000
40000
60000
80000
100000
Count
96380
60006
46232
31502
22956
16701
16247
15286
7320
3864
3418
2707
2278
1227
953
715
560
190
21
2
Figure 10. Distribution of SPICE elements in the dataset.
0 20 40 60 80 100 120 140
Components per netlist
100
101
102
103
104
Frequency
Figure 11. Histogram of netlist sizes in the dataset. Frequency axis is logarithmic.
16

## Page 20

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
Table 7. Distribution of Source Books by Category Used in Dataset Construction.
Category Count
Fundamental Circuit Analysis 1
Microelectronics Basics 3
General Analog IC Design 4
CMOS Design, Layout & Modeling 4
Practical Electronics & Troubleshooting 3
Operational Amplifiers (Theory & App.) 5
Specialized Amplifiers (Opto/Feedback) 2
Audio Amplifiers & Systems 3
Analog Filters 2
Data Converters (ADC/DAC) 2
Oscillators & Frequency Control 1
RF General Design 2
RFIC Design (CMOS/Integrated) 3
Microwave Engineering & Active Circuits 3
RF Passive, Lumped Elements & Wireless 2
Power Electronics Principles 4
Switching Power Supply Design 3
Magnetics (Inductors/Transformers) 1
Sensor Technologies 1
Optoelectronic ICs 1
Interfacing & Embedded Systems 1
Circuit Encyclopedias 6
Hobbyist Circuit Collections 4
Beginner/DIY Projects 1
Total 62
17

## Page 21

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
D. CircuitBench-100
Table 8 summarizes the circuit categories represented in the CircuitBench-100 test set.
Table 8. Distribution of Circuit Categories in the CircuitBench-100 Test Set
Category Constituent Topologies Count
Amplifiers Common-Emitter, Common-Source, Cascode, Differential, Op-
Amps, Voltage Follower, Darlington, Output Stages, Microwave,
JFET
24
Basic Components & Net-
works
Single Transistor Test, Diode Circuits, Resistive Bridges, RLC Ba-
sics, Parallel/Series Networks, Two-Port Networks
18
Modeling & Equivalent Cir-
cuits
Small/Large-Signal Models, Layout Models, Transformer Models,
Resistor Models
13
Biasing & References Current Mirrors, Widlar Source, Voltage Bias, Biasing Networks,
Current Steering, Constant Current Sources
12
Filters Low-Pass, High-Pass, LC Ladder, Chebyshev, Butterworth, Pi-
Network, Attenuators
11
Power Electronics Buck-Boost Converter, Switching Circuits, Rectifiers, Class E Am-
plifier
6
Logic & Digital Logic Gates, Pass-Transistor Logic, Shift Registers, Transmission
Gates, CMOS Inverter
5
RF & Systems Transmission Lines (Kuroda), Y-Parameter Test, Feedback Systems,
Control Blocks
5
Oscillators Colpitts, Cross-Coupled LC, RF Video Modulator 3
Sensors & Opto Phototransistor, IR LED Driver, LED Indicators 3
Total 100
E. Model Architecture and Training Strategy
CircuitFormer is a hybrid encoder-decoder Transformer architecture (Vaswani et al., 2017) designed to translate natural
language specifications into valid circuit topologies. The architecture integrates a semantic-rich encoder with a syntax-aware
decoder, bridged by cross-attention mechanisms.
E.1. The Hybrid Architecture
Encoder (Sentence-BERT): Standard BERT embeddings often suffer from the anisotropy problem, where embeddings
occupy a narrow cone in the vector space, limiting their utility for semantic similarity tasks. To mitigate this, we employ
a pre-trained Sentence-BERT (sBERT) (Reimers & Gurevych, 2019) as the encoder backbone. sBERT is optimized via
siamese networks to derive semantically meaningful sentence embeddings, ensuring the model captures the nuances of the
input prompts. The encoder utilizes a standard WordPiece tokenizer to process natural language.
Decoder: The decoder utilizes a GPT-2 architecture (Radford et al., 2019) extended with cross-attention layers. Unlike the
encoder, the decoder employs our custom CKT Tokenizer (Section 3) to efficiently represent circuit topologies.
Integration via Cross-Attention: We integrate the encoder and decoder by injecting cross-attention layers into each
block of the decoder. While the decoder’s self-attention models intra-netlist dependencies, the cross-attention mechanism
conditions the generation process on the sBERT encoder’s semantic embeddings, effectively grounding the circuit generation
in the natural language prompt.
18

## Page 22

CircuitFormer: A Circuit Language Model for Analog Topology Design from Natural Language Prompt
E.2. Three-Stage Optimization Strategy
To effectively align the pre-trained modules and ensure robust performance, we adopt a three-stage training curriculum:
Stage 1: Domain-Adaptive Decoder Pre-training: Before integration, we pre-train the decoder in isolation on our
large-scale netlist corpus using a standard causal language modeling (CLM) objective. This step allows the model to
internalize the SPICE netlist grammar and the topological structures encoded by the CKT tokenizer, independent of any
natural language conditioning.
Stage 2: Alignment (Cross-Attention Warmup): We then assemble the full architecture, initializing the cross-attention
layers randomly while loading the pre-trained weights for the encoder and decoder. In this stage, we freeze the weights
of both the sBERT encoder and the GPT-2 decoder, optimizing only the newly initialized cross-attention layers. This
phase aligns the decoder’s latent space with the encoder’s semantic representations without catastrophically forgetting the
pre-learned syntactic features.
Stage 3: End-to-End Fine-Tuning: Finally, we unfreeze the entire architecture and fine-tune all parameters in a supervised
manner. This stage optimizes the global model to minimize the cross-entropy loss between the generated tokens and the
ground-truth netlists, refining the interaction between semantic understanding and topological generation.
F. The Judge LLM
The judge LLM evaluates functional equivalence by comparing the generated netlist against the prompt and the reference
ground-truth topology. Using Gemini 3 as the judge, we performed a correlation study between automated LLM-based
evaluations and human expert assessments of state-of-the-art models and CircuitFormer. As shown in Table 9, the judge
LLM shows strong agreement with human evaluations, supporting its use as a reliable and reproducible proxy for large-scale
benchmarking.
Table 9. Comparison of success rates between Human Expert evaluation and the LLM Judge. The minimal difference validates the
automated judge’s reliability.
MODEL
MANUAL
EVAL.
JUDGE LLM
EVAL.
ABS.
DIFF
GPT-OSS 69% 71% 2%
LLAMA 3.3 31% 34% 3%
MISTRAL LARGE 2 32% 29% 3%
GEMMA 3 23% 22% 1%
DEEPSEEK V3 63% 65% 2%
CIRCUITFORMER 83% 85% 2%
19

