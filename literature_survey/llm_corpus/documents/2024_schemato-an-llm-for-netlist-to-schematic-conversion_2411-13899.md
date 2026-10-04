---
title: "Schemato -- An LLM for Netlist-to-Schematic Conversion"
authors: "Matsuo, Ryoga; Uhlich, Stefan; Venkitaraman, Arun; Bonetti, Andrea; Hsieh, Chia-Yu; Momeni, Ali; Mauch, Lukas; Capone, Augusto; Ohbuchi, Eisaku; Servadei, Lorenzo"
date: "2024/11/21"
category: "topology_schematic"
source_url: "https://arxiv.org/abs/2411.13899"
pdf: "papers/2024_schemato-an-llm-for-netlist-to-schematic-conversion_2411-13899.pdf"
sha256: "339a7fbed38c2536e794629bf7a7d27fa78f47cf941a7400d318c85d3a1d30d9"
pages_extracted: 7
---

# Schemato -- An LLM for Netlist-to-Schematic Conversion

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

arXiv:2411.13899v2
[cs.LG]
2
Jun
2025
Schemato – An LLM for Netlist-to-Schematic Conversion
Ryoga Matsuo1,2
, Stefan Uhlich3
, Arun Venkitaraman1
, Andrea Bonetti1
, Chia-Yu Hsieh1
, Ali Momeni1,2
Lukas Mauch3
, Augusto Capone3
, Eisaku Ohbuchi4
, Lorenzo Servadei1,5
1
SonyAI, Switzerland 2
EPFL, Switzerland
3
Sony Semiconductor Solutions Europe, Germany 4
Sony Semiconductor Solutions, Japan 5
TU Munich, Germany
Abstract—Machine learning models are advancing circuit design,
particularly in analog circuits. They typically generate netlists that
lack human interpretability. This is a problem as human designers
heavily rely on the interpretability of circuit diagrams or schematics
to intuitively understand, troubleshoot, and develop designs. Hence, to
integrate domain knowledge effectively, it is crucial to translate ML-
generated netlists into interpretable schematics quickly and accurately.
We propose Schemato, a large language model (LLM) for netlist-
to-schematic conversion. In particular, we consider our approach in
converting netlists to .asc files, text-based schematic description used
in LTSpice. Experiments on our circuit dataset show that Schemato
achieves up to 76% compilation success rate, surpassing 63% scored
by the state-of-the-art LLMs. Furthermore, our experiments show that
Schemato generates schematics with an average graph edit distance score
and mean structural similarity index measure, scaled by the compilation
success rate that are 1.8× and 4.3× higher than the best performing
LLMs respectively, demonstrating its ability to generate schematics that
are more accurately connected and are closer to the reference human
design.
Index Terms—Netlist to Schematic, Large language models (LLMs),
Electronic design automation (EDA), Fine-tuning.
I. INTRODUCTION
Recent years have seen a rapid increase in the number of machine
learning (ML)-based solutions that address different aspects of analog
design, specially towards both topology and sizing [1]–[6]. The netlist
representation of the analog circuit forms a central part of all these
approaches, since they all operate by either encoding the information
in the netlist or generating a netlist as part of an end-to-end ML
model. However, often such circuits, while they may maximize a
certain objective function and, hence, are feasible in the eyes of an
ML practician, might not result in a practical design in the eyes of
an analog designer. An analog designer usually views and assesses
a circuit through the circuit diagram or schematics and not a netlist.
This means that having a schematic or circuit diagram corresponding
to a netlist is crucial for the ML models engineer to benefit from the
domain knowledge of the designer’s feedback. This is specially the
case when the circuits become large in size and when one considers
problems such as synthesizing circuits [7], [8], where the designer
must spend considerable resources in drawing a schematic by hand
each time to be able to give feedback to the ML models more
effectively. Thus, it is highly desirable to be able to efficiently and
automatically translate netlists into circuit diagrams or schematics.
Traditionally, circuit schematics are manually generated using
conventional electronic design automation (EDA) tools by analog
designers and visualized with associated viewing tools, such as
LTSpice, xschem and Cadence Virtuoso. LTSpice and xschem process
text-based structured schematic files (.asc for LTSpice and .sch
for xschem) to produce circuit diagrams [9], [10]. These methods
rely on structured programs with specific semantics or “languages”.
Similarly, netlists adhere to well-defined semantics for circuit repre-
sentation, making the task of conversion to a schematic or diagram
analogous to a language translation task. This analogy highlights the
potential of leveraging large language models (LLMs), which excel
at understanding and predicting semantic structures [11], [12]. LLMs
have also recently shown great potential in different aspects of EDA,
particularly, in digital and logic synthesis [4], [13]–[17] wherein
the models learn to understand, abstract, and build upon the design
process of human designers. This motivates us to propose an LLM-
based solution for translating netlists to circuit schematics/diagrams.
The earliest works on automated netlist to schematic generation
have mostly been in the domain of digital ICs [18], [19]. Some
of the early works for analog circuits include [20] that introduced
a symmetry-based placement algorithm that identifies symmetrical
structures within a netlist, and the work of [21] that concentrated
on schematic routing aimed at minimizing net crossings. The more
recent work of [22] presents a ML-based approach that combines
identification of subblocks with an RL block that generates the
schematic by optimizing for a reward based on the building block
compliance rate that measures aesthetic of the generated schematic
based on topological and heuristic constraints.
While these approaches have been shown to perform well on differ-
ent circuit topologies, they rely on either the explicit use of heuristic
rules or an understanding of the underlying topology/hierarchy of its
subblocks or subcircuits or on the use of specific aesthetic reward
functions. Schemato overcomes these limitations by developing an
automated netlist to schematic conversion system that is independent
of the circuit type or functionality, enabling broad applicability
across diverse designs without requiring prior knowledge of specific
circuit characteristics, while maintaining as faithfully as possible
the aesthetics of a human design. This is because our approach
learns from human-designed circuit examples and therefore does not
require an explicit aesthetics measure. We also note that due to the
universal nature of the framework, we are not limited to specific
schematic templates or the choice of the schematic editor; although
we have used LTSpice with the .asc file format in our experiments,
our approach is equally valid with the use of other EDA tools with
their schematic formats.
In particular, our contributions are:
• We propose Schemato, an LLM for automatic translation of
netlists to text-based schematic representations, fine-tuned on
human-created design examples.
• In particular, we perform a task of netlist-to-schematic conver-
sion, namely, netlist-to-asc conversion for use with LTSpice.
• We perform exhaustive experiments on publicly available circuit
and evaluate the performance of our approach using various
metrics such as graph edit distance (GED) scores, mean struc-
tural similarity index measure (MSSIM), compilation success
rate (CSR), and bilingual evaluation understudy (BLEU).
• Experiments show that Schemato outperforms state-of-the-art
LLMs with a CSR of 76%, a 1.8× higher average GED score
scaled by CSR, and a 4.3× higher average MSSIM scaled by
CSR over the test set.
To the best of our knowledge, no prior work exists that addresses this
specific and important problem using LLMs, and given the increasing

## Page 2

Fig. 1: Use case of Schemato: AI conversion of SPICE netlists into human-readable schematics.
influence of LLMs, we believe our contribution is both timely and
valuable for advancing ML-based approaches for analog design.
II. PRELIMINARIES
In this section, we present a short review of LLMs followed by
a short review of the LTSpice .asc schematic format that builds the
basis of our current work.
A. Review of LLMs
Large Language Models (LLMs) are advanced AI systems ca-
pable of performing complex linguistic tasks such as translation,
summarization, and reasoning by leveraging large datasets and
neural architectures [11], [12]. Their effectiveness often hinges on
prompt engineering, with techniques like Chain-of-Thought and few-
shot prompting enabling more accurate and nuanced outputs [23].
However, as most LLMs are trained on general-purpose data, their
performance in specialized domains like EDA may be limited due
to the lack of task-specific optimization [13]–[15], [17]. Addressing
this gap typically requires fine-tuning or domain-specific prompt
strategies.
In our work, we consider different combinations of prompt design,
in-context examples, and fine-tuning of LLMs to translate netlists
into circuit schematics. Our experiments demonstrate that while both
prompt engineering and the incorporation of few-shot examples are
crucial for improving the output of the original LLM, fine-tuning
yields a substantially enhanced LLM, called Schemato.
B. Review of LTSpice-specific .asc file format
We next shortly review the key terms in the LTSpice .asc file1
As we discuss later, this will also motivate some of the prompts
used in our experiments. In the .asc file, each line starts with one
of the following keywords accompanied by the corresponding input
arguments
• “Version index” defines the LTSpice version.
• “SHEET index height width”: defines the sheet index
and size on the LTSpice application.
• “WIRE xstart ystart xend yend”: creates a wire that extends
from position (xstart, ystart) to (xend, yend).
• “SYMBOL component xcomp ycomp θ”: instantiates a cir-
cuit component (e.g. nmos, pmos, resistor, opamp, inverter)
at the specified position (xcomp, ycomp) and orientation θ,
where θ = R90 corresponds to 90◦
anti-clockwise rotation
and θ = M270 for 270◦
anti-clockwise rotation combined with
mirroring. This line is followed by one of these lines that sets
a symbol attribute:
– “SYMATTR InstName component_name”: assigns the
component’s name.
– “SYMATTR SpiceModel model_name”: assigns the
component’s spice model.
– “SYMATTR ModelFile model_file”: assigns the com-
ponent’s spice model path.
1More details may be found in the documentation of LTSpice [9].
III. PROPOSED APPROACH: Schemato
In Fig. 1, we present an illustration of the Schemato-based circuit
design workflow. The left half illustrates the fully data-driven model,
while the right half integrates domain knowledge contributions from
analog designers.
As discussed in Sec. I, Schemato addresses the task of automating
circuit schematic generation from netlists. At its core, Schemato is
an LLM fine-tuned using a curated dataset of human-created netlist-
to-schematic pairs. Its development focuses on three key components:
• Task Definition: Constructing prompts that define the netlist-to-
schematic conversion task.
• Guidance via Context: Incorporating in-context examples within
prompts to guide the model.
• Fine-Tuning: Adapting the model using a specialized dataset of
netlist-schematic pairs.
Schemato supports the .asc file format for schematic genera-
tion. This file format is compatible with the LTSpice schematic
viewer/editing tool. While our experiments focus on LTSpice due
to its data availability, the approach generalizes to any text-based
schematic format.
A. Task definition and guidance via context
We consider the task to translate netlists to LTSpice text-based
schematics for the evaluation of different models. Furthermore, to
develop and refine Schemato, we consider different prompting
techniques to guide the LLM effectively. The summary of these
prompt variants are provided in Fig. 2. As shown in the figure,
Prompt 1 is a simple, task-oriented description that serves as the base-
line for comparison. Prompt 2 introduces syntax-specific keywords
relevant to the target format, such as LTSpice symbols(Sec. II-B).
Prompt 3 enhances guidance by providing the initial two lines of the
expected output, including the sheet size of the reference schematic.
This guides the LLM to generate a schematic with the intended
layout and representation. Prompt 4 incorporate specific netlist-to-
schematic conversion examples, offering the model concrete patterns
for improved accuracy. Finally, Prompt 5 is merely a combination of
Prompt 3 and 4. In summary, Prompts 1-3 utilize zero-shot learning,
while Prompts 4-5 employ one-shot learning.
B. Fine-tuning
Schemato is developed by fine-tuning state-of-the-art LLMs using
human-generated netlist-schematic data pairs. To address the limited
availability of such data, we employ data augmentation techniques
to expand the dataset, ensuring better coverage of diverse scenarios.
To select the optimal base model, we first evaluate various LLMs
using all proposed prompts. The combination of prompt and model
yielding the highest performance is then fine-tuned resulting in
Schemato. The comprehensive details on fine-tuning methodology,
data preprocessing, and augmentation strategies are provided in the
next section.

## Page 3

Zero-shot One-shot
Prompt 1: Simply instruct the LLM for the task
Convert the following netlist to a .asc file, a text-based circuit schematic file for
LTSpice. Only output the code and contain it in “‘.
Input: “‘Netlist“‘
—
Output:
Prompt 2: Specify the types of keywords to start each line of the output
Convert the following netlist to a .asc file, a text-based circuit schematic file for
LTSpice. Start each line with keywords such as ‘Version‘, ‘SHEET‘, ‘WIRE‘,
‘FLAG‘, ‘SYMBOL‘, and ‘SYMATTR‘. Only output the code and contain it in “‘.
Input: “‘Netlist“‘
—
Output:
Prompt 3: Specify the first two lines including the sheet size
Convert the following netlist to a .asc file, a text-based circuit schematic file for
LTSpice. Start your answer with
“‘Version 4
SHEET index height width “‘
. Only output the code and contain it in “‘.
Input: “‘Netlist“‘
—
Output:
Prompt 4: Provide an in-context example of conversion
Convert the following netlist to a .asc file, a text-based circuit schematic
file for LTSpice. Only output the code and contain it in “‘.
Here is an example:
Input: “‘Example netlist of a low-pass filter (LPF)“‘
—
Output: “‘Example .asc file of the LPF“‘
Input: “‘Netlist“‘
—
Output:
Prompt 5: Combination of Prompt 3 and 4
Convert the following netlist to a .asc file, a text-based circuit schematic
file for LTSpice. Start your answer with
“‘Version 4
SHEET index height width “‘
. Only output the code and contain it in “‘.
Here is an example:
Input: “‘Example netlist of an LPF“‘
—
Output: “‘Example .asc file of the LPF“‘
Input: “‘Netlist“‘
—
Output:
Fig. 2: Zero-shot Prompt 1-3 and one-shot Prompt 4-5 for netlist-to-schematic conversion.
IV. EXPERIMENTAL SETUP
In our experiments, we use LTSpice schematics from GitHub,
which contain circuits with diverse features and applications. We
also include the publicly available LTSpice circuit files packaged
with LTSpice XVII (examples in folders, Eduational & Applications
[9]). These sets are used for training and validation. In addition,
we consider the LTSpice schematics contained in the public GitHub
repository: Circuits-LTSpice2
for testing. The circuit examples in
this repository are mostly composed of generic components such as
resistors, capacitors, inductors, bipolar transistors, MOS transistors,
voltage, and current sources. This feature of the test set facilitates the
model validation for generalization. In all the cases, we start from
the available .asc files. These are then processed through LTSpice to
create the corresponding netlists. Some .asc files cannot be compiled
into netlist due to missing symbols in the default libraries of LTSpice.
These .asc files are omitted from our dataset. To ensure the validity
of our method in measuring the generalization performance, we
remove the training samples of .asc and netlist files that overlap
with the existing test samples. We next describe the details of data
preprocessing and filtering.
A. Preprocessing and filtering
To facilitate Schemato to learn the translation task, we preprocess
the dataset by removing unnecessary information for the schematic
visualizations in LTSpice. This includes spice simulation commands,
spice model definitions, user annotations, comments, etc. We explain
how we preprocess .asc and netlist files below.
LTSpice-specific .asc files:
• Modify the schematic sheet size defined by “SHEET index
height width” by setting the height and width to the max-
imum vertical/horizontal coordinate minus the minimum verti-
cal/horizontal coordinate out of all the components. Schemato
can use the modified values to learn the geometries of the
schematics.
2https://github.com/mick001/Circuits-LTSpice
• Modify the coordinates of all the wires and components such
that the center of the schematic is defined at the origin (0, 0).
This makes the dataset shift-invariant.
• Replace SpiceModel and ModelFile with InstName
while using the original model_name and model_file for
component_name. This allows for any line with SYMATTR
to be compiled and component_name can be set arbitrarily.
• Remove the lines starting with *, TEXT, RECTANGLE, WINDOW,
LINE, and CIRCLE.
• Remove the user annotations/comments since they are not
needed for the schematic visualization by LTSpice.
• Discard .asc files that do not contain SYMBOL-SYMATTR pair
as this type of example is not composed of meaningful circuit
components except for wires.
Netlists: The file format we use for netlist is .net. We remove the
lines with commands such as .backanno, .lib, and .model
as they are not required for the schematic visualizations.
We next consider the augmentation strategy used to increase the
effective dataset size.
B. Data augmentation
For Schemato to be agnostic to the line ordering of .asc files, we
create an augmented data example by swapping the lines of the .asc
files. Specifically, .asc files can have different orders of SYMBOL-
SYMATTR pairs resulting in the same schematic representations.
Therefore, we randomly shuffle their line ordering and generate five
different augmentations for .asc files with more than three SYMBOL-
SYMATTR pairs. For .asc files with two SYMBOL-SYMATTR pairs, we
swap their order to generate two samples and we leave the files with
one SYMBOL-SYMATTR pair unmodified as we cannot augment such
samples. After the generation of these augmented .asc file samples,
we convert them into netlists to create the dataset as described in
Sec. IV-A. The changes of the line ordering in .asc files are reflected
in the line ordering of netlists without any modifications in the
connectivity of the circuits.
Initially, the self-prepared dataset contains 27,092 (22,045 from
Github and 5,047 from LTSpice XVII) publicly available .asc circuit

## Page 4

files. After filtering and removing the training samples overlapping
with the test samples, it is reduced to 10,428 different circuit
schematics. This filtered dataset is split into 9,907 (95%) training and
521 (5%) validation circuit files, which after augmenting each circuit
file into up to 5 samples, result in a total of 44,995 and 2,407 samples
for training and validation, respectively. For testing, we obtain 117
filtered circuits out of 122 circuit examples from Circuits-LTSpice.
C. Inference and fine-tuning
We start with Llama 3.1 base & instruct versions with 8 billion
parameters (Llama-3.1-8B) [12] to develop Schemato. The tasks are
first executed on the raw models to select the best performing prompts
that are then used for fine-tuning. We use the recently proposed
Torchtune [24] framework for LLM fine-tuning. We first discuss the
details of inference and fine-tuning of the models.
During inference, Schemato generates schematics in an auto-
regressive way. Here, we used no special techniques to save memory
or compute except for KV-caches. We apply greedy search, where
the LLM selects the most probable next sample. Decoding terminates
either if maximum number of tokens (Ntokens) are reached or the stop-
token is sampled. We choose Ntokens = 8192 as this is the maximum
number of tokens that can fit in a single NVIDIA Ada-6000 GPU.
With this setup, the inference on Schemato achieved approximately
23 tokens/sec on a single NVIDIA Ada-6000.
To adapt the LLM to our specific task of translating netlists
into schematics, we employed supervised instruction tuning on a
paired dataset containing prompts with netlists and corresponding
reference schematics as responses. However, fine-tuning LLMs like
Schemato with 8 billion parameters requires a significant amount
of GPU memory. To address this issue, we utilized fully-sharded
data parallel (FSDP) training [25], which enables the distribution
of the model’s parameters across multi-GPUs while minimizing
communication overhead. To further reduce memory consumption
during training, we applied several additional techniques:
• We used bfloat16 representation for model parameters to de-
crease the memory usage compared to float32.
• Activation checkpointing was employed to compute gradients in
a more memory-efficient manner.
• Low-rank adaptation (LoRA) with rank r = 8 and scaling factor
α = 16 was applied to reduce the memory requirements of the
trainable weights.
• Cross entropy with chunked output loss [24] is used to measure
the difference between the generated token and the ground truth.
By combining these techniques, we fine-tune Llama-3.1-8B over 10
epochs (in each epoch, Llama-3.1-8B goes through the entire training
set). The number of weight updates per epoch is:
Ntrain
Niter × NGPU × Nbatch
=
44, 995
8 × 8 × 1
≈ 703
where Ntrain is the number of training samples, Niter is the number of
iterations per accumulation step, NGPU is the number of NVIDIA
Ada-6000 GPUs used for fine-tuning, and Nbatch is the per-GPU
batch size. This setup achieved a throughput of approximately 563
tokens/sec per GPU, allowing us to efficiently adapt the model to
our specific task within a reasonable timeframe. We next discuss the
different metrics used to evaluate the performance of the models.
D. Performance metrics
We use the following four metrics to evaluate the samples gen-
erated by the different models in order to capture various aspects
of schematic generation: (1) graph edit distance (GED) score that
measures the similarity of ground truth and generated graphs to
compare their connectivity, (2) mean structural similarity index mea-
sure (MSSIM) that measures visual similarity with human generated
design, (3) compilation success rate (CSR) that measures syntatic
correctness, and (4) bilingual evaluation understudy (BLEU) that
measures an N-gram similarity. We describe these metrics next.
1) Graph-based metric: The graph edit distance (GED) measures
the minimum number of modifications (inserting/deleting/substituting
a node/edge) required to transform one graph to another. The algo-
rithm to compute this metric is often implemented with shortest-
path algorithms like the Dijkstra’s algorithm [26]. As this metric is
provided as an integer value, we need to normalize it such that it is
agnostic to the graph size. We formulate the GED score as
GED score = 1 −
GED
max(Nn, gen, Nn, ref) + max(Ne, gen, Ne, ref)
where Na, b is the number of a (n for nodes and e for edges) in b
(gen for generated and ref for reference) samples. The computation
of GED relies on the function, “graph edit distance” from NetworkX
[27], which utilizes the depth-first graph edit distance algorithm [28].
The time-out is set to 60 seconds for each sample pair comparison to
prevent an excessively long search. The function returns the minimum
GED found so far once this limit is reached. To map the generated asc
files into the form of graphs, we again use LTSpice to compile them
back into netlists. Then, we convert these netlists and the reference
netlists provided in the prompt into nxgraph class using NetworkX
and compute the GED scores. For cases where the generated asc script
does not successfully compile into a netlist, we omit the sample from
the GED score calculation. We then compare the performance of the
models using GED score, that is the GED score averaged over all
compilable test samples.
2) Image-based metric: The structural similarity index measure
(SSIM) is a metric that computes the structural similarity of two
images based on the statistical comparisons in luminance, contrast,
and structure [29]. For a given pair of image patches x and y, SSIM
is defined as
SSIM(x, y) =
(2µxµy + C1)(2σxy + C2)
(µ2
x + µ2
y + C1)(σ2
x + σ2
y + C2)
where µp and σp are the average and the standard deviation of an
image patch p, respectively, σxy is the square-root of the cross-
correlation between the two image patches x and y, C1 and C2 are
constants added to avoid computational instability. To evaluate the
overall similarity between an image pair X and Y , a sliding local
window is applied over the entire images and SSIM is computed
at each pixel step within this window. The mean SSIM (MSSIM)
between two images X and Y is then given by
MSSIM(X, Y ) =
1
M
XM
i=1
SSIM(xi, yi)
where xi and yi are the image patches in the i-th local window and
M is the number of local windows applied on the images. We use
this metric to derive the similarities between the reference schematics
in the test set and the schematics generated by LLMs from netlists.
To obtain the images of generated and reference schematics, we load
each .asc file with LTSpice and take a screenshot. These images are
subsequently cropped to eliminate the artifacts from the graphical
user interface. Similarly to the GED scores, we omit the incompilable
samples from the calculation. We then compare the performance
of the models using MSSIM, the MSSIM score averaged over all
compilable test samples.

## Page 5

TABLE I: Test performance comparison. GED is equivalent to the
average GED score, GED score. The values on the left and right side
of / indicate the raw scores and scores scaled by CSR, respectively
for GED score and MSSIM. The best and second best metrics are
highlighted in red and blue respectively.
Model Version Prompt GED MSSIM CSR(%) BLEU
Pretrained models
Llama-3.1-8B base 1 - - 0.00 0.00
Llama-3.1-8B instruct 1 - - 0.00 0.00
GPT 4o 1 0.21/0.06 0.05/0.01 27.35 9.24
Llama-3.1-8B base 2 0.26/0.01 0.02/0.00 3.42 6.40
Llama-3.1-8B instruct 2 0.26/0.01 0.02/0.00 3.42 6.40
GPT 4o 2 0.22/0.06 0.04/0.01 27.35 11.97
Llama-3.1-8B base 3 - - 0.00 0.00
Llama-3.1-8B instruct 3 - - 0.00 0.00
GPT 4o 3 0.13/0.00 0.02/0.00 0.85 8.76
Llama-3.1-8B base 4 0.30/0.13 0.06/0.03 41.88 10.51
Llama-3.1-8B instruct 4 0.30/0.13 0.03/0.01 41.88 10.51
GPT 4o 4 0.23/0.12 0.07/0.04 53.85 15.22
Llama-3.1-8B base 5 0.27/0.10 0.06/0.02 38.46 10.55
Llama-3.1-8B instruct 5 0.27/0.10 0.06/0.02 38.46 10.55
GPT 4o 5 0.23/0.15 0.03/0.02 63.25 14.84
Best fine-tuned model
Schemato instruct 3 0.35/0.27 0.22/0.17 76.07 22.17
TABLE II: Ablation study for base model selected for fine-tuning.
The values on the left and right side of / indicate the raw scores
and scores scaled by CSR, respectively for GED score and MSSIM.
For each prompt, the best performing weights are chosen. The best
metrics are highlighted in red.
Model Epoch Version Prompt GED MSSIM CSR(%) BLEU
Models fine-tuned with different prompts
Llama-3.1-8B 4 instruct 2 0.39/0.27 0.03/0.02 70.09 20.11
Llama-3.1-8B 5 instruct 4 0.43/0.28 0.04/0.03 64.10 15.26
Llama-3.1-8B 9 instruct 5 0.33/0.26 0.20/0.16 77.78 23.40
Best fine-tuned model
Schemato 6 instruct 3 0.35/0.27 0.22/0.17 76.07 22.17
3) Compilation-based metric: The compilation success rate (CSR)
measures the syntactical correctness of the generated .asc files and is
defined as
CSR =
Ncompilable
Nref
where Ncompilable is the total number of generated schematics in the
test set that can be compiled into netlists using LTSpice and Nref is
the total number of reference samples.
4) Text-based metric: The bilingual evaluation understudy (BLEU)
compares N-grams of the generated text and the reference translation
and counts the number of matches independently of the positions
[30]. With this measure, we compare the N-gram similarity between
the reference and generated text-based schematics. While BLEU is
not perfectly suited for our tasks due to the syntactical nature of
.asc files, it can indicate the extent to which the pre-trained models
already understand their file syntax. Furthermore, we selected the
standard BLEU as our metric over CodeBLEU [31] specialized to
capture syntactic and semantic features of codes since this measure
does not support .asc file formats. We compare the performance of
the models by computing BLEU, that is the average BLEU score
across all test samples.
V. RESULTS & DISCUSSION
In order to select the starting model for fine-tuning, we evaluated
the performance of Llama-3.1-8B raw models across various config-
urations. Table I shows the performance of the different models on
Fig. 3: Training and validation losses. Llama-3.1-8B is fine-
tuned with four different prompts: (a) Prompt 2, (b) Prompt 3, (c)
Prompt 4, and (d) Prompt 5. Validations losses are computed after
every training epoch.
the evaluation using the test set. As can be seen, the performance
did not vary across the two versions: base and instruct. Moving from
zero-shot (Prompt 1-3) to one-shot prompting (Prompt 4-5) improved
the best CSR significantly from 3.42 to 41.88% while its impact
is difficult to observe for GED scores and MSSIM due to the low
number of compiled samples (i.e. low CSR) for zero-shot prompting.
The effect of providing the schematic sheet size is negligible across
all the metrics between Prompt 4 to Prompt 5. We select Prompt 2
for fine-tuning as it performs well out of all the zero-shot prompts and
Prompt 4 due to the observed superiority in CSR. While, the effect
of providing sheet size is not observable from this result, we also
include Prompt 3 and 5 to examine how this affects the performance
of the fine-tuned model. Furthermore, we used Llama-3.1-8B instruct
model to leverage its knowledge base for text synthesis.
During fine-tuning, training and validation losses (Fig. 3) revealed
overfitting (an increase in validation loss) after epoch 3. Since the
cross entropy loss does not directly help quantify the correctness
of the code and the generated schematics, we use GED score and
MSSIM discussed in Section IV-D to evaluate the model performance
across training epochs. GED scores scaled by CSR for samples
containing different numbers of components from epoch 0 (raw
model of Llama-3.1-8B) to epoch 10 are shown in Fig. 4. We
scale the scores by CSR for a fairer comparison since samples that
are more difficult to generate (e.g. circuits with a large number
of components) can become compilable during fine-tuning and be
included in the calculation of the average score. This can prevent
the scores from improving because these larger circuits tend to
have lower scores than the smaller circuits as observed. The effect
of different prompting techniques is not observable while the fine-
tuning significantly improves the performance until the model overfits
around epoch 3, which aligns with the observation from Fig. 3.
Similarly, MSSIM scaled by CSR for different circuit size is shown
in Fig. 5. In contrast to the previous observations, the scores improve
until the last epoch. Furthermore, the impact of providing schematic
sheet size is clearly reflected by the scores that are almost one-order
of magnitude higher for (b) Prompt 3 and (d) Prompt 5 while one-
shot prompting did not impact the performance.
We then choose Llama-3.1-8B fine-tuned with Prompt 3 using
weights from epoch 6 as our best model and call it Schemato since it
demonstrated the best combination of scores in Fig. 4 and Fig. 5. We
evaluate this model by comparing its performance with the Llama-
3.1-8B baselines and GPT-4o, the latest and most powerful model
from OpenAI. Table I shows that Schemato outperforms the other
models with 1.8× and 4.3× higher GED score and MSSIM scaled
by CSR than the best performing pretrained models. Furthermore,
Schemato scores 76% CSR compared to 63% scored by the best
performing pretrained model. Table II shows the ablation study of
Llama-3.1-8B instruct fine-tuned with different prompts. There is no

## Page 6

Fig. 4: GED scores scaled by CSR evaluated with the test set. Llama-
3.1-8B is fine-tuned with (a) Prompt 2, (b) Prompt 3, (c) Prompt 4,
and (d) Prompt 5. The missing data points indicate that no generated
samples are compilable (i.e. CSR = 0). The number of components
in the test circuits is denoted by Nc and the number of reference
samples composed of Nc components is denoted by Ns.
Fig. 5: MSSIM scaled by CSR evaluated with the test set. Llama-
3.1-8B is fine-tuned with (a) Prompt 2, (b) Prompt 3, (c) Prompt 4,
and (d) Prompt 5. The missing data points indicate that no generated
samples are compilable (i.e. CSR = 0). The number of components
in the test circuits is denoted by Nc and the number of reference
samples composed of Nc components is denoted by Ns.
significant difference in the scores between the models fine-tuned
with the zero-shot and one-shot prompts. In contrast, the dominance
of the models fine-tuned with Prompt 3 (Schemato) and Prompt 5
in MSSIM and CSR showcases the effectiveness of including sheet
size in the prompt.
We show the reference and generated schematics for test circuit
examples of a transformer circuit and an RC band-pass filter in
Fig. 6 and Fig. 7 for different models. We note that GPT-4o failed
to produce a compilable code for the first example across all the
prompts. Although Schemato’s schematics differ geometrically from
the reference (MSSIM ̸= 1), they maintain identical topological fea-
tures (GED score = 1), demonstrating successful netlist-to-schematic
conversion and Schemato’s ability to generalize over different circuit
topologies.
VI. CONCLUSIONS & FUTURE WORK
In this paper, we introduced and evaluated Schemato, a fine-
tuned LLM designed for converting circuit netlists into inter-
pretable visual schematics. Our experimental results demonstrate that
Schemato significantly outperforms state-of-the-art LLMs, including
Llama-3.1-8B and GPT-4o, across all the graph-based, image-based,
compilation-based, and text-based metrics. These findings underscore
the potential of Schemato in reliably generating human-intuitive
schematics from netlists, paving the way for enhanced interpretability
and utility in ML-based analog circuit synthesis.
Our experiment in Fig. 4 show that for circuits with more than
5 components, Schemato struggles to generate schematics with
accurate connectivity. This is expected as there are more possible
variations of circuits with different connectivity and components for
these larger circuits, making it harder for Schemato to generalize.
The .asc file format further complicates generalization capabilities
T1 N002 0 N003 0 Td=50n Z0=50
V1 N001 0 V
RS N002 N001 R
RL N003 0 R
(a) Input netlist
T1
Td=50n Z0=50
V1
V
RS
R
RL
R
--- C:\Users\dematsur\Downloads\schematic_1_reference.asc ---
(b) Reference schematic
T1
Td=50n Z0=50
V1
V
RS
R
RL
R
--- C:\Users\dematsur\Downloads\schematic_1_generated.asc ---
(c) Schemato
V1
V
T1 C1
C
--- C:\Users\dematsur\Downloads\schematic_1_generated_llama_prompt5.asc ---
(d) Llama 3.1 instruct
with Prompt 5
Fig. 6: Netlist & LTSpice schematics of a transformer circuit
V1 N001 0 V
C1 N002 N001 C
R1 N002 Vout R
R2 Vout 0 R
C2 Vout 0 C
(a) Input netlist
V1
V
C1
C
R1
R
R2
R
C2
C
Vout
--- C:\Users\dematsur\Downloads\schematic_97_reference_prompt4.asc ---
(b) Reference schematic
V1
V
C1
C
R1
R
R2
R
C2
C
Vout
--- C:\Users\dematsur\Downloads\schematic_97_generated_prompt4.asc ---
(c) Schemato
V1
V
C1
C
C2
C
R1
R
R2
R
--- C:\Users\dematsur\Downloads\schematic_97_generated_llama_prompt5.asc ---
(d) Llama 3.1 instruct
with Prompt 5
V1
V
C1
C
R1
R
R2
R
C2
C
N002
N001
Vout
--- C:\Users\dematsur\Downloads\schematic_97_generated_gpt_prompt5.asc ---
(e) GPT-4o
with Prompt 5
Fig. 7: Netlist & LTSpice schematics of an RC band-pass filter
since it only specifies the components’ name and coordinate, but not
symbol size. This means that Schemato has to learn their symbol
size from how wires are placed around the components across our
training samples and therefore the model cannot generate schematics
containing less familiar or unfamiliar components. The analysis on
our dataset revealed that 5,227 out of the 9,907 training samples
contain components that appear less than 10 times in our training set.
Hence, more than half is composed of components of low familiarity.
This highlights the need for a larger dataset with circuits composed
of more generic and constrained set of components that are used in
various configurations.
To address this challenge, future improvements to Schemato will
focus on enriching the dataset using advanced data augmentation
techniques. For instance, we could combine two schematics into a
new one using a technique similar to mixup used in vision [32]. To
improve the generalizability of Schemato to large and unseen circuit
topologies, we will investigate the decomposition of larger circuits
into smaller, more manageable subcircuits, thereby enhancing the
ability of the model to learn hierarchical structures and the interplay
between subcircuits. In addition, we plan to further expand the dataset
by converting widely available schematic images into netlists and
digitized schematics, using a method similar to that implemented in
[33].

## Page 7

REFERENCES
[1] K. Kunal, T. Dhar, M. Madhusudan, J. Poojary, A. K. Sharma, W. Xu,
S. M. Burns, J. Hu, R. Harjani, and S. S. Sapatnekar, “GANA:
graph convolutional network based automated netlist annotation
for analog circuits,” in 2020 Design, Automation & Test in
Europe Conference & Exhibition, DATE 2020, Grenoble, France,
March 9-13, 2020. IEEE, 2020, pp. 55–60. [Online]. Available:
https://doi.org/10.23919/DATE48585.2020.9116329
[2] W. Cao, M. Benosman, X. Zhang, and R. Ma, “Domain knowledge-
infused deep learning for automated analog/radio-frequency circuit
parameter optimization,” in Proceedings of the 59th ACM/IEEE
Design Automation Conference, ser. DAC ’22. New York, NY, USA:
Association for Computing Machinery, 2022, p. 1015–1020. [Online].
Available: https://doi.org/10.1145/3489517.3530501
[3] K. Kunal, M. Madhusudan, A. K. Sharma, W. Xu, S. M. Burns,
R. Harjani, J. Hu, D. A. Kirkpatrick, and S. S. Sapatnekar, “Invited:
Align – open-source analog layout automation from the ground up,” in
2019 56th ACM/IEEE Design Automation Conference (DAC), 2019, pp.
1–4.
[4] Y. Yin, Y. Wang, B. Xu, and P. Li, “Ado-llm: Analog design
bayesian optimization with in-context learning of large language
models,” ArXiv, vol. abs/2406.18770, 2024. [Online]. Available:
https://api.semanticscholar.org/CorpusID:270765055
[5] W. Cao, M. Benosman, X. Zhang, and R. Ma, “Domain knowledge-
infused deep learning for automated analog/radio-frequency circuit
parameter optimization,” in Proceedings of the 59th ACM/IEEE
Design Automation Conference, ser. DAC ’22. New York, NY, USA:
Association for Computing Machinery, 2022, p. 1015–1020. [Online].
Available: https://doi.org/10.1145/3489517.3530501
[6] S. Uhlich, A. Bonetti, A. Venkitaraman, A. Momeni, R. Matsuo, C.-Y.
Hsieh, E. Ohbuchi, and L. Servadei, “GraCo – A graph composer for
integrated circuits,” arXiv preprint arXiv:2411.13890, 2024.
[7] K. Settaluri, A. Haj-Ali, Q. Huang, K. Hakhamaneshi, and B. Nikolic,
“Autockt: Deep reinforcement learning of analog circuit designs,” in
2020 Design, Automation & Test in Europe Conference & Exhibition,
DATE 2020, Grenoble, France, March 9-13, 2020. IEEE, 2020,
pp. 490–495. [Online]. Available: https://doi.org/10.23919/DATE48585.
2020.9116200
[8] Z. Zhao and L. Zhang, “Analog integrated circuit topology synthesis
with deep reinforcement learning,” IEEE Trans. Comput. Aided Des.
Integr. Circuits Syst., vol. 41, no. 12, pp. 5138–5151, 2022. [Online].
Available: https://doi.org/10.1109/TCAD.2022.3153437
[9] A. Devices, “Ltspice simulator,” accessed November 15,
2024. [Online]. Available: https://www.analog.com/en/resources/
design-tools-and-calculators/ltspice-simulator.html
[10] S. Schippers and J. J. Jones, “Xschem : schematic capture
and netlisting eda tool,” May 2025. [Online]. Available: https:
//nanohub.org/resources/xschem
[11] OpenAI and et al., “Gpt-4 technical report,” 2024. [Online]. Available:
https://arxiv.org/abs/2303.08774
[12] A. Dubey and et al., “The llama 3 herd of models,” 2024. [Online].
Available: https://arxiv.org/abs/2407.21783
[13] R. Qiu, G. L. Zhang, R. Drechsler, U. Schlichtmann, and B. Li,
“Autobench: Automatic testbench generation and evaluation using llms
for HDL design,” in Proceedings of the 2024 ACM/IEEE International
Symposium on Machine Learning for CAD, MLCAD 2024, Salt Lake
City, UT, USA, September 9-11, 2024, H. Amrouch, J. Hu, S. Garg,
and Y. Lin, Eds. ACM, 2024, pp. 18:1–18:10. [Online]. Available:
https://doi.org/10.1145/3670474.3685956
[14] P. Vijayaraghavan, A. Nitsure, C. Mackin, L. Shi, S. Ambrogio, A. Ha-
ran, V. Paruthi, A. Elzein, D. Coops, D. Beymer, T. Baldwin, and
E. Degan, “Chain-of-descriptions: Improving code llms for vhdl code
generation and summarization,” in 2024 ACM/IEEE 6th Symposium on
Machine Learning for CAD (MLCAD), 2024, pp. 1–10.
[15] A. Ayalasomayajula, R. Guo, J. Zhou, S. K. Saha, and F. Farahmandi,
“Lasp: Llm assisted security property generation for soc verification,”
in 2024 ACM/IEEE 6th Symposium on Machine Learning for CAD
(MLCAD), 2024, pp. 1–7.
[16] C. Chang, Y. Shen, S. Fan, J. Li, S. Zhang, N. Cao, Y. Chen,
and X. Zhang, “Lamagic: Language-model-based topology generation
for analog integrated circuits,” in Forty-first International Conference
on Machine Learning, ICML 2024, Vienna, Austria, July 21-27,
2024. OpenReview.net, 2024. [Online]. Available: https://openreview.
net/forum?id=MjGCD8wk1k
[17] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and P. Luo,
“Analogcoder: Analog circuit design via training-free code generation,”
2024.
[18] B. Naveen and K. Raghunathan, “An automatic netlist-to-schematic
generator,” IEEE Design & Test of Computers, vol. 10, no. 1, pp. 36–41,
1993.
[19] C. Lageweg, “Designing an automatic schematic generator for a netlist
description,” Delft University of Technology, Delft, Netherlands, Tech.
Rep, 1998.
[20] B. Arsintescu, “A method for analog circuits visualization,” in Proceed-
ings of the International Conference on Computer Design (ICCD), 1996,
pp. 454–459.
[21] Lee and McNamee, “Aesthetic routing for transistor schematics,” in 1992
IEEE/ACM International Conference on Computer-Aided Design. IEEE,
1992, pp. 35–38.
[22] H.-Y. Hsu and M. P.-H. Lin, “Automatic analog schematic diagram
generation based on building block classification and reinforcement
learning,” in Proceedings of the 2022 ACM/IEEE Workshop on Machine
Learning for CAD, 2022, pp. 43–48.
[23] J. Wei and X. al., “Chain of thought prompting elicits reasoning in
large language models,” CoRR, vol. abs/2201.11903, 2022. [Online].
Available: https://arxiv.org/abs/2201.11903
[24] torchtune maintainers and contributors, “torchtune: Pytorch’s finetuning
library,” Apr. 2024. [Online]. Available: https//github.com/pytorch/
torchtune
[25] Y. Zhao, A. Gu, R. Varma, L. Luo, C.-C. Huang, M. Xu, L. Wright,
H. Shojanazeri, M. Ott, S. Shleifer, A. Desmaison, C. Balioglu,
P. Damania, B. Nguyen, G. Chauhan, Y. Hao, A. Mathews, and S. Li,
“Pytorch fsdp: Experiences on scaling fully sharded data parallel,”
2023. [Online]. Available: https://arxiv.org/abs/2304.11277
[26] A. Robles-Kelly and E. Hancock, “String edit distance, random walks
and graph matching,” 02 2002, pp. 107–129.
[27] A. A. Hagberg, D. A. Schult, and P. J. Swart, “Exploring network
structure, dynamics, and function using networkx,” in Proceedings of
the 7th Python in Science Conference, G. Varoquaux, T. Vaught, and
J. Millman, Eds., Pasadena, CA USA, 2008, pp. 11 – 15.
[28] Z. Abu-Aisheh, R. Raveaux, J.-Y. Ramel, and P. Martineau, “An
Exact Graph Edit Distance Algorithm for Solving Pattern Recognition
Problems,” in 4th International Conference on Pattern Recognition
Applications and Methods 2015, Lisbon, Portugal, Jan. 2015. [Online].
Available: https://hal.science/hal-01168816
[29] Z. Wang, A. Bovik, H. Sheikh, and E. Simoncelli, “Image quality assess-
ment: from error visibility to structural similarity,” IEEE Transactions
on Image Processing, vol. 13, no. 4, pp. 600–612, 2004.
[30] K. Papineni, S. Roukos, T. Ward, and W.-J. Zhu, “Bleu: a method
for automatic evaluation of machine translation,” in Proceedings of the
40th Annual Meeting on Association for Computational Linguistics, ser.
ACL ’02. USA: Association for Computational Linguistics, 2002, p.
311–318. [Online]. Available: https://doi.org/10.3115/1073083.1073135
[31] S. Ren, D. Guo, S. Lu, L. Zhou, S. Liu, D. Tang, N. Sundaresan,
M. Zhou, A. Blanco, and S. Ma, “Codebleu: a method for
automatic evaluation of code synthesis,” 2020. [Online]. Available:
https://arxiv.org/abs/2009.10297
[32] H. Zhang, M. Cisse, Y. N. Dauphin, and D. Lopez-Paz, “mixup:
Beyond empirical risk minimization,” 2018. [Online]. Available:
https://arxiv.org/abs/1710.09412
[33] Y. Shi, Z. Tao, Y. Gao, L. Huang, H. Wang, Z. Yu, T.-J. Lin, and
L. He, “Amsnet 2.0: A large ams database with ai segmentation for net
detection,” 2025. [Online]. Available: https://arxiv.org/abs/2505.09155

