---
title: "AI Analog Circuit Design Agents: On Knowledge Extraction and Transfer with Knowledge Graphs"
authors: "Karthik Somayaji N.S.; Peng Li"
date: "2025-10-28"
category: "data_knowledge"
source_url: "https://doi.org/10.1109/ICCAD66269.2025.11240770"
pdf: "papers/2025_ai-analog-circuit-design-agents_10-1109-iccad66269-2025-11240770.pdf"
sha256: "176139ca9538523d04cec9729e359d9b2d072661560b8f90323fccaa00658d31"
pages_extracted: 9
---

# AI Analog Circuit Design Agents: On Knowledge Extraction and Transfer with Knowledge Graphs

Source PDF text extracted with pdftotext -raw; page breaks are retained. Figures and equations should be checked against the PDF.

## Page 1

AI Analog Circuit Design Agents - On Knowledge
Extraction and Transfer with Knowledge Graphs
Karthik Somayaji N.S
Dept. of Electrical and Computer Engineering
University of California, Santa Barbara
karthi@ucsb.edu
Peng Li
Dept. of Electrical and Computer Engineering
University of California, Santa Barbara
lip@ucsb.edu
Abstract—Emulating expert human designers’ cognitive strate-
gies is a long-standing aspiration in analog design automation. A
persistent challenge is achieving generalizability, speciﬁcally, how
insights from optimizing certain circuits can effectively inform
the design of related circuits with differing topologies. For the
ﬁrst time, we identify required key components towards en-
abling AI-based topology-agnostic design knowledge transfer and
demonstrate how such methodology can be developed for analog
sizing optimization utilizing large language models (LLMs). The
proposed framework, called Analog Design Optimization with
Knowledge Transfer (ADO-KT) consists of three key integrated
steps: 1) LLM-enabled knowledge graph (KG) generation for
structured extraction of prior analog design knowledge, 2) LLM-
enabled knowledge graph reﬁnement for quality KG generation,
and 3) topology-agnostic knowledge transfer for LLM-assisted
optimization of new circuits. ADO-KT enables seamless summa-
rization and transfer of quality knowledge ﬂow between circuits
of varying types and topologies, effectively mirroring human de-
signers’ strategies. Empirical evaluations demonstrate signiﬁcant
improvements in design quality and data efﬁciency, highlighting
the potential of the proposed approach for automated analog
circuit design with generalized knowledge reuse.
Index Terms—analog design, LLM assisted design, knowledge
graphs, knowledge graph ﬁltering
I. INTRODUCTION
Analog circuit design remains central to modern integrated
circuits (ICs), directly impacting performance, power efﬁ-
ciency, and reliability. Unlike digital design, analog design
demands careful navigation of continuous, interdependent re-
lationships between circuit topologies, design parameters, and
performance metrics [1], [2]. Traditionally, this process has
relied heavily on human expertise, with designers drawing
upon accumulated experience and intuition to optimize com-
plex analog blocks.
Recent progress in analog design automation has introduced
algorithmic approaches such as genetic algorithms [1], [3],
Bayesian Optimization (BO) [2], [4], reinforcement learning
(RL) [5], [6], [7], [8] and LLM based solutions [9], [10],
[11], [12] to explore design spaces more systematically. In
particular, GCN-RL [6] enables knowledge transfer across
circuits by leveraging graph convolutional networks within
an RL framework. However, GCN-RL relies on black-box
RL optimization rather than interpretable, cognitive reasoning
via language models. Its transfer is limited to circuits with
similar functionalities and assumes architectural compatibility
between the circuit graph and the GCN structure, making
it difﬁcult to generalize across circuits with signiﬁcantly
differing or more complex topologies.
This motivates a central research question: How can design
knowledge be effectively transferred across analog circuits that
share only partial structural or functional similarities?
Consider the task of optimizing a low dropout regulator
(LDO), which incorporates ampliﬁer sub-blocks. Existing ana-
log EDA techniques typically require an exact architectural
match with a previously optimized LDO to enable knowledge
reuse. In contrast, human designers often draw on insights
from prior sub-block-level designs—such as ampliﬁers—even
when there is only partial topological or functional alignment.
This ﬂexible reuse of design experience reﬂects the adaptive
reasoning of expert engineers and highlights a fundamental
limitation of existing automated methods.
To emulate this ﬂexibility, we present the ﬁrst work on
analog sizing optimization with topology-agnostic knowledge
transfer. In the proposed framework called Analog Design
Optimization with Knowledge Transfer (ADO-KT), we es-
tablish a structured knowledge representation that captures
generalizable design insights beyond rigid topology matching.
A desirable framework should encode the complex causal
relationships among performance metrics, substructures, and
design parameters—an area where knowledge graphs (KGs)
have shown promise across domains involving causal rea-
soning [13], [14]. Motivated by this, we introduce a hi-
erarchical KG-based formalism to represent analog design
knowledge. Since manually constructing analog-speciﬁc KGs
is impractical, we develop an automated methodology that uses
hierarchical prompting with Large Language Models (LLMs)
to generate these representations efﬁciently.
Yet, generating KGs is only part of the solution. To integrate
structured knowledge into the optimization of new circuits,
we embed hierarchical KGs into a hybrid LLM and Bayesian
optimization (BO) loop. This allows this LLM-assisted opti-
mization process to dynamically incorporate prior knowledge
from structurally or functionally related circuits, guiding the
search for optimal design points. Unlike recent LLM-based
approaches such as [9], [10], which treat each circuit opti-
mization task independently, our framework explicitly supports
cross-circuit knowledge reuse via structured context injection.
To ensure robustness in knowledge extraction, we employ
consistency-based ﬁltering to discard unreliable or contradic-
tory knowledge entries, enhancing trust in LLM-generated
KGs. Our ADO-KT approach enables practical, generalizable
knowledge transfer across circuits without requiring architec-
tural equivalence—closely emulating expert design intuition.
AI Analog Circuit Design Agents : On Knowledge
Extraction and Transfer with Knowledge Graphs
2025
IEEE/ACM
International
Conference
On
Computer
Aided
Design
(ICCAD)
|
979-8-3315-1560-7/25/$31.00
©2025
IEEE
|
DOI:
10.1109/ICCAD66269.2025.11240770
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 2

K
Knowledge
Extraction
ǥ ǥ ǥ
Knowledge
Graph
Delete spurious
entries
ǥ ǥ ǥ
Refined Knowledge Graph
Circuit to be optimized
Different topology
with similar
sub-structures
Prior circuit
Inject Knowledge
Graphs into LLM
assisted optimization
Knowledge Graph + LLM assisted
Analog Optimization Engine
Delete sp rio s
Knowledge
Refinement
Inject Knowledge
Knowledge
Transfer
Fig. 1: How can we transfer knowledge between circuits with different topologies ? Our methodology consists of constructing
Knowledge Representations for related circuits using knowledge graphs, reﬁning such knowledge graphs using our Knowledge
Reﬁnement mechanism and ﬁnally using a LLM assisted optimization framework for Knowledge Transfer
.
Our key contributions include:
1) Structured Knowledge Extraction: A hierarchical
LLM-based prompting approach to systematically gen-
erate analog-speciﬁc knowledge graphs (KG) encoding
essential causal relationships.
2) Robust Knowledge Graph Reﬁnement: Consistency-
driven ﬁltering techniques ensuring reliability and preci-
sion of generated knowledge graphs.
3) Topology-Agnostic Knowledge Transfer: A knowledge
graph infused hybrid BO-LLM optimization framework
that integrates structured knowledge across varying types
of circuits, emulating human-like knowledge transfer
ﬂexibility.
The proposed ADO-KT framework is illustrated in Figure
1. Extensive experiments conﬁrm substantial improvements in
design quality and sample efﬁciency, highlighting the effec-
tiveness of structured knowledge representation and transfer
in automated analog circuit optimization.
II. TRANSFERRABLE STRUCTURAL ANALOG KNOWLEDGE
REPRESENTATION
A. Structural Analog Design Knowledge
Given a circuit topology, analog circuit design involves
selecting and tuning design parameters (sizing values) while
trading off among multiple competing performance metrics
(design objectives). Design knowledge, in this context, refers
to a structured understanding of the causal interdependen-
cies among design parameters, subcircuit structures, and the
performance metrics. This knowledge can be systematically
organized along three key dimensions:
(1) Interactions between Performance Metrics: A key
challenge in analog design stems from its multi-objective
nature. For this, understanding the intrinsic relationships
between various performances such as trade-offs or di-
rect proportionality is essential. Such relationships offer
reasoning about the impact of improving one metric on
others.
(2) Dependence on Topology: Speciﬁc sub-structures within
the circuit topology tend to exert dominant inﬂuence
over certain performance metrics. Recognizing these as-
sociations enables a principled understanding of how
topology of sub-circuits shapes circuit behavior through
certain performance metrics. Since our goal is to enable
topology-agnostic knowledge transfer across circuits, it
is essential to capture how individual sub-circuit blocks
inﬂuence performance. This is particularly motivated by
the observation that many analog circuits share recurring
substructures, allowing such localized knowledge to gen-
eralize across different larger circuit topologies.
(3) Structure in Design Parameters: Design parameters
can be categorized based on their association with sub-
structures. The inﬂuence of each parameter on perfor-
mance is typically mediated through the sub-structure to
which it belongs to.
These structural perspectives form the basis for systemat-
ically representing and reasoning about the causal nature of
analog design knowledge.
B. Exacting Design Knowledge Using Knowledge Graphs
Inspired by the inherently causal nature of analog de-
sign processes, we propose using causal graphs, speciﬁcally,
Knowledge Graphs (KGs) [15]–[17] to explicitly capture the
complex interdependencies among performance metrics, cir-
cuit substructures, or design parameters. Formally, a KG for
a given circuit Ck is deﬁned as:
Gk = (Vk, Ek), (1)
where Vk denotes nodes representing key entities such as
performance metrics Pi ∈ P, circuit substructures Si ∈ S,
and design parameters xi ∈ X. Directed edges Ek ⊆ Vk × Vk
characterize relationships between these entities. Each edge
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 3

ऑचG
Graph
Gain Bandwidth
Offset
Voltage
Hysteresis
Error
Ambiguous
relationship
Trade-off
Directly proportional
Directly
proportional
Differential
Pair
Load
Stage
Gain Bandwidth
Offset
Voltage
Hysteresis
Error
Current
Mirror
Influences
ऑचࡿGraph
Hysteresis Comparator
ऑचࡿࢄGraph
w1 ǥ
Belongs to
Directly
proportional
Inversely
proportional
Inversely
proportional
Inversely
proportional
Inversely
proportional
Differential
Pair
Current
Mirror
w7 w11 w4
Gain Bandwidth Offset
Voltage
Hysteresis
Error
Influences
Belongs to Belongs to
Belongs to
Fig. 2: Pictorial representation of some sample entries of the KG for the Hysteresis Comparator generated by LLM (MKG:
LLama3-70B-instruct) using the proposed hierarchical approach.
e = (vi, r, vj) ∈ Ek connects nodes vi and vj, with r describ-
ing the relationship type. This structured representation offers
an efﬁcient way for interpreting and representing structural and
functional knowledge integral to the analog design process.
C. Hierarchical Analog Design Knowledge Graphs: Overview
The absence of readily available analog-speciﬁc knowledge
graphs, coupled with the substantial manual effort required
for their construction, presents a key bottleneck in knowledge
transfer driven design. To address this, we introduce a hier-
archical methodology that leverages Large Language Models
(LLMs) to automatically generate analog design knowledge
graphs (KGs). An illustration of our framework to construct
KGs is shown in Figure 3.
Our approach constructs the KG in a staged, hierarchical
fashion—progressively capturing the relationships among per-
formance metrics, circuit substructures, and design parameters.
A representative example of some sample entries of the
resulting KG generated by our approach using a sample LLM
(LLama3-70B-instruct) is illustrated in Figure 2. The KG is
constructed sequentially and consists of the following three
subgraphs:
(1) Performance Metric Graph (GP ): Captures the interac-
tions among performance metrics P, such as trade-offs
and proportional dependencies. For instance, in Figure 2,
GP identiﬁes a trade-off relationship between gain and
bandwidth.
(2) Performance–Structure Graph (GPS): Maps each per-
formance metric to the substructures S that predomi-
nantly inﬂuence it. As shown in the example, the differen-
tial pair signiﬁcantly inﬂuences both gain and bandwidth.
(3) Performance–Structure–Parameter Graph (GPSX ):
Encodes how design parameters X, associated with spe-
ciﬁc substructures, affect the corresponding performance
metrics. For example, parameter w1 within the differential
pair exhibits a direct proportional relationship to the gain.
The complete knowledge graph Gk for a circuit Ck is
formed by unifying the three subgraphs:
Gk = GP ∪ GPS ∪ GPSX .
This hierarchical construction enables a structured and in-
terpretable representation of analog design knowledge, serving
as a foundation for interpretable knowledge transfer.
Our approach constructs the KG for a given circuit Ck by
querying a language model MKG using a hierarchical querying
procedure. Each sub-graph is generated sequentially, with each
stage leveraging the results of the previous one, along with a
shared circuit context. This process is expressed as:
GP ← MKG(circuit context, GP query)
GPS ← MKG(circuit context, GP , GPS query)
GPSX ← MKG(circuit context, GP , GPS, GPSX query)
We next describe the circuit context and the hierarchical
querying procedure used for generating the sub-graphs.
D. Circuit Context
To enable context-aware generation, we ﬁrst provide the
LLM MKG with the netlist of circuit Ck, which conveys the
circuit’s topological structure. Additionally, we supply repre-
sentative design instances obtained from prior optimization
runs, which reveal empirical relationships between perfor-
mance metrics and design parameters. We also provide the
speciﬁcations that were provided for the previous optimization
runs of the circuit (for example - Gain > 40dB, hysteresis
error < 0.2V etc). Together, this information along with the
hierarchical queries allows the model to infer the underlying
causal interactions that govern circuit behavior. This enables
the generation of interpretable design knowledge.
E. Constructing the GP Graph
The ﬁrst step in our hierarchical KG generation process in
ADO-KT is the construction of the performance metric graph
GP . In this graph, nodes correspond to the performance metrics
P relevant to the given circuit Ck. The goal is to capture the
qualitative relationships among the metrics. The relationships
permitted between performance metric nodes include:
• trade-off — an improvement in one metric is typi-
cally accompanied by degradation in another,
• directly proportional — both metrics tend to
improve or degrade together,
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 4

G
Given the P-Graph and P-S Graph, identify relationships between design
parameters X, circuit substructures S, and performance metrics P. Output
entries of the form (Parameter, Relationship, Substructure) where
Relationship is ‘belongs-to’ if a parameter belongs to a sub-structure.
Next output entries of the form (Parameter, Relationship, Metric) where
Relationship can be ‘directly-proportional’ if increase in parameter
value increases metric value and vice versa or ‘inversely-proportional’
if increase in parameter value decreases metric value and vice versa.
ऑचझढ-Graph Query
Identify performance metrics from the specifications. Next identify
relationships among these performance metrics of the circuit. Output
entries of the form (Metric A, Relationship, Metric B), where Relationship
can only be of type ‘trade-off’, ‘directly-proportional’, ‘ambiguous’.
ऑच-Graph Query
Given the P-graph, identify the circuit substructures that influence
performance metrics in the P-graph. Output tuples of the form
(Substructure, Relationship, Metric), to connect circuit sub-structures
with performance metrics. The Relationship in the tuple can only be of
type ‘influences’ if the substructure significantly influences a
performance metric.
ऑचझ-Graph Query
ऑच-Graph
ऑचࡿ-Graph
ऑचࡿࢄ-Graph
Hierarchical Queries
c
LLM ࣧ୏ୋ
p
LLM ࣧ୏ୋ
LLM ࣧ୏ୋ
(Gain, trade-off, GBW)
(CMRR, trade-off, GBW)
. . .
(Load,influences, Pow)
(Load,influences, Gain)
. . .
(l1,belongs-to, load)
(l1, directly-prop, Pow)
. . .
Generated Sub-Graphs
Netlist . . .
Specifications . . .
Representative Design Samples:
{(ࢄ૚ǡ ࡼ૚), ...}
Circuit Context
Fig. 3: Hierarchical Generation of the Knowledge Graph
• ambiguous relationship — the relationship be-
tween the metrics is unclear or context-dependent.
To construct GP , we prompt the LLM MKG with the GP -
Graph query (shown in Figure 3) to ﬁrst identify the relevant
performance metrics from the circuit context and design spec-
iﬁcations - and then determine pairwise relationships between
them using only the predeﬁned categories above.
F. Constructing the GPS Graph
Given the constructed GP , we proceed to build the perfor-
mance–structure graph GPS. In addition to the performance
metrics from GP , this graph includes new nodes correspond-
ing to the circuit substructures S, which are automatically
extracted from the circuit netlist.
The possible relationship in this graph is deﬁned as:
• influences — indicating that a particular substructure
has a signiﬁcant impact on a speciﬁc performance metric.
The LLM MKG is queried using the GPS Graph query
(shown in Figure 3) to identify additional nodes that are a
substructure of the circuit topology and link them to relevant
performance metrics using the influences relationship.
G. Constructing the GPSX Graph
The ﬁnal stage involves constructing the perfor-
mance–structure–parameter graph GPSX , using GP and
GPS sub-graphs as inputs. This graph augments the existing
nodes with design parameters X, such as transistor widths
and lengths which are also extracted from the circuit netlist.
Here two types of relationships between nodes are introduced:
• belongs to — connecting a design parameter to the
substructure it is associated with,
• directly proportional or inversely
proportional — expressing the nature of the
design parameter’s inﬂuence on a performance metric.
Here, the LLM MKG is queried using the GPSX Graph
query (shown in Figure 3) to identify the relevant design
parameters and establish both structural relationships (pa-
rameter–substructure) and functional relationships (parame-
ter–performance metric).
III. KNOWLEDGE TRANSFER FOR NEW CIRCUIT
OPTIMIZATION WITH ADO-KT
A core motivation for emulating human design strategies
is the ability to reuse knowledge from one circuit to guide
the optimization of another, even when their topologies differ.
While knowledge transfer can be facilitated by feeding our
extracted analog design knowledge graphs into any suitable
LLM-based design agent, we have found that the recently
developed hybrid Bayesian Optimization and LLM framework
ADO-LLM [9] is a good choice. ADO-LLM combines the
principled exploration of BO with the zero/few shot-learning
capabilities of LLMs. BO proposes new design points us-
ing a built-in Gaussian process (GP) model with quantiﬁed
uncertainty while the LLM, referred to as a backbone-LLM
Mbackbone generates additional suggestions based on circuit
speciﬁcations and few-shot examples. These components work
in tandem to optimize a given new circuit using a fairly small
amount of circuit-speciﬁc simulation data.
Although Mbackbone offers broad general-purpose knowl-
edge, it lacks the circuit-speciﬁc insights necessary for trans-
ferring optimization strategies across different circuit topolo-
gies—particularly at the level of sub-blocks or substructures.
To bridge this gap, we enrich the LLM’s context with struc-
tured knowledge graphs (KGs) derived from previously opti-
mized, structurally related circuits. These KGs encode causal
relationships among performance metrics, substructures, and
design parameters, enabling effective knowledge transfer even
in the absence of exact topology matches.
At each optimization iteration in ADO-KT the backbone-
LLM Mbackbone is prompted with a combined context com-
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 5

prising: (i) the circuit netlist and speciﬁcations, (ii) best per-
forming design points, and (iii) KGs {Gi} from related cir-
cuits. This enriched context in ADO-KT enables Mbackbone
to generate design suggestions that are grounded in the
current task while beneﬁting from transferable insights.
We illustrate the ADO-KT framework in Algorithm 1. This
KG-infused BO-LLM framework mirrors the way human
designers apply experience across designs, signiﬁcantly en-
hancing both sample efﬁciency and design point quality.
Algorithm 1 Knowledge Transfer using ADO-KT
Input: Target circuit Ck, Knowledge graphs from related cir-
cuits {G1, G2, . . . , Gk−1}, backbone-LLM: Mbackbone, KG-
generative LLM model: MKG
Output: Optimized design parameters x∗
k and KG Gk
a. Initialize dataset Dk = {(xj, f(xj))}n
j=1 with n evaluated
design points for circuit Ck
b. Construct combined context Ck for the backbone-LLM,
consisting of:
• Circuit netlist and speciﬁcations for Ck
• Best few-shot examples from Dk
• Knowledge graphs {Gi}k−1
i=1 from related circuits
c. Repeat until convergence or budget is exhausted:
i. Use GP-based BO to propose design point xBO
ii. Get xLLM ← Mbackbone(Ck)
iii. Simulate f(xBO) and f(xLLM), evaluate performance.
iv. Update dataset:
Dk ← Dk ∪ {(xBO, f(xBO)), (xLLM, f(xLLM))}
d. Return best design point:
x∗
k = arg max
x∈Dk
f(x)
e. Return knowledge graph Gk obtained from MKG
IV. ANALOG KNOWLEDGE REFINEMENT
While knowledge graphs generated by LLMs offer a pow-
erful means of representing design knowledge, they may
occasionally contain inconsistencies or ambiguous entries due
to the underlying LLM’s generation process. The adopted
hybrid ADO-LLM approach mitigates potential hallucinations
of the LLM [9] because of incorporation of the Bayesian
optimization agent, which also reduces the negative inﬂuences
of imprecise knowledge graphs. Nevertheless, the quality of
knowledge directly still impacts the effectiveness of knowl-
edge transfer. To address this, we propose a sub-graph-wise
reﬁnement technique to reﬁne the knowledge graph before it
is used in the ADO-KT framework for knowledge transfer. We
detail the knowledge reﬁnement mechanism in Algorithm 2
and illustrate it in Figure 4.
Our method aims to quantify the uncertainty of the LLM-
enabled KG generation process. It begins by generating mul-
tiple knowledge graphs for a given circuit using repeated
LLM MKG invocations, each consisting of a GP -Graph, GPS-
Graph, and GPSX -Graph. One of these KGs is arbitrarily des-
ignated as the anchor KG, while the remaining KGs serve as
reference KGs. We then evaluate the consistency of each entry
in the anchor KG against matching entries in the reference
Algorithm 2 Knowledge Reﬁnement
Input: Generated KGs {G1
k, . . . , GM
k } for circuit Ck, rejection
threshold τ
Output: Reﬁned KG Greﬁned
k
a. Select one KG Ga as the anchor KG; treat the others
{Gr} = {G1
k, . . . , GM
k } \ {Ga} as reference KGs.
b. For each sub-graph g ∈ GP , GPS, GPSX :
i. For each entry e ∈ g:
a. Initialize contradiction_count ← 0, and
match_count m ← 0.
b. For each reference KG Gr ∈ {Gr}:
– For each entry er ∈ Gg
r:
• If er and e have matching entities:
– Increment m ← m + 1
– If e contradicts er:
∗ Increment contradiction_count
c. Compute rejection rate:
γ = contradiction_count/m (if m > 0)
d. If γ > τ, discard entry e
c. Return the reﬁned KG Greﬁned
k containing surviving entries
from Ga
KGs. If an anchor entry frequently contradicts corresponding
entries in the references, i.e., it has a rejection rate γ greater
than a rejection threshold τ, it is ﬂagged as unreliable and is
discarded.
This ﬁltering step systematically removes inconsistent en-
tries from the KG. The reﬁned KG is subsequently used as part
of ADO-KT to ensure that only high-conﬁdence knowledge
informs the design suggestions. We illustrate the average
rejection rate of the knowledge entries generated by different
LLMs (MKG) for different circuits in Figure 5. From our
knowledge reﬁnement mechanism, we ﬁnd that some LLMs
generate initial consistent KGs (evident by smaller rejection
percentages), while other smaller LLMs (with less than 70 bil-
lion parameters) may generate inconsistent KG entries within
the KG (evidenced by higher rejection percentages).
V. EXPERIMENTAL EVALUATION
The objective of the developed topology-agnostic knowl-
edge transfer method ADO-KT is twofold: (i) improve ﬁnal
design quality, and (ii) enhance data efﬁciency by accelerating
convergence. We evaluate our framework on ﬁve analog cir-
cuits shown in Fig. 6), which demonstrates effective structural
knowledge transfer across different types of circuits with
varying topologies.
We benchmark against two baselines: (i) GP-based Bayesian
Optimization (GP-BO) [2], and (ii) ADO-LLM [9], which
applies BO-LLM optimization without knowledge transfer.
Performance and sample efﬁciency are assessed using both
the ﬁnal Figure of Merit (FOM), computed as a weighted
combination of normalized metrics, and its evolution as a
function of data consumption respectively.
Each optimization begins with 5 initial zero-shot design
points from the backbone LLM Mbackbone, followed by 20
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 6

R
Rejection Rate ࢽ :
࢔࢕Ǥ࢕ࢌࢉ࢕࢔࢚࢘ࢇࢊ࢏ࢉ࢚࢏࢕࢔࢙
࢔࢕Ǥ࢕ࢌ࢓ࢇ࢚ࢉࢎࢋ࢙
Agreement b/w entries Contradiction b/w entries
ǥ
Anchor KG ࡳࢇ
Reference KG 1
(Gain, trade-off, CMRR)
Reference KG N
Ref: (Gain, trade-off, CMRR) Ref:(Gain, directly-prop, CMRR)
LLM ग۹۵
Generate
multiple
KGs
Select Anchor
and reference
KGs
Reference KGs Anchor KG ࡳࢇ Refined KG
Filtering
Filtering
ࢌ
Reject entry if rejection rate > ૌ
Refined KG
ǥ ǥ
ǥ ǥ ǥ ǥ
(Gain, directly-prop, CMRR)
Anchor: (Gain, trade-off, CMRR)
(Gain, trade-off, CMRR)
(GBW, trade-off, CMRR)
Rejected entry
(Gain, trade-off, CMRR)
Anchor: (Gain, trade-off, CMRR)
Fig. 4: Knowledge Reﬁnement Framework: Every entry in anchor KG is checked for consistency with every matching entry
of each reference KG.
Fig. 5: Average rejection rate of the knowledge entries for
different LLMs (MKG) for each circuit example
iterations where one candidate is proposed by Mbackbone and
one by BO, totaling 45 simulations per circuit (65 for the
Hysteresis Comparator with 30 iterations).
All the experiments with ADO-LLM [9] and ADO-KT we
use Mbackbone : GPT-3.5 [18] in the BO-LLM optimization
loop. However, in order to generate KG, in ADO-KT, we
employ a different LLM (MKG). To show the generality of
the proposed techniques, we employ six different LLMs as
candidates for (MKG) and tabulate optimization results in
Tables I, II, III for each circuit. We use DeepSeek-R1-Distill-
Llama-70B [19], DeepSeek-R1-Distill-Qwen-32B [19], GPT-
4o [20], llama3-70B-Instruct [21], Mistral-Small-24B-Instruct-
2501 [22] and Qwen2.5-32B-Instruct [23] as different choices
for MKG to generate the KG. We also compare performance
with and without knowledge reﬁnement to assess the role of
KG consistency.
For knowledge reﬁnement, we apply a conservative rejection
threshold of τ = 0.3. We begin with the two-stage differential
ampliﬁer (TSDA) to establish a baseline. As no prior circuits
precede it, we optimize it using the ADO-LLM framework
without knowledge transfer and generate its knowledge graph
thereafter using LLM (MKG). Subsequent experiments eval-
uate topology-agnostic transfer across circuits.
A. Folded Cascode Ampliﬁer (FC)
The folded cascode ampliﬁer, which comprises 32 design
parameters, contains functional substructures that are similar
but not identical to those of the two-stage differential ampliﬁer
(TSDA). We inject the KG of the differential ampliﬁer into
the optimization process for the folded cascode circuit whose
design specs are: gain > 60 dB, CMRR > 80 dB, and UGF
> 1 MHz. As shown in Table I, knowledge transfer using
KGs with or without reﬁnement improves overall performance
while the proposed KG reﬁnement leads to more metrics
meeting speciﬁcation constraints and achieves even greater
FOM improvements compared to ADO-LLM.
B. Low Dropout Regulator (LDO)
We evaluate our framework on a low dropout regulator
(LDO) tasked with maintaining a stable 0.6 V output. The
circuit spans 40 design parameters across 20 transistors and
includes substructures such as error ampliﬁers and biasing
blocks, similar to both the two-stage differential ampliﬁer and
the hysteresis comparator. The design constraints require a
quiescent current below 12 mA and a regulated voltage error
within 0.1 V of the target. As shown in Table II, KG-based
transfer improves the FOM over ADO-LLM, with reﬁned KGs
yielding further gains.
C. Hysteresis Comparator (Comp)
The hysteresis comparator, used in high-speed switching ap-
plications, consists of 24 design parameters with the following
design speciﬁcations: gain > 40 dB, bandwidth > 0.1 MHz,
offset error < 20 mV, and hysteresis error < 0.4 V. It contains
key substructures—such as gain stages and biasing blocks
which also appear in the two-stage differential ampliﬁer. The
structured KGs derived from the differential ampliﬁer are
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 7

Vout
Vref
Iload
Vin
C0
R1
R2
C3
C2
C1
Ib
M1
M2
M12
M13
M11
M17
M16
M14
M15
M20
M19 M18
M3
M4
M5
M6
M9
M7
M10
M8
Error Amplifier
Biasing
Inverting
Amplifier Output Stage
(a) Low Dropout Regulator
clk
Vref
Vout
Iload
RL
CL
R0 L
M1
Ib
M2
M3
M4
M5
M6
M7
M8
M9
M10
Vin
M11
M13
M12
M14
M15
M16
M17
M18
M19
M20
M21
M22
Error Amplifier
Biasing Comparator Buffer Output Stage
(b) DCDC Converter
M1 M2
M3 M4
M5
M6
M7
Cload
vout
vin+ vin-
Rz Cc
Mb
Ibias
VDD
(c) Two Stage Diff Ampliﬁer
Mb1
Cload
vout
VDD
vin+ vin-
Ibias
VSS
R1 R2
M1 M2
M3
M4 M5
M7
M6
M8
M9
M11
M10
Mb4
Mb2
Mb3
(d) Folded Cascode Ampliﬁer
Mb
Cload
vout
VDD
Ibias
VSS
M1
vin+ vin-
M2
M3 M4
M5
M6 M7
M8 M9
M10 M11
(e) Hysteresis Comparator
Fig. 6: Circuit Schematics.
TABLE I: Folded Cascode Ampliﬁer Optimization Results
Method Gain ↑ (Spec: > 60dB) CMRR ↑ (Spec: >80dB) GBW ↑ (Spec: >1MHz) Power ↓ (Spec: <50 μW) FOM ↑
GP-BO 4.96 55.54 1.439 65.86 -0.200
ADO-LLM 43.16 73.87 1.786 306.13 -0.102
ADO-KT : Knowledge Transfer (from TSDA) without Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 61.69 69.89 2.347 60.79 0.351
DeepSeek-R1-Distill-Qwen-32B 43.153 80.53 2.131 50.23 0.214
GPT-4o 65.58 89.79 2.054 54.12 0.364
Llama3-70B-instruct 66.19 88.41 2.154 54.12 0.380
Mistral-Small-24B-Instruct-2501 28.25 47.34 1.612 37.45 -0.018
Qwen2.5-32B-Instruct 45.24 96.42 1.726 259.6 -0.043
ADO-KT : Knowledge Transfer (from TSDA) with Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 65.18 87.84 2.215 55.27 0.389
DeepSeek-R1-Distill-Qwen-32B 68.26 91.55 1.934 52.06 0.352
GPT-4o 66.75 89.89 2.132 54.04 0.386
Llama3-70B-instruct 64.49 87.23 2.230 56.51 0.388
Mistral-Small-24B-Instruct-2501 61.80 84.25 2.284 59.98 0.382
Qwen2.5-32B-Instruct 60.617 91.38 1.921 54.09 0.321
TABLE II: LDO Optimization Results
Method Qui Curr ↓ (Spec: < 12mA) Reg. Voltage Error ↓ (Spec: < 0.1V) FOM ↑
BO 5.018 0.466 0.064
ADO-LLM 5.011 0.457 0.055
ADO-KT : Knowledge Transfer (from TSDA, Comp) without Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 13.57 0.028 0.294
DeepSeek-R1-Distill-Qwen-32B 5.011 0.417 0.089
GPT-4o 30.66 0.068 0.166
Llama3-70B-instruct 10.05 0.0169 0.340
Mistral-Small-24B-Instruct-2501 5.011 0.457 0.055
Qwen2.5-32B-Instruct 5.011 0.457 0.055
ADO-KT : Knowledge Transfer (from TSDA, Comp) with Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 12.97 0.0285 0.301
DeepSeek-R1-Distill-Qwen-32B 14.8 0.0339 0.280
GPT-4o 12.0 0.021 0.315
Llama3-70B-instruct 8.31 0.0134 0.365
Mistral-Small-24B-Instruct-2501 5.03 0.3402 0.152
Qwen2.5-32B-Instruct 23.38 0.083 0.185
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 8

Fig. 7: Assessing Sample Efﬁciency: The evolution of FOM with consumed data. Knowledge transfer with reﬁned KGs shows
increased sample efﬁciency.
utilized to aid the design of the hysteresis comparator. We
show the FOM evolution of the comparator in Figure 7.
D. DC-DC Converter
The DC-DC converter is a PWM-controlled circuit designed
to step down voltage, with 44 design parameters across 22
transistors. It contains substructure types such as the error am-
pliﬁer and comparator which are also found in prior circuits.
To enable knowledge transfer, we incorporate KGs from the
differential ampliﬁer, hysteresis comparator, and LDO. The
design targets include output power < 15 μW and regulated
voltage error < 0.02 V relative to a 0.6 V reference. As
shown in Table III, ADO-KT consistently outperforms ADO-
LLM, particularly when reﬁned KGs are used, highlighting
the beneﬁt of multi-source knowledge transfer.
E. Key Observations
1) Impact of Knowledge Transfer on Design Quality and
Sample Efﬁciency: Across all the circuits, incorporating KG-
based knowledge transfer signiﬁcantly improves design quality
and sample efﬁciency. As shown in Tables I, II, and III,
transferred knowledge enables higher FOMs and more designs
that meet speciﬁcations. Figure 7 further shows faster FOM
convergence, underscoring the value of knowledge reuse for
sample efﬁcient optimization.
2) Role of Knowledge Graph Quality: Optimization per-
formance is highly sensitive to KG quality, with reﬁned
KGs yielding further improvements over their unreﬁned coun-
terparts. These results afﬁrm that (i) the effectiveness of
knowledge transfer depends on the reliability of the injected
knowledge, and (ii) improving knowledge representation and
reﬁnement directly enhances transfer-driven analog design.
VI. CONCLUSION AND FUTURE WORK
Our work demonstrates the promise of structured knowledge
representations for enabling knowledge transfer in analog
circuit design. Moving forward, we anticipate that more ad-
vanced strategies for summarizing and distilling analog design
knowledge from LLMs is critical to improving transferability.
Furthermore, the development of specialized knowledge re-
ﬁnement mechanisms will be essential, not only to enhance
consistency, but also to yield deeper qualitative insights that
support interpretable and robust analog design optimization.
ACKNOWLEDGMENT
This material is based upon work supported by the Semi-
conductor Research Corporation Task No. 3160.055 through
UT Dallas’ Texas Analog Center of Excellence, and by the
National Science Foundation under Grants No. 1956313 and
No. 2334380.
REFERENCES
[1] R. Zhou, P. Poechmueller, and Y. Wang, “An analog circuit design
and optimization system with rule-guided genetic algorithm,” IEEE
Transactions on Computer-Aided Design of Integrated Circuits and
Systems, vol. 41, no. 12, pp. 5182–5192, 2022.
[2] R. Garnett, Bayesian Optimization. Cambridge University Press, 2023.
[3] A. Lberni, M. A. Marktani, A. Ahaitouf, and A. Ahaitouf,
“Analog circuit sizing based on evolutionary algo-
rithms and deep learning,” Expert Systems with Appli-
cations, vol. 237, p. 121480, 2024. [Online]. Available:
https://www.sciencedirect.com/science/article/pii/S0957417423019826
[4] W. Lyu, F. Yang, C. Yan, D. Zhou, and X. Zeng, “Batch Bayesian
optimization via multi-objective acquisition ensemble for automated
analog circuit design,” ser. Proceedings of Machine Learning Research,
J. Dy and A. Krause, Eds., vol. 80, 2018, pp. 3306–3314.
[5] K. Settaluri, A. Haj-Ali, Q. Huang, K. Hakhamaneshi, and
B. Nikolić, “Autockt: Deep reinforcement learning of analog circuit
designs,” 2020 Design, Automation & Test in Europe Conference
& Exhibition (DATE), pp. 490–495, 2020. [Online]. Available:
https://api.semanticscholar.org/CorpusID:210023627
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

## Page 9

TABLE III: DCDC Converter Optimization Results
Method Output Power ↓ (Spec: < 15μA) Reg. Voltage Error ↓ (Spec: < 0.02V) FOM ↑
BO 15.25 0.091 -0.073
ADO-LLM 15.21 0.041 -0.027
ADO-KT : Knowledge Transfer (from TSDA, Comp, LDO) without Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 14.3 0.029 -0.014
DeepSeek-R1-Distill-Qwen-32B 16.54 0.026 -0.022
GPT-4o 15.37 0.008 0.000
Llama3-70B-instruct 11.54 0.015 0.012
Mistral-Small-24B-Instruct-2501 11.61 0.030 -0.009
Qwen2.5-32B-Instruct 16.54 0.026 -0.022
ADO-KT : Knowledge Transfer (from TSDA, Comp, LDO) with Reﬁnement with different MKG
DeepSeek-Distill-Llama-70B 14.49 0.029 0.001
DeepSeek-R1-Distill-Qwen-32B 11.57 0.037 -0.007
GPT-4o 12.31 0.007 0.015
Llama3-70B-instruct 9.13 0.000 0.041
Mistral-Small-24B-Instruct-2501 12.01 0.013 0.012
Qwen2.5-32B-Instruct 16.54 0.026 -0.022
[6] H. Wang, K. Wang, J. Yang, L. Shen, N. Sun, H.-S. Lee, and S. Han,
“Gcn-rl circuit designer: Transferable transistor sizing with graph neural
networks and reinforcement learning,” 2020 57th ACM/IEEE Design
Automation Conference (DAC), pp. 1–6, 2020. [Online]. Available:
https://api.semanticscholar.org/CorpusID:218470023
[7] N. S. K. Somayaji, H. Hu, and P. Li, “Prioritized
reinforcement learning for analog circuit optimization with
design knowledge,” 2021 58th ACM/IEEE Design Automation
Conference (DAC), pp. 1231–1236, 2021. [Online]. Available:
https://api.semanticscholar.org/CorpusID:243915186
[8] K. S. Ns and P. Li, “Pareto optimization of analog circuits using
reinforcement learning,” ACM Transactions on Design Automation of
Electronic Systems, vol. 29, pp. 1 – 14, 2024. [Online]. Available:
https://api.semanticscholar.org/CorpusID:267000553
[9] Y. Yin, Y. Wang, B. Xu, and P. Li, “Ado-llm: Analog design bayesian
optimization with in-context learning of large language models,” 06
2024.
[10] C. Liu, Y. Liu, Y. Du, and L. Du, “Ladac: Large language
model-driven auto-designer for analog circuits.” [Online]. Available:
https://api.semanticscholar.org/CorpusID:268259740
[11] Y. Lai, S. Lee, G. Chen, S. Poddar, M. Hu, D. Z. Pan, and
P. Luo, “Analogcoder: Analog circuit design via training-free code
generation,” ArXiv, vol. abs/2405.14918, 2024. [Online]. Available:
https://api.semanticscholar.org/CorpusID:270045319
[12] D. V. Kochar, H. Wang, A. Chandrakasan, and X. Zhang, “Ledro:
Llm-enhanced design space reduction and optimization for analog
circuits,” ArXiv, vol. abs/2411.12930, 2024. [Online]. Available:
https://api.semanticscholar.org/CorpusID:274149830
[13] S. Pieper, C. W. Mehling, D. Hirsch, T. Lüke, and
S. Ihlenfeldt, “causalgraph: A python package for modeling,
persisting and visualizing causal graphs embedded in knowledge
graphs,” ArXiv, vol. abs/2301.08490, 2023. [Online]. Available:
https://api.semanticscholar.org/CorpusID:256080522
[14] I. RobertL.Logan, N. F. Liu, M. E. Peters, M. Gardner, and S. Singh,
“Barack’s wife hillary: Using knowledge graphs for fact-aware
language modeling,” ArXiv, vol. abs/1906.07241, 2019. [Online].
Available: https://api.semanticscholar.org/CorpusID:189998980
[15] N. Karalis, G. M. Mandilaras, and M. Koubarakis, “Extending
the yago2 knowledge graph with precise geospatial knowledge,” in
International Workshop on the Semantic Web, 2019. [Online]. Available:
https://api.semanticscholar.org/CorpusID:204754876
[16] S. Ji, S. Pan, E. Cambria, P. Marttinen, and P. S. Yu,
“A survey on knowledge graphs: Representation, acquisition,
and applications,” IEEE Transactions on Neural Networks and
Learning Systems, vol. 33, pp. 494–514, 2020. [Online]. Available:
https://api.semanticscholar.org/CorpusID:211010433
[17] A. Hogan, E. Blomqvist, M. Cochez, C. d’Amato, G. de Melo,
C. Gutierrez, J. E. L. Gayo, S. Kirrane, S. Neumaier, A. Polleres,
R. Navigli, A.-C. N. Ngomo, S. M. Rashid, A. Rula, L. Schmelzeisen,
J. Sequeda, S. Staab, and A. Zimmermann, “Knowledge graphs,” ACM
Computing Surveys (CSUR), vol. 54, pp. 1 – 37, 2020. [Online].
Available: https://api.semanticscholar.org/CorpusID:212414739
[18] T. B. Brown, B. Mann, N. Ryder, M. Subbiah, J. Kaplan, P. Dhariwal,
A. Neelakantan, P. Shyam, G. Sastry, A. Askell et al., “Language models
are few-shot learners,” Advances in Neural Information Processing
Systems, vol. 33, pp. 1877–1901, 2020.
[19] D. Guo and et al., “Deepseek-r1: Incentivizing reasoning capability
in llms via reinforcement learning,” arXiv preprint arXiv:2501.12948,
2025. [Online]. Available: https://arxiv.org/abs/2501.12948
[20] OpenAI et al., “Gpt-4o system card,” https://arxiv.org/abs/2410.21276,
2024, accessed: 2025-04-21.
[21] M. AI et al., “Meta llama 3 70b instruct model card,”
https://huggingface.co/meta-llama/Meta-Llama-3-70B-Instruct, 2024,
accessed: 2025-04-21.
[22] ——, “Mistral-small-24b-instruct-2501 model card,”
https://huggingface.co/mistralai/Mistral-Small-24B-Instruct-2501,
2025, accessed: 2025-04-21.
[23] Q. Team et al., “Qwen2.5-32b-instruct model card,”
https://huggingface.co/Qwen/Qwen2.5-32B-Instruct, 2024, accessed:
2025-04-21.
Authorized licensed use limited to: University of Florida. Downloaded on September 28,2026 at 03:52:41 UTC from IEEE Xplore. Restrictions apply.

