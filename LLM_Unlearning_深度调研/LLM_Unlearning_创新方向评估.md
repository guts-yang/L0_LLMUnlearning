# LLM Unlearning 创新方向评估（idea-evaluator）

**日期**：2026-09-08
**评估对象**：4 个候选方向（含您已有的方向 A / 方向 G）
**评估框架**：Higher / Faster / Stronger / Cheaper / Broader 五维 + 致命缺陷审计 + 范式转移探针 + 可行性
**能力假设**：研究生，单机算力（可跑 7B LoRA / 全参微调），熟悉进化算法与多目标优化，已在复现 SimNPO；**每周有效工时按 20–25h 估（此为假设，若实际不同请告知，生命周期判定需重算）**

---

## 0. 结论速览

| 方向 | 论文类型 | 判定 | 一句话理由 |
|---|---|---|---|
| **A · MOGP-U（原版：进化搜索 + 多目标遗忘损失）** | Novel Method | ❌ **Reject and Pivot** | 两个主卖点分别被 **EvoMU（arXiv:2602.02139）** 与 **HAMU（ICML 2026, arXiv:2606.02119）** 占据，且 HAMU 基座与设定和你完全重合 |
| **G↑ · ES/零阶 × 低秩子空间 × 部署鲁棒遗忘** | Novel Method | ✅ **Strong Accept** | 经检索确认**完全空白**；两条已发表实证可拼成可证伪假设；范式转移探针 4/4 |
| **B2 · 部署格点不变遗忘** | New Setting + Method | ✅ **Strong Accept** | 算力最低、确定性最高；把多目标从 2 维升到 2+\|G\| 维，绕开 HAMU/OFMU |
| **B1 · 可忘性（learn-to-be-unlearnable）** | Novel Problem | 🟡 **Accept with Revisions** | 叙事最漂亮，但须先证明"不是让模型记得更浅"，且需先排除一篇 2026 综述已占立论 |

**推荐执行顺序**：**G↑ 与 B2 二选一主攻（二者可合并），B1 作为备选。**

---

## 1. 方向 A · MOGP-U（原版）

### 1.1 First impression

- **Paper type**：Novel Method
- **One-sentence story**：现有遗忘方法用标量化加权和耦合 forget/retain 目标，我们用机制型语义 DSL + GP/NSGA-II 在固定预算下搜索可审计的遗忘损失函数。

这句话本身是清楚的、可讲的。问题不在表述，在** novelty 已被 concurrent work 吃掉**。

### 1.2 Fatal-flaws audit（早门）

| # | 缺陷 | 严重度 | 检测规则 | 防御 |
|---|---|---|---|---|
| **F1** | **Novelty 已被 concurrent work 占据**：核心卖点 S1（进化搜索遗忘损失）与 S2（显式多目标替代标量化）分别被 EvoMU 与 HAMU 占据 | 🔴 **CRITICAL** | F1 新颖性缺陷 + "Dominated by a prior method" | 见下方判定 |
| F2 | 实验设定与 HAMU 近乎完全重合（Llama-2-7B + TOFU），HAMU 且有理论阈值 κ₁/κ₂ 与停止准则 | 🔴 CRITICAL（并入 F1） | 同 F1 | — |
| F3 | SOTA 基线策略失效：OFMU（arXiv:2509.22483）已把 SimNPO 列为 baseline 并声称超越，"先复现 SimNPO 锁 SOTA"已不足 | 🟠 MAJOR | 基线不充分 | 把 OFMU、HAMU、EvoMU 全部纳入对比 |

**F1 的依据（已亲自核实 arXiv / ICML 官方页，2026-09-08）**：

| 卖点 | 占据者 | 证据 |
|---|---|---|
| S1 进化搜索遗忘损失 | **EvoMU, arXiv:2602.02139** | Qwen3-4B-Thinking 作 LLM 变异算子，进化搜索**遗忘损失代码 + 训练预算**，LoRA 微调，在 **TOFU-5%/10%/MUSE/WMDP 超越现有损失式方法**，代码开源 `github.com/Batorskq/EvoMU` |
| S2 显式多目标替代标量化 | **HAMU, arXiv:2606.02119, ICML 2026 已录用** | 约束优化视角，κ = g_r·g_f 同时作硬度度量/更新切换/停止条件；**Llama-2-7B + WaterDrum-TOFU**；代码开源 `github.com/aoi3142/HAMU` |
| （同向补刀） | **OFMU, arXiv:2509.22483** | 惩罚式双层优化，TOFU/WMDP/CIFAR，已把 NPO/SimNPO/RMU/GradDiff 列为手下败将 |
| （同向补刀） | MUNBa 2411.15537 / RAUL 2606.00399 / FUPareto 2602.01852 / 2410.22086 | 多目标/帕累托已成红海 |

**差异轴残存情况**：S3（语法约束 DSL 表示）🟡 部分未被占（遗忘领域内只有 EvoMU 的 LLM 自由代码版，无语法约束版）；S4（可审计性）🟢 完全未被占。**但这两条都太窄，单独撑不起一篇 CCF-A 的 novelty。**

> **关键提醒**：novelty 判定的铁律是"重复需要找不到任何一个差异轴"。S3/S4 确实构成差异轴，所以严格说不是完全重复。但从**审稿人行为**看，2026 年这个 abstract 会被同时引 EvoMU 与 HAMU，判为 concurrent/incremental —— 这在实践中等价于拒稿。

### 1.3 Verdict

**Reject and Pivot**（按此版本）

**Top three actions**：
1. **不要把"多目标"写成贡献点**——2026 年它已是 baseline 常识（HAMU 甚至给了理论阈值和停止准则）。把"多目标"降级为**方法**，把贡献点钉在**问题**上。
2. **换搜索对象**：不搜"遗忘损失"，改搜**"可忘性编码目标"**（方向 B1）或**"部署不变性正则"**（方向 B2）。进化/多目标工具箱原样保留，问题域是空的。
3. **或升维**：从 (forget, retain) 二维升到 (forget, retain, \|G\| 个部署格点)。HAMU/OFMU/MUNBa 全部只在 2 维上做，升维是结构性差异。

---

## 2. 方向 G↑ · ES/零阶 × 低秩子空间 × 部署鲁棒遗忘 【首推】

### 2.1 First impression

- **Paper type**：Novel Method
- **One-sentence story**：遗忘目前只能靠反向梯度执行；我们用进化策略在 **LoRA 低秩子空间**里直接搜索遗忘更新方向，目标是 (forget, retain, 部署格点鲁棒性) 三维帕累托前沿——从而同时得到**免反向传播**与**抗量化复活**两种现有方法给不了的性质。

**一句话点破**：**"进化"在这个方向里不是卖点，而是算子**——这是它和方向 A 的根本区别，也是它能绕开 EvoMU 的原因。

### 2.2 Fatal-flaws audit

| # | 缺陷 | 严重度 | 防御 |
|---|---|---|---|
| F1 | Novelty：ES/零阶优化作为**遗忘的执行算子**是否已被做？ | 🟢 **无** | 已检索 `ES + unlearning`、`zeroth-order unlearning`、`evolutionary unlearning`：**未检索到任何一篇用 ES/零阶优化直接执行遗忘的工作**。⚠️"未检索到"不等于"不存在"，建议投稿前再做一次针对性检索 |
| F2 | 动机风险：**遗忘本来就有梯度，为什么要用 ES？** 若答案只是"免梯度"，动机偏弱 | 🟠 MAJOR | 必须锚定 ES 的**不可替代优势**（见下 2.3 动机三锚点），不能只说"免梯度" |
| F3 | 语义混淆风险：【149】AWD 讲的是 ES 微调引发的**灾难性遗忘**（catastrophic forgetting），不是机器遗忘 | 🟡 MINOR | related work 中显式区分这两个 forgetting |

### 2.3 动机三锚点（必须至少占两个，否则动机不成立）

1. **黑盒 / API-only 遗忘**：真实场景（GPT/Claude/闭源商用模型）拿不到梯度，现有遗忘方法全部失效。库内【221】CBD 已开这条线，但只做行为分歧。**ES 是唯一能在纯前向 API 上执行参数级遗忘的算子**——不过黑盒场景下改不了权重，需澄清是"白盒权重 + 不可微目标"还是"真黑盒"。
2. **不可微目标**：如"通过 MIA 审计""top-p 采样下不泄漏（Leak@k）"这类离散/采样指标，现有方法要可微化（【42】用隐式函数定理），**ES 可绕过**。
3. **多目标帕累托前沿的免梯度探索**：NSGA-II 天然处理 3+ 目标，不需把目标可微化。

### 2.4 Lifecycle and capability match

| 方面 | 输入 | 评估 |
|---|---|---|
| Idea 类别 | Innovative Technique | — |
| 生命周期 | 12–18 个月（空白 + 需自建基线） | — |
| 每周有效工时 | 20–25h（假设） | 可行 |
| **Fit** | — | 🟢 **Green** |

### 2.5 Five-dimension radar

| 维度 | 分数 | 证据 | 提升建议 |
|---|---|---|---|
| **Higher** | **6** | 在标准白盒 TOFU 设定上，ES 不太可能超过 SimNPO（梯度信息更充分）。**不要在这里设战场** | 把主战场设在"梯度不可用或目标不可微"的设定，此处可到 8 |
| **Faster** | **6** | ES 免反向传播，但种群评估需 N 次前向。LoRA 参数化 + 种子重构（【146】）可把种群评估压到极低 | 报告"等效 GPU 时 vs NPO"的对照曲线，而非只报墙钟时间 |
| **Stronger** | **8**（**mechanism-based，未经验证**） | 核心卖点。两条已发表实证可拼成可证伪假设：① arXiv:2601.20861（UC Berkeley）证明 **ES 更新是"全局稠密 + 大范数"的**（ℓ₂ 比 GRPO 高 3 个数量级）；② 【327】证明**全参数微调的遗忘撑不过 4bit 量化，LoRA（低秩集中）才抗量化**。→ **假设：ES 式大范数稠密更新做出的遗忘在部署格点上极脆弱；加低秩/稀疏约束（AWD 锚定 + LoRA 参数化）后可得到既免反向传播、又抗部署扰动的遗忘算子。** | 命名验证实验（见 2.8） |
| **Cheaper** | **7** | 免梯度 → 显存省（无激活存储）；可在内存受限与 API 场景执行 | 量化"峰值显存 vs NPO/GDiff"的对照 |
| **Broader** | **9** | 可迁移到：黑盒 API 遗忘、不可微目标（MIA 审计/Leak@k 离散指标）、多目标帕累托免梯度探索、联邦/隐私场景（只传标量适应度不传梯度） | 这是最强的维度，**abstract 应该从这里切入** |

**Two dimensions at 8+**：Stronger (8) + Broader (9) ✅

### 2.6 Paradigm-shift probe

| 探针 | 是/否 | 理由 |
|---|---|---|
| **First Principles** | ✅ | 挑战"遗忘必须靠反向梯度执行"这一隐含假设 |
| **Elephant in the Room** | ✅ | 闭源商用模型（GPT/Claude）根本无法执行现有遗忘方法——这是所有人都看到、但没人碰的问题 |
| **Technology Cycle** | ✅ | ES-at-Scale（2025–2026）使 ES 在 LLM 上首次可行（【146】、Hyper-ES、EGGROLL、ESSA 等） |
| **Hamming's Rule** | ✅ | 若黑盒/免梯度遗忘成立，"模型厂商必须开放权重才能满足删除权"这一整条争论链会改变 |

**Disruptive potential：strong（4/4）**

### 2.7 Feasibility

| 风险 | 等级 | 缓解 |
|---|---|---|
| Compute | 🟡 中 | ES 种群评估成本是主要瓶颈。缓解：LoRA 参数化（r=8/16）+ 种子重构（只传 seed 与标量扰动，不传参数）+ 先在小模型（Phi-1.5/MiniCPM）验证再上 7B |
| Data | 🟢 低 | 完全复用 TOFU / MUSE，零额外数据成本 |
| Engineering | 🟡 中 | 需 ES 基建，但可直接基于已有 ES-at-Scale 实现改造，不必从零写 |
| Timeline | 🟡 中 | 6–9 个月。**注意：这是空白方向，需自建基线协议，比改一个损失慢** |

### 2.8 Verdict

**Strong Accept**（mechanism-based high scores → *worth pursuing, pending the validation experiment*）

**Top three actions**：
1. **先跑验证实验（1–2 周，决定是否值得做）**：在 TOFU Forget05 + LLaMA-2-7B-chat 上，用 ES（种群 32、LoRA r=16、种子重构）直接优化 forget/retain 双目标，**对照 NPO/SimNPO**。三个必须回答的问题：(a) ES 能否达到可接受的 forget quality？(b) 同等遗忘质量下，ES 的权重更新 ℓ₂ 范数是否显著大于 NPO？(c) 把两者都做 4bit 量化，ES 版是否如假设那样"复活"得更惨？**若 (c) 成立而加低秩约束后可修复——这个方向就成立了。**
2. **立即做一次针对性 novelty 复检**：检索 `zeroth-order optimization unlearning`、`derivative-free unlearning`、`black-box unlearning`、`ES unlearning`，确认空白。
3. **把动机钉死在 Broader 而非 Faster**：abstract 从"闭源模型与不可微目标下现有遗忘全面失效"切入，而不是"ES 更快"。

---

## 3. 方向 B2 · 部署格点不变遗忘（Deployment-Lattice Invariant Unlearning）【算力最低】

### 3.1 First impression

- **Paper type**：New Setting + Novel Method
- **One-sentence story**：现有遗忘只在 BF16 审计点成立，而真实部署要经历量化、剪枝、下游微调、蒸馏的组合；我们定义**部署格点** G = bitwidth × 稀疏率 × 下游微调步数 × 蒸馏，刻画遗忘在 G 上的失效结构，并给出跨格点一致的部署不变性正则。

### 3.2 Fatal-flaws audit

| # | 缺陷 | 严重度 | 防御 |
|---|---|---|---|
| F1 | 与【54】【276】【327】【259】高度相邻——单轴量化鲁棒已有人做 | 🟠 MAJOR | **把"单轴结论互斥"作为立论起点**：【434】"更简单优化器反而更抗逆转" vs 【327】"全参更新太分散撑不过 4bit，LoRA 更抗" vs 【54】"要大学习率局部更新"——**三条实证互相矛盾，说明这个空间还没被理清**。组合格点（0 篇）与单轴（已有）是不同问题 |
| F2 | 被判"只是工程评测" | 🟠 MAJOR | 必须给出**可优化的部署不变性正则**（对标【71】ILU 学不变表示），不能只报一张失效表 |

### 3.3 Lifecycle and capability match

| 方面 | 评估 |
|---|---|
| Idea 类别 | New Setting + Method |
| 生命周期 | 6–10 个月 |
| Fit | 🟢 **Green**（算力需求最低，与单机配置完全匹配） |

### 3.4 Five-dimension radar

| 维度 | 分数 | 证据 |
|---|---|---|
| **Higher** | **7** | 不追求更高遗忘率，而是定义"部署后有效遗忘率"这一更真实的目标：现有方法 BF16 达标的遗忘，4bit 后从 21% 回退到 83%（【54】）——**现有 SOTA 的实际部署有效遗忘率远低于其报告值** |
| **Faster** | **5** | 不提升效率；反而增加格点评估成本。无提升理由，维持 5 |
| **Stronger** | **9** | 核心贡献。跨整个部署格点 G 的组合一致性保证，是现有任何方法都没有的性质 |
| **Cheaper** | **8** | **算力极低**：3×3×3=27 格点 × 4 种算法 ≈ 108 次短程微调，每次 0.5–1.5 GPU 时 → **约 100–160 GPU 时** |
| **Broader** | **8** | 可作为**评估协议**被社区采用（填补 C6"解法有、标准无"的缺口），也可作为**正则项**插入任意遗忘方法；还能与认证理论结合（DP 认证结果目前全部假设全精度/无剪枝/无下游微调） |

**Two dimensions at 8+**：Stronger (9) + Broader (8) + Cheaper (8) ✅

### 3.5 Paradigm-shift probe

| 探针 | 是/否 | 理由 |
|---|---|---|
| First Principles | ✅ | 挑战"在训练/审计精度上验证过就算遗忘成功"这一隐含假设 |
| Elephant in the Room | ✅ | "BF16 审计通过 ≠ INT4 部署安全"所有人都知道，但**量化鲁棒未进入任何主流基准默认评估项** |
| Technology Cycle | ❌ | 无技术周期红利 |
| Hamming's Rule | ✅ | 若解决，"遗忘是否部署安全"的判定标准会从单点审计改为格点不变性 |

**Disruptive potential：strong（3/4）**

### 3.6 Feasibility

| 风险 | 等级 | 缓解 |
|---|---|---|
| Compute | 🟢 **低** | 约 100–160 GPU 时，单机可完成 |
| Data | 🟢 低 | 复用 TOFU/MUSE |
| Engineering | 🟢 低 | bitsandbytes 量化 + 稀疏化 + LoRA 微调，均为成熟工具 |
| Timeline | 🟢 低 | 3–5 个月可出结果 |

### 3.7 Verdict

**Strong Accept**

**Top three actions**：
1. **先画失效图**（2–3 周）：4 种代表性算法（GA/NPO/SimNPO/RMU）× 量化 {BF16, 8bit, 4bit} × 稀疏 {0, 30%, 50%} × 下游微调 {0, 100, 500 步}，测 forget quality 与 MMLU。预期能画出一张"现有方法在格点上大面积失效"的图——**这是立论的硬证据**。
2. **验证"单轴结论是否可外推"**：明确回答"量化安全 ⟹ 剪枝安全 是否成立"，预期**否**，给反例。这是与单轴已有工作划清界限的关键。
3. **设计部署不变性正则**：跨格点一致性项（对标【71】ILU），并在 27 格点上验证一致遗忘。**没有这一步就是纯评测论文，会被降档。**

---

## 4. 方向 B1 · 可忘性（Learn-to-be-Unlearnable）

### 4.1 First impression

- **Paper type**：Novel Problem
- **One-sentence story**：现有研究默认"知识一旦编码，其可删除性既定"；我们问前序问题——能否在 SFT 阶段以可微方式塑造知识编码几何，使目标知识事后遗忘时具有显著更低的遗忘成本？

### 4.2 Fatal-flaws audit

| # | 缺陷 | 严重度 | 防御 |
|---|---|---|---|
| F1 | **审稿人必问：这是否只是让模型记得更浅？** | 🔴 **可能 CRITICAL** | 必须证明 retain utility / MMLU / TruthfulQA / AlpacaEval **不降**。若降了，整个方向退化为"用效用来买可忘性"，与 C4 无区别 |
| F2 | **立论可能已被一篇 2026 综述占据**："Rethinking LLM Unlearning: From Safety Constraints to Functional Utility"（Xiaoyu Xu 等，2026）提出 **functional forgetting**，与本方向高度相关 | 🟠 MAJOR | **必须先读原文确认边界**（arXiv ID 未核实，需检索） |
| F3 | 与 HAMU 的 κ 太像，被判增量 | 🟠 MAJOR | 把 U 定义在**训练期（优化前的梯度几何）**并给理论上界；与 HAMU 构成"**怎么忘** vs **怎么记**"的对偶叙事，而非竞争 |
| F4 | 空白度：库内 0 篇提出可优化的编码目标 | 🟢 无 | 【490】只做观察、【231】是架构隔离、【108】是阻止知识形成——均非"让已形成的知识变得易删" |

### 4.3 Five-dimension radar

| 维度 | 分数 | 证据 |
|---|---|---|
| Higher | **5** | 不直接提升遗忘质量，无提升理由 |
| Faster | **8**（mechanism-based） | 事后遗忘所需的步数/学习率/效用损失显著下降 |
| Stronger | **6** | 间接提升（遗忘更温和 → 更少副作用），但无直接证据 |
| Cheaper | **8**（mechanism-based） | 遗忘成本（算力 + 效用损失）大幅下降；对一个要处理大量删除请求的服务商，这是真实的经济收益 |
| Broader | **7** | 可推广到训练期数据治理、合规-by-design；与 GDPR "设计即隐私"（privacy by design）天然契合 |

**Two dimensions at 8+**：Faster (8) + Cheaper (8) ✅（均为 mechanism-based）

### 4.4 Paradigm-shift probe

| 探针 | 是/否 | 理由 |
|---|---|---|
| First Principles | ✅ | 挑战"知识一旦编码，可删除性既定"这一从未被质疑的假设 |
| Elephant in the Room | 🟡 部分 | "遗忘很贵"是共识，但没人把它归因为编码方式 |
| Technology Cycle | ❌ | 无 |
| Hamming's Rule | ✅ | 若成立，数据合规会从"事后补救"转向"训练期设计" |

**Disruptive potential：possible（2.5/4）**

### 4.5 Feasibility

| 风险 | 等级 | 缓解 |
|---|---|---|
| Compute | 🟡 中 | 需一次受控 SFT + 18 次 7B LoRA 遗忘 → 约 200–350 GPU 时 |
| Data | 🟢 低 | 可用 `locuslab/tofu_ft_llama2-7b` 作"未经塑造"的天然对照，省一次 SFT |
| Engineering | 🟡 中 | 需实现训练期正则 + 事后遗忘的双层流程 |
| Timeline | 🟡 中 | 6–9 个月 |

### 4.6 Verdict

**Accept with Revisions**（mechanism-based → *worth pursuing, pending validation*）

**Top three actions**：
1. **先读"Rethinking LLM Unlearning: From Safety Constraints to Functional Utility"确认边界**——若 functional forgetting 已被形式化，本方向需重新定位。
2. **先做 F1 的自证实验**：加可忘性正则后，MMLU / TruthfulQA / AlpacaEval 是否不降？**这一步不通过就不要继续。**
3. **把 HAMU 的 κ 作为可用原语并入**（"我们的编码目标可调用难度感知门控"），把 HAMU 从竞争者变成组件——这是审稿人最喜欢的姿态。

---

## 5. 候选方向汇总表（可导入 Excel / Obsidian）

| 方向 | 类型 | 库内篇数 | Higher | Faster | Stronger | Cheaper | Broader | 范式转移 | 算力(GPU时) | 周期 | 判定 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| A · MOGP-U 原版 | Novel Method | — | — | — | — | — | — | — | — | — | ❌ Reject and Pivot |
| **G↑ · ES/零阶 × 低秩 × 部署鲁棒** | Novel Method | **0** | 6 | 6 | **8** | 7 | **9** | **4/4 strong** | 200–400 | 6–9 月 | ✅ **Strong Accept** |
| **B2 · 部署格点不变遗忘** | New Setting+Method | 0（单轴 11） | 7 | 5 | **9** | **8** | **8** | 3/4 strong | **100–160** | 3–5 月 | ✅ **Strong Accept** |
| B1 · 可忘性 | Novel Problem | 0（3 沾边） | 5 | **8** | 6 | **8** | 7 | 2.5/4 possible | 200–350 | 6–9 月 | 🟡 Accept with Revisions |
| B3 · 参数×外置记忆联合遗忘 | Novel Problem | 0–1 | 6 | 5 | 8 | 6 | 8 | 3/4 | 150–250 | 6–9 月 | 🟡 未评分（备选） |
| B4 · 遗忘前审计 / Out-of-Knowledge | Novel Problem | 0（2 提问） | 5 | 7 | 6 | 8 | 6 | 2/4 | 100–200 | 4–6 月 | 🟡 未评分（备选） |

> 注：B3/B4 未做完整五维评分，仅列出供备选。若主攻方向受阻，B3（RAG 时代，零额外数据成本）是次优选择。

---

## 6. 跨方向铁律（三条，写在开头也写在结尾）

1. **不要把"多目标"写成贡献点** —— 2026 年它已是 baseline 常识（HAMU 甚至给了理论阈值和停止准则）。**把"多目标"写成方法，把贡献点钉在问题上。**
2. **不要在 TOFU Forget05 单点上争 0.97 → 0.99** —— 这不构成 CCF-A 贡献，而且你自己就是 SimNPO 的复现者，做变体等于和自己抢饭吃。
3. **任何"SOTA"声明都要附鲁棒性脚注** —— 无量化存活、无再学习复活、无对抗提示复活、概率解码下不泄漏。2025–2026 的评估生态已把"静态单点指标 SOTA"降级为**必要非充分条件**。

---

## 8. 主攻方向建议（2026-09-17 追加）：G↑ × B2 合并版

### 8.1 结论

**主攻：免梯度低秩遗忘 × 部署格点鲁棒（G↑ 与 B2 合并为一个 idea，B2 的格点失效图作为立论硬证据，G↑ 的 ES 算子作为方法主体）。**

执行顺序上**先用 B2 打底（低风险先手棋），再用 G↑ 定方法**：

| 步骤 | 内容 | 成本 | 产出 |
|---|---|---|---|
| S1 | 部署格点失效图：GA/NPO/SimNPO/RMU × 量化{BF16,8bit,4bit} × 稀疏{0,30%,50%} × 下游微调{0,100,500步} | 约 100–160 GPU 时，2–3 周 | 一张"现有方法在格点上大面积失效"的图（几乎必然出结果），同时是 B2 与 G↑ 共用的 motivation |
| S2 | G↑ 验证实验：ES（种群 32、LoRA r=16、种子重构）直接优化 forget/retain，对照 NPO/SimNPO 的权重 ℓ₂ 范数 + 4bit 复活测试 | 约 50–100 GPU 时，1–2 周 | 决定 G↑ 生死的三个问题：(a) ES 能否达到可接受 FQ；(b) 同等 FQ 下 ES 更新 ℓ₂ 是否显著大于 NPO；(c) ES 版是否复活更惨、低秩约束能否修复 |
| S3 | 根据 S1+S2 定论文主轴，写 proposal | — | — |

### 8.2 Fatal-flaws audit（合并版）

| # | 缺陷 | 严重度 | 防御 |
|---|---|---|---|
| F1 | "两个空白拼接不算 novelty"的质疑 | 🟠 MAJOR | 强调机制必然性：ES 更新大范数稠密（arXiv:2601.20861）⇄ 量化桶宽抹平小 ΔW（【54】【259】）——**一个是遗忘算子的性质，一个是部署算子的性质，两者相撞才有此问题**，非随意组合 |
| F2 | "遗忘本来有梯度，为什么要 ES" | 🟠 MAJOR | 动机钉在 Broader：黑盒/闭源模型、不可微目标（MIA 审计、Leak@k 采样指标）、联邦场景（只传标量适应度） |
| F3 | ES 在白盒 TOFU 上打不过 SimNPO | 🟡 MINOR | 不在白盒单点设战场；S2 的对照实验只为刻画性质，不为争 SOTA |

无 CRITICAL → 进入完整评分。

### 8.3 五维雷达（合并版）

| 维度 | 分 | 依据 |
|---|---|---|
| Higher | 6 | 白盒主战场不打 SimNPO；黑盒/不可微战场可到 8 |
| Faster | 6 | 免反传但种群前向有开销；LoRA+种子重构可压 |
| **Stronger** | **9**（机制-based） | 双重鲁棒：部署格点不变 + 免梯度；S1 失效图将提供实证 |
| Cheaper | 7 | 显存省（无激活存储）；合并版总预算 250–450 GPU 时 |
| **Broader** | **9** | 黑盒 API 遗忘、联邦、不可微目标、多目标免梯度帕累托——**abstract 从这里切入** |

**范式转移探针 4/4**（继承 G↑）：挑战"遗忘必须反向梯度"+"BF16 审计即安全"两个隐含假设；ES-at-Scale 技术周期红利；若成立则"厂商必须开权重才能满足删除权"的争论链改变。

### 8.4 Verdict

**Strong Accept**（mechanism-based → *worth pursuing, pending S1/S2 validation*）

**为什么不是其他**：
- **纯 B2**：算力最低、确定性最高，但纯评测易被降档 → 降级为 S1 motivation 模块最划算；其"部署不变性正则"可作论文第二贡献。
- **B1 可忘性**：叙事最漂亮，但必须先读 "Rethinking LLM Unlearning: … Functional Utility"（2026，ID 未核实）排雷，且"记得更浅"质疑需全量效用证据自证 → 作第二篇储备。
- **B3/B4**：备选，不作主攻。
- **MOGP-U 原版**：Reject and Pivot（EvoMU + HAMU 已占，见 8.2 前文）。

**投稿窗口**：NeurIPS 2027（约 2027-05 截稿）最稳；ICML 2027（约 2027-01 底）需 S1+S2 在 11 月中前完成。

## 9. 待办 / 需您确认

1. **每周有效工时**：本文按 20–25h/周假设，若实际不同，B1 与 G↑ 的周期判定需重算（G↑ 需自建基线协议，比改一个损失慢）。
2. **必须亲自核实的两条**（影响 B1 与 G↑ 的立论边界）：
   - "Rethinking LLM Unlearning: From Safety Constraints to Functional Utility"（Xiaoyu Xu 等，2026）—— 其 **functional forgetting** 是否已占据 B1 的立论。
   - 针对性检索 `zeroth-order / derivative-free / black-box unlearning`，确认 G↑ 的空白性。
3. **27 篇 OpenReview 论文本地缺 PDF**，其中 *The Fundamental Limits of LLM Unlearning*、*White-Box Auditing of LLM Unlearning*、*LUSB* 与本报告高度相关，建议补齐后复核。

## 10. 主攻方向的三段式拆解（2026-09-17 追加）：问题 → 现状 → 创新解法

> 本节是 G↑ × B2 合并版的论文 Introduction 骨架。所有编号【n】可定位本地 PDF。

### 10.1 面临的问题（四层问题链）

**单行总结**：现有遗忘 = 在「白盒 + 全精度 + 单点审计」这个最理想设定下的概率压制——真实部署中这三个前提**全部不成立**。

| 层 | 问题 | 机制根因 | 证据 |
|---|---|---|---|
| **P1 执行层** | 遗忘依赖反向梯度 | 现有全部方法（GA/NPO/SimNPO/RMU/蒸馏/表征编辑）都要求拿到权重与梯度；闭源模型与不可微目标（MIA AUC、Leak@k 采样指标）下**无法执行** | 【221】CBD 只能做行为分歧；【42】需隐式函数定理把 MIA 可微化，近似误差大 |
| **P2 鲁棒层（核心）** | 部署即复活 | 遗忘把参数推到**锐利、浅层**极小值；量化桶抹平小 ΔW、下游微调沿原梯度拉回 | 【54】全精度保留 21% → 4bit **83%**；【276】BF16 审计通过 ≠ INT4 安全；【174】良性微调唤醒；【514】retain 集微调 50%→近 100% |
| **P3 几何层** | 更新两难 | 要抗量化桶，ΔW 必须"大而集中"；要保效用，ΔW 必须"小而分散"——全参微调两头不占 | 【327】全参更新太分散撑不过 4bit，LoRA 低秩集中才抗；【54】要大学习率局部更新；【434】反直觉：更简单优化器反而更抗逆转——**三条实证互相矛盾，说明几何空间未被理清** |
| **P4 评估层** | 单点审计系统性乐观 | 评估在 BF16 + 贪心解码 + 单点完成，与部署分布脱节 | 【410】Leak@k：概率解码下全线泄漏；【581】全概率评估更悲观；**量化鲁棒未进任何基准默认评估项** |

### 10.2 已经实现了什么（四条线，各自到哪、差什么）

| 线 | 已实现 | 缺口 |
|---|---|---|
| **量化鲁棒遗忘（单轴）** | SURE【54】显著性大 LR 局部更新；DurableUn【276】量化恢复攻击刻画；LoRA 低秩集中【327】；QUAIL【348】量化感知遗忘（量化桶重叠解释）；GROM【152】闭式解天然抗量化 | **全部单轴**：无组合格点（量化×稀疏×微调×蒸馏），无跨轴联合保证；且【54】【327】【434】结论互斥，无统一几何解释 |
| **ES/零阶做 LLM** | ES-at-Scale【146】使 ES 微调 LLM 可行（种子重构降成本）；Hyper-ES / EGGROLL / ESSA 等基建；AWD（arXiv:2605.30148）锚定权重衰减修复 ES 的灾难性遗忘；arXiv:2601.20861 刻画 ES 更新大范数稠密（ℓ₂ 比 GRPO 高 3 个数量级） | **无人把 ES 用作遗忘的执行算子（0 篇）**；ES 的稠密性在微调语境是缺陷，其遗忘语境的含义未被研究 |
| **黑盒遗忘** | CBD【221】行为分歧（仅输出层）；ALU Agent 工作流（不碰权重）；ICUL【117】上下文遗忘 | **参数级遗忘在黑盒/不可微目标下 0 篇**；上下文类方法可被绕过【189】 |
| **评估** | Leak@k【410】、FPE【581】、OpenUnlearning 元评估【154】（ES/EM 最可靠） | 部署格点不变性**未进任何基准**；"部署后有效遗忘率"概念不存在 |

### 10.3 创新如何解决（三创新点 → 四问题映射）

| 创新 | 解决 | 机制 | 为什么成立（证据支点） |
|---|---|---|---|
| **C1 · ES 免梯度遗忘算子** | P1 | 在 LoRA 参数空间做进化搜索，每个个体 = 一组低秩扰动；适应度 = 前向 rollout 直接测得的 (forget quality, retain utility, 甚至 MIA AUC / Leak@k)——**不可微指标无需可微化，直接进适应度** | ES-at-Scale 已证 ES 在 LLM 上可行【146】；种子重构使种群评估只需传 seed + 标量；【42】用隐式函数定理可微化 MIA 的近似误差，ES 天然绕过 |
| **C2 · 低秩子空间约束** | P3（连带缓解 P2） | 把 ES 搜索限制在 LoRA 低秩空间：**核心反转——ES 大范数稠密更新在无约束时是缺陷（复活更惨），低秩参数化把它转为"更新集中"的优点**（借【327】的实证：低秩集中才抗量化桶） | 【327】LoRA 低秩集中撑过 4bit；AWD 的锚定机制可借鉴为约束实现；【54】大学习率局部更新与低秩集中方向一致 |
| **C3 · 部署格点不变性目标** | P2 + P4 | 定义格点 G = bitwidth × 稀疏率 × 下游微调步数 × 蒸馏；目标从 (forget, retain) 二维升到 **(forget, retain, 格点一致性) 3+ 维**，NSGA-II 免梯度探索帕累托前沿；产出新指标「部署后有效遗忘率」 | 【54】【276】已证单点审计失效；HAMU/OFMU/MUNBa 全部只在 2 维做，**升维是结构性差异**；格点评估计算廉价（前向 + 量化，无需重训） |

**收尾判断**：这个 idea 的本质是一句话——**把 ES 的已知缺陷（大范数稠密）通过低秩参数化转化为已知优点（更新集中抗量化），并用部署格点把"遗忘成功"的定义从单点审计升级为格点不变性**。S1（格点失效图）提供"问题存在"的硬证据，S2（ES vs NPO 的 ℓ₂ 与 4bit 复活对照）提供"解法有效"的第一证据，两者都是低成本先手棋。

### 10.4 风险与证伪点（诚实版）

1. **S2 可能证伪主假设**：若 ES 版遗忘在 4bit 下并不比 NPO 更惨、或低秩约束修复不了，则 C2 的"反转"叙事不成立——退路是纯 B2（部署格点评测+正则）论文，S1 的图依然可用。
2. **ES 的遗忘质量上限**：若种群规模 × 预算内 ES 达不到可接受 FQ，C1 不成立——退路同上，或转向"ES 只用于格点一致性正则的系数搜索"（回到 DSL 搜索但对象是正则系数）。
3. **黑盒声明的边界**：ES 仍需白盒权重（LoRA 参数空间）；"黑盒"仅指**不可微目标/无梯度**场景，论文里必须如实限定，不可声称纯 API 黑盒遗忘。
