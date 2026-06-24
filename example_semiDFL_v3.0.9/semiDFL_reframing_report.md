# SemiDFL 目标论文增量贡献重包装报告

目标 PDF：[semiDFL.pdf](E:/diagram制图skill/example_semiDFL_v3.0.9/semiDFL.pdf)  
分析时间：2026-05-21  
工作流：delta-reframer-fl / target-paper reframing  
公开报告路径：[semiDFL_reframing_report.md](E:/diagram制图skill/example_semiDFL_v3.0.9/semiDFL_reframing_report.md)

## 0. 语言、证据与可读性契约

本报告用中文做诊断、排序、风险和修改建议；只在 manuscript / rebuttal 可直接使用的片段中使用英文。目标论文证据全部标记为 `new-paper-derived`。匿名 casebook 只作为 `base-corpus-derived analogy`，用于提示包装方式，不作为 SemiDFL 这篇论文的事实证据。

未提供真实审稿意见或作者回复，因此第 4 节所有 reviewer 攻击均标记为 `simulated-review`。PDF 文本抽取总体可读；公式符号和个别希腊字母有编码噪声，但摘要、方法、表格、图注、实验设置和 appendix 证据足以支撑本文分析。

可读性规则：每个抽象包装点先讲白话含义，再给技术论点、例子或类比、PDF 证据锚点、不可越界的边界。

## 1. 深度证据底座

这篇论文的核心场景不是普通 semi-supervised FL，也不是普通 DFL，而是一个三重资源约束场景：没有中心服务器、客户端标签极少且来源不同、数据分布高度 non-IID。普通 SSL 依赖较稳定的本地伪标签；普通 CFL/FL-SSL 往往依赖中心协调或服务器侧有标签信息；普通 DFL 又多假设每个客户端有足够标签。SemiDFL 的价值在于：在这些假设同时失效时，用邻域信息和生成式共识构造可用的伪监督、数据混合空间和模型聚合信号。

| 证据项 | PDF 锚点 | 对包装的意义 | 边界 |
|---|---|---|---|
| 目标 setting 是 DFL + SSL + diverse data sources | 摘要与 Introduction，p.1；L/U/M 三类客户端定义，p.2 | 明确不是把 SSL 套到 CFL，而是在无中心的 DFL 中处理 labeled / unlabeled / mixed clients | 不要写成解决所有 federated SSL |
| 旧假设 1：每个客户端本地模型能独立产生可靠伪标签 | Methodology / NPL，p.3；Table 2，p.7 | NPL 用邻域分类器和邻域自适应阈值修复 non-IID 下伪标签噪声 | Table 2 是经验支持，不是伪标签正确性的理论证明 |
| 旧假设 2：本地 labeled + pseudo-labeled 数据足以做 MixUp | Consensus MixUp，p.4；Table 3，p.7 | 共识 diffusion 生成类似分布的数据，作为 data-space bridge | diffusion 带来计算成本，不应声称免费或轻量 |
| 旧假设 3：可以用共享验证集评估客户端模型 | Adaptive Aggregation，p.5；Table 4，p.7 | 用生成数据构造资源兼容的聚合权重评估代理 | 这是 surrogate evaluation，不等同真实共享 test set |
| 端到端效果 | Table 1，p.6；Fig.2/3，p.6；appendix topology Table 1，p.12 | 支持在 label scarcity、non-IID 和 topology variation 下的整体收益 | DFL-UB 是全标签上界，不是同资源 baseline |
| 复现实验信息 | Algorithm 1，p.5；supplement Experimental Setup，p.10；Reproducibility Checklist，p.12 | 支持方法流程和主要超参可复核 | 如果代码链接未实际可用，仍有复现风险 |

### Claim-Support Matrix

| 当前或潜在 claim | 支撑证据 | 风险 | 修改动作 |
|---|---|---|---|
| SemiDFL 是第一个面向 diverse data sources 的 semi-supervised DFL paradigm | 摘要、Introduction、贡献 1 | “first” 容易被 closest related work 攻击 | 加上 precise scope: decentralized, no central server, L/U/M clients, non-IID |
| Consensus model and data spaces 是解决机制 | Overview，p.3；Fig.1；Algorithm 1 | 如果只写成两个 buzzwords，会显得空泛 | 改成 pressure -> module -> evidence 的闭环 |
| NPL 改善 noisy pseudo-labels | NPL 方法，p.3；Table 2，p.7 | ablation 证明性能提升，不直接证明伪标签质量提升 | 增加“supports”措辞，避免 “proves” |
| C-MixUp 缓解本地数据稀缺和 non-IID | Consensus MixUp，p.4；Table 3，p.7 | diffusion 是否只是强 generator 替换 GAN | 把贡献写为 decentralized consensus data-space construction，而不是 diffusion 新颖性 |
| AdaGen 能避免共享 test dataset | Adaptive Aggregation，p.5；Table 4，p.7 | generated validation set 可能有 circular evaluation 质疑 | 明确它是 resource-compatible surrogate，Table 4 只说明 comparable / useful |
| 方法鲁棒于 topology | appendix topology experiment，p.11-12 | 只覆盖三种 synthetic topology | 写 evaluated topologies，不写 arbitrary network |

## 2. 强包装诊断

### 2.1 弱表层 delta

弱表层写法是：“我们把 pseudo-labeling、MixUp、diffusion generation 和 adaptive aggregation 组合到 DFL 里，得到第一个 semi-supervised DFL 方法。”这个写法容易触发四类审稿攻击：第一，primitive 都已知，像 A+B/C；第二，diffusion + MixUp 看起来是工程增强；第三，adaptive aggregation 用生成数据评估模型会被问是否可靠；第四，实验如果只按 dataset 排列，会被看作性能堆表，而不是机制证据。

### 2.2 更强实际 delta

更强的贡献不是“用了 diffusion”或“用了邻居伪标签”，而是：在无中心、无共享原始数据、无共享验证集、标签来源异质的 DFL-SSL 场景中，构造一个可闭环运行的共识监督系统。NPL 解决伪标签信号不稳定；C-MixUp 用共识 diffusion 补齐本地数据空间缺口；AdaGen 用生成数据作为模型聚合的评估代理。这三个模块回答的是同一个问题：当中心协调和可靠标签都不可用时，DFL 如何获得可交换、可聚合、可评估的训练信号。

推荐核心句：

```text
SemiDFL is best framed as a constraint-driven consensus-supervision framework for decentralized semi-supervised learning: it replaces unavailable centralized coordination, shared labeled data, and shared validation signals with neighborhood pseudo-labeling, consensus generative data-space construction, and generated-data-based adaptive aggregation.
```

### 2.3 为什么当前写法会招攻击

| 攻击入口 | 触发原因 | 更强包装方式 |
|---|---|---|
| Novelty / incrementality | “first SSL DFL”后面紧接模块列表 | 先定义旧假设失效，再说每个模块修复哪个缺失信号 |
| A+B/C combination | pseudo-labeling、MixUp、diffusion、adaptive aggregation 都有前作 | 承认 primitive 已知，强调 constraint-driven coupling |
| Mechanism evidence | ablation 分散在 Table 2-4，没组织成 failure-mode table | 把 ablation 改写成三类失败模式排除 |
| Baseline fairness | CFL/SSL baseline 被 adapt 到 DFL，但资源契约不够醒目 | 区分 same-contract、diagnostic 和 upper-bound baseline |
| Cost / scalability | diffusion 训练和生成样本有额外成本 | 主动写 DPM-Solver、1000 samples / 10 rounds、100 validation samples，并承认成本边界 |

## 2.5 潜在贡献挖掘

| Candidate contribution | Evidence label | 白话解释 | Evidence anchor | Safe use | Unsafe overclaim |
|---|---|---|---|---|---|
| Semi-supervised DFL 的问题契约 | `paper-explicit` | 论文真正打开的是“无中心 + 少标签 + non-IID + 三类客户端”的组合场景 | 摘要、Introduction、L/U/M objective，p.1-2 | Abstract / Introduction / Contribution 1 | “solves federated semi-supervised learning” |
| 邻域伪标注是 non-IID 下的监督去偏机制 | `latent-but-supported` | 不是简单拿邻居投票，而是用邻居模型补本地模型偏置，再用邻域 qualified count 调阈值 | NPL，p.3；Table 2，p.7 | Method overview / ablation discussion | “guarantees correct pseudo-labels” |
| 共识 diffusion 是 data-space bridge | `latent-but-supported` | 客户端不能共享数据，但可通过 consensus-updated generator 生成相似分布样本，让 MixUp 不局限于本地偏斜数据 | C-MixUp，p.4；Table 3，p.7 | Figure 1 caption / method / contribution | “learns the global data distribution exactly” |
| 生成数据作为聚合评估代理 | `latent-but-supported` | 没有共享验证集时，生成数据给 adaptive aggregation 一个资源兼容的评价信号 | Adaptive Aggregation，p.5；Table 4，p.7 | Method / rebuttal / limitation | “equivalent to evaluation on real test data” |
| 实验体系可重排为 reviewer concern map | `story-level reframing` | Table 1 回答性能，Fig.2/3 回答 non-IID 和 label scarcity，Table 2-4 回答模块必要性，appendix 回答 topology | p.6-7, p.10-12 | Experiment section roadmap | “comprehensive real-world validation” |
| 去中心化自监督/弱监督共识的更大方向 | `future-boundary hook` | 这篇论文暗示“没有中心监督也能形成训练信号”，但当前只有图像分类 benchmark | Discussion / conclusion | Motivation / future work | 把它当作已完成的通用理论贡献 |

## 2.6 顶会故事重构

### 2.6.1 Problem Equation

白话版：以前的 SSL-FL 默认系统里有某种“共同参照物”：中心服务器、共享监督信息、或至少较可靠的本地标签。SemiDFL 的问题是这些参照物同时缺失，客户端还因为 non-IID 看见不同世界。论文要回答的是：没有中心和共享验证信号时，客户端如何仍然形成可信伪标签、可混合数据空间和有效聚合权重。

```text
Paper-ready problem equation:
Central coordination is unavailable + client supervision is sparse and heterogeneous + local pseudo-labels are biased under non-IID data + no shared validation set exists for adaptive aggregation -> decentralized SSL needs a resource-compatible consensus mechanism for supervision, data-space augmentation, and model weighting.
```

证据：摘要和 Introduction 的 key question，p.1；L/U/M objective，p.2；NPL / C-MixUp / AdaGen，p.3-5。边界：不能扩展为任意 decentralized learning 或任意真实部署。

### 2.6.2 Contribution Ladder

| Level | 白话含义 | Paper-ready claim | Evidence anchor | Risk if overclaimed |
|---|---|---|---|---|
| Component | 三个模块分别修一个缺口 | NPL, C-MixUp, and AdaGen address pseudo-label noise, local data insufficiency, and aggregation evaluation | p.3-5 | 像模块拼装 |
| Coupling | 模块必须连起来才形成闭环 | The method couples neighborhood supervision, consensus generative data, and generated-data-based weighting into a decentralized SSL loop | Fig.1, Algorithm 1 | 如果不解释接口，会被说 A+B/C |
| Problem regime | 贡献来自受约束 setting | SemiDFL targets DFL clients with labeled, unlabeled, and mixed sources under non-IID distributions | p.1-2 | “first” claim 太宽 |
| Evidence system | 结果覆盖主要 reviewer doubts | Main results, label-ratio/non-IID curves, ablations, topology appendix jointly support the story | p.6-7, p.10-12 | 不覆盖真实大规模部署 |
| Future boundary | 更大方向是无中心弱监督共识 | The work suggests a path toward decentralized consensus supervision without raw-data sharing | Discussion only | 不能当主贡献 |

### 2.6.3 可直接改写的输出

| Output | 白话目的 | What to write | Evidence anchor | Boundary |
|---|---|---|---|---|
| Abstract rewrite | 先立资源约束，再讲闭环 | Emphasize unavailable central coordinator, scarce heterogeneous labels, and missing shared validation signal before modules | Abstract, p.1 | 不要只说 “first” |
| Introduction framing | 把 challenge 写成三缺失 | Missing reliable pseudo-labels, missing data-space coverage, missing validation signal for aggregation | p.1-5 | 不要夸成理论问题全解 |
| Contribution bullets | 从“模块清单”改成“缺失信号修复” | Contribution 1 setting, Contribution 2 consensus supervision/data space, Contribution 3 resource-compatible aggregation/evidence | p.1, p.3-7 | 保留 primitive 已知边界 |
| Related-work boundary | 明确哪些工作有中心/服务器/共享资源 | Separate CFL-SSL, supervised DFL, and DFL-compatible SSL | Related Works, p.2-3 | 不要贬低 baseline |
| Method overview | 用 pressure -> mechanism -> evidence | Each subsection begins with the failure it repairs | p.3-5 | 不要让 diffusion 看成孤立增强 |
| Figure/caption direction | Figure 1 应显示闭环 | Label steps 1/3/6 as “supervision signal”, “data-space bridge”, “aggregation signal” | Fig.1, p.4 | 不要在图里加入未实现组件 |

English abstract-style snippet:

```text
We study semi-supervised decentralized federated learning under a stricter resource contract: clients have heterogeneous labeled, unlabeled, or mixed data sources, no central coordinator is available, and no shared validation set can be assumed for model weighting. SemiDFL addresses this setting by building consensus in both supervision and data/model spaces. It uses neighborhood pseudo-labeling to reduce local pseudo-label bias, consensus-updated diffusion models to construct a shared data-space surrogate for MixUp without raw-data sharing, and generated-data-based adaptive aggregation to weight neighboring models without an external validation set.
```

## 3. Story Route Candidate Board

### 3.1 六条路线排序

| Rank | Story route | Novelty defense | Evidence fit | A+B/C resistance | Baseline control | Mechanism control | Cost/reproducibility control | Rewrite cost | New-experiment pressure | Best use |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Constraint-driven consensus supervision | 强 | 强 | 强 | 中 | 强 | 中 | 中 | 低 | 默认主线 |
| 2 | Broken-assumption / no-common-reference route | 强 | 强 | 中 | 强 | 中 | 中 | 中 | 低 | Intro + related work |
| 3 | Missing-signal-to-module route | 中 | 强 | 强 | 中 | 强 | 中 | 高 | 低 | Method + Figure 1 |
| 4 | Evidence-system route | 中 | 强 | 中 | 强 | 强 | 中 | 低 | 低 | Experiment rewrite |
| 5 | Resource-compatible aggregation route | 中 | 中 | 中 | 中 | 中 | 强 | 中 | 中 | AdaGen defense |
| 6 | Future decentralized weak-supervision route | 弱到中 | 弱 | 中 | 弱 | 弱 | 弱 | 低 | 高 | Discussion only |

### 3.1b 白话路线说明

| Route | Plain-language meaning | Concrete example or analogy | Why this helps reviewers | Where to use it | When unsafe |
|---|---|---|---|---|---|
| 1 | 论文不是堆模块，而是在无中心弱监督条件下造共识训练信号 | 像没有裁判和统一试卷时，邻居互校、共同生成练习题、再用练习题评估谁更可靠 | 直接回答 novelty 和 A+B/C | Abstract, intro, contribution | 如果没有解释三模块接口 |
| 2 | 旧方法默认有共同参照物，这里共同参照物缺失 | 中心服务器、共享 labeled server、共享 validation set 都不可用 | 给 related work 清晰边界 | Intro, related work | 如果把所有前作都说成不适用 |
| 3 | 每个模块对应一个缺失信号 | NPL 修伪标签，C-MixUp 修数据覆盖，AdaGen 修聚合权重 | 让模块必要性更强 | Method, Figure 1 | 如果 ablation 没连到 failure |
| 4 | 实验不是堆表，而是在回答 reviewer doubts | Table 1 性能，Fig.2 non-IID，Fig.3 label scarcity，Table 2-4 机制 | 减少“实验不系统”的攻击 | Experiments | 如果声称覆盖真实部署 |
| 5 | AdaGen 的价值是资源兼容，不是更准的 test set | 没有共享 test set，只能用生成数据近似评估邻居模型 | 防守 adaptive aggregation | Method, rebuttal | 如果说等价真实验证集 |
| 6 | 更大的方向是无中心弱监督共识 | 当前只是图像分类 benchmark 的第一步 | 给 discussion 增加 vision | Limitation/future work | 如果作为主贡献 |

### 3.2 Option 1: Constraint-Driven Consensus Supervision

白话含义：把主线从“我们提出三个模块”改成“在三个关键参照物缺失时，我们构造一个能闭环工作的监督共识系统”。技术论点：SemiDFL replaces centralized coordination and shared supervision signals with neighborhood-supervision, consensus data-space construction, and generated-data-based model weighting.

证据：Introduction 的 key question，p.1；NPL/C-MixUp/AdaGen，p.3-5；Table 1-4，p.6-7。边界：不要说每个 primitive 都新，防守的是 constraint-driven coupling。

Ready-to-paste English:

```text
The key novelty of SemiDFL is not any single primitive in isolation, but the resource-compatible coupling of three consensus signals: neighborhood pseudo-labels for supervision, consensus-generated samples for data-space alignment, and generated-data-based model evaluation for adaptive aggregation.
```

### 3.3 Option 2: Broken-Assumption / No-Common-Reference Route

白话含义：强调旧 SSL-FL/DFL 方法依赖某种共同参照物，而目标 setting 中共同参照物不存在。技术论点：SemiDFL addresses the absence of central coordination, reliable local labels, and shared validation resources in one DFL-SSL formulation.

证据：CFL 依赖中心服务器、DFL 移除中心、SSL 需要伪标签，p.1-2；AdaGen 说明共享 test dataset 不现实，p.5。边界：不要说所有 CFL-SSL baseline 不可比较，而是区分 same-resource baseline 和 diagnostic baseline。

Ready-to-paste English:

```text
Unlike centralized semi-supervised FL methods that can rely on server-side coordination or shared supervision, SemiDFL targets a no-common-reference regime where clients must infer supervision, data-space coverage, and aggregation weights through decentralized neighborhood interactions.
```

### 3.4 Option 3: Missing-Signal / Pressure-To-Mechanism Route

白话含义：每个模块都要回答“没有它时系统缺什么信号”。这能把 A+B/C 攻击转成“场景压力驱动的接口设计”。

| Missing signal or pressure | Failure in target regime | Target-paper replacement | Evidence |
|---|---|---|---|
| Reliable pseudo-label supervision | local model is biased under non-IID | neighborhood classifiers + neighborhood adaptive threshold | NPL, Table 2 |
| Broader data-space coverage for MixUp | local labeled/pseudo-labeled pool is sparse and skewed | consensus-updated diffusion generated samples | C-MixUp, Table 3 |
| Model weighting without shared validation set | constant weights ignore model quality; shared test set is impractical | generated-data-based adaptive aggregation | AdaGen, Table 4 |

Ready-to-paste English:

```text
Each component is introduced as a replacement for an unavailable signal in decentralized SSL: NPL replaces unreliable local pseudo-labeling, C-MixUp replaces missing cross-client data coverage, and AdaGen replaces the unavailable shared validation signal for adaptive model aggregation.
```

### 3.5 Option 4: Evidence-System Route

白话含义：现在的实验已经有很多防守材料，但需要重排成“审稿人疑问地图”。技术论点：The evaluation should be organized around failure modes rather than datasets alone.

证据：Table 1 主性能；Fig.2 non-IID；Fig.3 label ratio；Table 2 NPL；Table 3 C-MixUp；Table 4 AdaGen；appendix topology Table 1。边界：这些是 benchmark 和 synthetic topology 证据，不是工业部署。

Ready-to-paste English:

```text
We organize the evaluation around the main failure modes of decentralized SSL: end-to-end performance under scarce labels, robustness to stronger non-IID splits, sensitivity to labeled-data ratio, component-level necessity, and robustness across representative decentralized topologies.
```

### 3.6 Option 5: Resource-Compatible Aggregation Route

白话含义：AdaGen 不要被包装成“比真实 test set 更好”，而是“在没有共享 test set 时仍可工作的聚合代理”。这条路线用于防守成本、隐私和评估边界。

证据：Adaptive Aggregation 方法明确共享 test dataset 不现实，p.5；Table 4 显示 AdaGen 优于 constant，并与 AdaTest 接近但不总是更高，p.7。边界：不能说生成验证集等价真实验证集；要承认 surrogate nature。

Ready-to-paste English:

```text
AdaGen should be interpreted as a resource-compatible surrogate for model weighting, not as a replacement for external test evaluation. It enables adaptive aggregation when a shared validation set is unavailable, while the final task performance is still assessed on held-out benchmarks.
```

### 3.7 Option 6: Future Decentralized Weak-Supervision Boundary

白话含义：论文可以在 discussion 中提出更大方向：无中心系统如何形成弱监督共识。但当前证据只覆盖图像分类、10 客户端左右的 synthetic topology、三个 benchmark，因此它只能是 future-boundary hook。

Ready-to-paste English:

```text
These results point to a broader direction: decentralized systems may construct weak supervision through consensus signals rather than relying on centralized labeled resources. We leave broader domains, larger-scale deployments, and theoretical guarantees to future work.
```

### 3.8 推荐组合

默认主线：Option 1。  
Method / Figure 路线：Option 3。  
Related-work / baseline 防守路线：Option 2。  
Experiment 组织路线：Option 4。  
Limitation / discussion 路线：Option 5 + Option 6。

最小 Tier 0 组合：只改 abstract、Introduction 最后一段、contribution bullets、Figure 1 caption、实验开头 roadmap、limitation paragraph。这样不需要新实验，也能把 paper 从“模块组合”拉到“无中心弱监督共识系统”。

## 4. Reviewer Attack Preplay

| Reviewer attack | Why reviewer will ask | Manuscript trigger | Strong defense posture | No-new-experiment repair |
|---|---|---|---|---|
| `simulated-review` novelty / incrementality | pseudo-labeling、MixUp、diffusion、adaptive aggregation 都是已知 primitive | contribution bullets 以模块为单位 | 承认 primitive 已知，强调 no-common-reference DFL-SSL 下的 coupling | 改 abstract/contribution：先讲 unavailable signals，再讲模块 |
| `simulated-review` A+B/C combination | 三个模块看起来可拆开 | Fig.1 和 Method 没把接口压力写够 | 每个模块对应一个缺失信号，ablation 支持必要性 | 加 pressure -> mechanism -> evidence table |
| `simulated-review` baseline fairness | CFL-SSL baseline 被 adapt 到 DFL，资源是否一致不清 | baseline section 只列方法 | 区分 same-contract、upper-bound、diagnostic baseline | 加 comparison contract paragraph |
| `simulated-review` mechanism evidence | Table 2-4 是性能 ablation，不等于机制证明 | “verifies superiority” 等措辞偏强 | 改成 supports / rules out simpler alternatives | 在 ablation 前说明每张表回答哪个 failure |
| `simulated-review` cost / scalability | diffusion 训练和生成开销明显 | 主文只说性能，不强调成本安排 | 主动承认 compute tradeoff，说明 DPM-Solver 和 sample schedule | 在 limitation 加成本边界；appendix 指向超参 |
| `simulated-review` privacy/safety boundary | 文中有 no raw data sharing / privacy constraints，但没有 formal privacy | 容易被读成 privacy guarantee | 只主张不共享原始数据和不需共享 test set | 避免 differential privacy / formal privacy wording |
| `simulated-review` scope / generalization | 数据集为 MNIST/Fashion-MNIST/CIFAR-10，topology 为 synthetic | “paradigm” 和 “real-world” 可能太宽 | 强调 evaluated regime 和 representative topologies | Limitation 加 broader domains / larger deployments |
| `simulated-review` reproducibility | supplement 说 code will be available，abstract 有 GitHub link；需一致 | code availability 表述不一致 | 明确代码、超参、硬件、训练轮数、生成频率 | 统一 Code statement，确保 repo 可访问 |

## 5. Manuscript Trigger Localization

| Manuscript area | Current strength | Weak trigger | Risk created | Concrete repair |
|---|---|---|---|---|
| Title / abstract | setting 明确，模块齐全 | “first semi-supervised DFL method”后接模块列表 | novelty 被读成 first + combination | 加 resource contract 和 unavailable-signal framing |
| Introduction | key question 好 | challenge 只列两个问题：伪标签和数据生成，聚合评估缺失出现较晚 | AdaGen 像额外优化 | 把三缺失提前：pseudo-label, data-space, aggregation evaluation |
| Contributions | 有 consensus data/model spaces | 每条仍像组件描述 | A+B/C 攻击 | 改为 problem-regime / mechanism-coupling / evidence-system |
| Related work | 覆盖 SSL、FL-SSL、DFL | closest prior 的资源边界未表格化 | baseline fairness 和 novelty 边界不清 | 加一张 related-work resource-contract table |
| Method overview / Fig.1 | 流程完整 | 图中步骤 2/4/5 是 general ideas，1/3/6 是 main contributions，但没有写成信号闭环 | 审稿人看不出为何三者必须连 | 在 caption 加 “supervision signal / data-space bridge / aggregation signal” |
| Experiments | 数据集、non-IID、label ratio、ablation、topology 都有 | 按 dataset / table 排列，防守目标不突出 | 被要求更多实验 | 开头加 reviewer-question roadmap |
| Limitation / discussion | 结论简洁 | 成本、privacy boundary、scope boundary 不足 | cost/privacy/generalization 攻击 | 加一段 bounded limitation，不自毁贡献 |

## 6. 分层修改计划

### Tier 0: 不做新实验

| Action | Manuscript location | Evidence reused or needed | Reviewer-defense purpose | Ready-to-paste English if applicable |
|---|---|---|---|---|
| 重写 abstract 前两句为 resource-contract framing | Abstract | p.1 setting | novelty / A+B/C | `We study decentralized semi-supervised learning under a stricter resource contract where central coordination, abundant labels, and shared validation signals are unavailable.` |
| 重写 contribution bullets | Contributions | p.1-5 | 模块组合防守 | `Our contribution is a consensus-supervision loop rather than an isolated component replacement.` |
| 加 pressure -> mechanism -> evidence table | Method overview | NPL/C-MixUp/AdaGen + Tables 2-4 | mechanism evidence | `Each module is tied to a specific missing signal in decentralized SSL.` |
| 重排实验开头 | Experiments | Table 1, Fig.2/3, Table 2-4, appendix topology | experiment scope | `The evaluation is organized around four concerns: end-to-end performance, robustness, component necessity, and topology sensitivity.` |
| 加 limitation paragraph | Discussion/Conclusion | existing setup and appendix | cost/scope/privacy boundary | `We do not claim formal privacy guarantees or universal topology robustness; our claim is bounded to no raw-data sharing and the evaluated decentralized settings.` |

### Tier 1: 复用已有材料

| Action | Manuscript location | Evidence reused or needed | Reviewer-defense purpose | Ready-to-paste English if applicable |
|---|---|---|---|---|
| 从训练日志或已有代码补充 runtime / generation overhead | Appendix / limitation | existing logs if available | cost defense | `We report the additional generation schedule and runtime overhead to make the quality-cost tradeoff explicit.` |
| 加 generated samples 的质量或类别分布可视化 | Appendix | existing generated images/statistics | C-MixUp mechanism | `The generated samples are used as a data-space surrogate for mixing, not as task-test evidence.` |
| 加 baseline resource-contract table | Related work / experiments | current baseline descriptions | baseline fairness | `We distinguish same-resource baselines from diagnostic baselines requiring additional coordination or supervision.` |
| 统一 code availability | Abstract footnote / checklist | GitHub repo, configs | reproducibility | `Code and configuration files are provided to support reproduction of the reported protocol.` |

### Tier 2: 需要新证据

| Action | Manuscript location | Evidence reused or needed | Reviewer-defense purpose | Ready-to-paste English if applicable |
|---|---|---|---|---|
| 更大客户端数或更复杂 topology | Appendix / new experiment | new runs | scalability/topology | 不建议主修前强推，除非目标会议非常看重系统规模 |
| 真实医疗/遥感/移动端等应用域 | New experiment | new dataset | scope/generalization | 当前证据不足以宣称真实部署泛化 |
| 伪标签准确率诊断 | Appendix | need labels for diagnostic subset | mechanism | 可加强 NPL，但不是 Tier 0 必需 |
| 通信/计算开销表 | Appendix | runtime/communication logs | cost | 若已有日志则 Tier 1，否则 Tier 2 |

## 7. Rebuttal Pattern Library

### Novelty / Incrementality

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 承认 primitive，强调约束驱动耦合 | `We agree that pseudo-labeling, MixUp, diffusion models, and adaptive aggregation are established primitives. The contribution of SemiDFL is the way these primitives are coupled to replace unavailable centralized signals in semi-supervised DFL: local pseudo-label reliability, cross-client data-space coverage, and validation-based model weighting.` | `Each component is novel.` | 会被逐项举前作反驳 |

### A+B/C Combination

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 把组合解释成接口约束 | `The components are not added independently. NPL produces higher-quality pseudo-labeled samples, C-MixUp uses them together with consensus-generated data to form a shared data-space surrogate, and AdaGen uses generated samples to make adaptive aggregation possible without a shared validation set.` | `We combine several effective techniques.` | 正中 A+B/C 攻击 |

### Baseline Fairness

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 定义 comparison contract | `We clarify the comparison contract by separating the fully labeled DFL upper bound from same-resource SSL baselines adapted to the decentralized setting. AdaTest is used only as a diagnostic ablation because it assumes an additional shared test set that SemiDFL does not use.` | `The baselines are unfair because they need extra resources.` | 听起来像回避强 baseline |

### Mechanism Evidence

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 用现有 ablation 支撑而非证明 | `The ablations support the proposed mechanism by isolating the three failure points: Table 2 evaluates neighborhood pseudo-labeling, Table 3 evaluates consensus data-space construction, and Table 4 evaluates generated-data-based adaptive aggregation.` | `The ablations prove the mechanism.` | 性能 ablation 不是理论证明 |

### Cost / Scalability

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 主动承认 quality-cost tradeoff | `SemiDFL introduces additional generation cost. We mitigate this by using accelerated diffusion sampling and generating samples periodically, and we bound the claim to improved performance under the reported resource schedule rather than claiming negligible overhead.` | `The overhead is negligible.` | PDF 目前没有完整 overhead 表 |

### Scope / Generalization

| Defense posture | Submit-ready English | Risky wording to avoid | Why risky |
|---|---|---|---|
| 保持 evaluated-regime claim | `We keep the claim focused on the evaluated semi-supervised DFL regimes: scarce labels, non-IID splits, and representative decentralized topologies on the reported benchmarks. Broader domains and larger deployments are important future validation targets.` | `SemiDFL works for real-world DFL broadly.` | 证据只覆盖 benchmark 和 synthetic topology |

## 8. 匿名案例类比附录

这些案例来自 packaged anonymous base knowledge，只用于提示包装动作，不作为 SemiDFL 的事实证据。

| Case or lens | Original weak pattern | Effective packaging move | How to borrow for SemiDFL | Boundary |
|---|---|---|---|---|
| `combination-method-needs-mechanism-claim` | 方法像 known components assembly | 把组合写成 constraint-driven interface | 用 pressure -> module -> evidence 表防守 NPL/C-MixUp/AdaGen | 不说 primitive 全新 |
| `incremental-gain-with-ablation-pressure` | 只展示性能提升 | 把 ablation 组织成 failure-mode 排除 | Table 2-4 分别回答伪标签、数据空间、聚合评估 | 不说 ablation 证明机制 |
| `baseline-contract-fairness-defense` | baseline 资源边界不清 | 先定义 same-contract vs diagnostic | DFL-UB、AdaTest、CFL/SSL baselines 分层 | 不要说 baseline 不可比 |
| `evidence-system-coverage-defense` | 实验有但叙事散 | 把结果映射到 reviewer concern | Table 1/Fig.2/Fig.3/Table2-4/appendix topology 重排 | 不扩大到真实部署 |
| `efficiency-system-claim-under-cost-scrutiny` | 系统方法易被问成本 | 主动定义 quality-cost tradeoff | 写 diffusion sampling schedule 和成本边界 | 没 overhead 表别说 negligible |
| `scope-generalization-boundary-defense` | claims 写太满 | 把 evaluated regime 前置 | 限定到 reported datasets/topologies/label ratios | broader deployment 放 future work |

## 9. 残余风险与安全边界

| Claim boundary | Safe wording | Unsafe wording | Evidence needed to strengthen |
|---|---|---|---|
| Mechanism | `supports the effectiveness of NPL/C-MixUp/AdaGen` | `proves the mechanism` | 伪标签准确率、生成样本分布、权重稳定性诊断 |
| Privacy / safety | `without raw-data sharing and without assuming a shared validation set` | `privacy-preserving` or `formally private` | DP proof, leakage analysis, secure aggregation protocol |
| Cost | `under the reported generation schedule` | `negligible overhead` | runtime, GPU hours, communication bytes, memory table |
| Generalization | `on MNIST, Fashion-MNIST, CIFAR-10 and evaluated topologies` | `general real-world DFL` | real-domain datasets, larger client counts, heterogeneous devices |
| Reproducibility | `algorithm, hyperparameters, hardware, and code link are reported in the PDF` | `fully reproducible` if repo/configs are absent | verified public code, seeds, scripts, exact topology configs |
| Baselines | `same-resource baselines plus upper-bound/diagnostic references` | `all baselines are directly comparable` | resource-contract table and adaptation details |

## 最推荐的改稿方向

把论文主线改成“无中心弱监督共识系统”，不是“semi-supervised DFL 的第一个模块组合”。一句话版本：

```text
SemiDFL studies a no-common-reference regime for decentralized semi-supervised learning and constructs a consensus-supervision loop that replaces unavailable centralized coordination, shared data-space coverage, and shared validation signals with neighborhood pseudo-labeling, consensus generative MixUp, and generated-data-based adaptive aggregation.
```

这条主线最能防守 novelty、A+B/C、baseline fairness 和 mechanism evidence，同时不需要新实验。最关键的 Tier 0 动作是重写 abstract/contributions、给 Method 加 pressure table、给 Experiment 加 reviewer-question roadmap、给 Discussion 加成本和范围边界。
