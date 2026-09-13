# 购物 Agent RL 项目问答面经

> 这是一份基于源码事实生成的口播训练稿，不是个人经历证明。所有第一人称答案都是候选模板；只有你确实完成了复现、修改或实验，才能去掉“待确认/完成后方可使用”。面试时宁可清楚说明“我基于开源项目做了复现与分析”，也不要把作者或贡献者的工作冒充为自己完成。

## 1. 简历可用项目简介

### 当前安全版本（尚未完成个人复现时）

研读并复现基于 ShopSimulator v2.1 的长程购物 Agent 后训练项目，梳理 Baseline、LoRA SFT、在线 GRPO 与 Final-200 Clean 评测闭环；重点分析严格轨迹验收、assistant-only 标签掩码、Reward v3、Observation 安全投影、动作守卫、动态 group 采样与盲测隔离机制，并通过源码和单元测试建立可核验的项目理解。

### 完成复现后方可使用的版本

基于 ShopSimulator v2.1 搭建长程购物 Agent 的 Baseline→SFT→GRPO→Evaluation 闭环，使用 LoRA SFT 学习多轮工具协议，接入 veRL 0.8 在线 GRPO 与确定性 Reward v3；通过 Observation 投影、Action Guard、task-ID 数据隔离和固定分母评测保障长上下文执行安全与实验可信度。`待补充：本人实际模型、数据规模、开发集指标、代码改动、硬件和产物哈希。`

## 2. 简历 Bullet 候选

- **评测先行与盲测隔离：** 研读 Final-200 Clean 固定测试协议，理解基于内容 SHA-256 与 task-ID 的双重泄漏保护，以及缺失任务仍保留在 200 题分母中的严格统计方式；`待确认：是否亲自运行并产出评测文件。`

- **确定性奖励与证据有效性：** 分析 Reward v3 的 category/budget 硬门槛、brand/model/function/option 软匹配和九类终局状态，能够区分严格成功、替代购买、部分满足与 reward-unverifiable；`待确认：是否修改或测试过奖励逻辑。`

- **长上下文投影与动作治理：** 梳理搜索页、详情页和通用页的差异化 token 预算，理解压缩后保持 ASIN/按钮动作集合一致及 Action Guard 只允许当前页目标的设计；`完成实际修复后方可写“实现/优化”。`

- **高质量轨迹与 Action-only SFT：** 核验 2,498 条原始轨迹→1,026 条严格通过→固定 1,000 条→800/200 划分的数据链路，理解只对 assistant 自然语言与工具调用计算 loss 的标签构造；`待确认：是否亲自训练。`

- **在线策略优化与动态采样：** 分析每题 4 条 rollout 的组内相对优势，理解过滤常量 utility group、剔除 sampling-invalid group、最多补采 3 批与连续跳步上限的有界机制；`待确认：是否运行 GRPO 或完成消融。`

- **可审计多面板评测：** 梳理代码硬检查、冻结 Rubric、Judge-safe trajectory 与 task-ID 配对比较，避免把 Reward、需求满足、轨迹质量和基础设施状态压成单一总分；`待确认：是否生成个人结果。`

## 3. 高频主问题

### Q1. 请用两分钟介绍这个项目

**第一人称 STAR 口播（模板）**

情境上，我关注的是长程购物 Agent：模型不仅要理解需求，还要在网页环境中持续搜索、核验属性、选择规格并合法终止，任何一步的非法点击、上下文丢失或错误奖励都会让最终结果失真。我的任务是把开源项目从“能运行”拆成可学习、可审计的训练闭环。我按 Baseline→SFT→GRPO→Evaluation 阅读并复现：先用严格成功教师轨迹做 LoRA SFT，再让策略在 ShopSimulator v2.1 中在线采样，由 Reward v3 提供确定性终局信号，最后在 Final-200 Clean 上用固定分母和多面板指标比较。结果上，我目前能从源码解释数据验收、动作守卫、动态采样和盲测隔离；仓库公开指标只作为背景，我不会把它们表述成个人实验结果。

**追问 1：这个项目最难的地方是什么？**

情境上，购物 Agent 的难点不只是模型能否生成工具 JSON，而是一次决策跨越搜索、候选比较、证据验证和规格选择，错误可能来自策略、环境、奖励或基础设施。我的任务是找到真正决定可信度的工程边界。我重点追了三条链：Observation 压缩后是否仍保留全部可执行目标，Reward 不可验证时是否会被错误当成零分训练，以及测试 task ID 是否可能进入训练。行动上，我逐个对照投影器、Reward validator 和 blind guard 的 fail-closed 分支。结果是我把“成功率高低”拆成了可定位的问题，并能解释为什么端到端 Agent 的难点往往在契约一致性，而不仅是训练算法本身。

**追问 2：你个人在项目中做了什么？**

情境上，这个仓库来自开源作者，我不能把已有训练与公开评测认领为自己的成果。我的真实任务是把它变成可复现、可面试的个人学习项目。当前我完成的是源码级结构梳理、关键指标口径核验、当前数据与历史实验的分离，以及 Reward、SFT mask、动态采样和评测隔离的证据索引。行动上，我为每个结论定位到具体函数和配置，并把尚未运行的训练、Final-200 评测标成待确认。结果上，我已经能诚实讲清系统；下一步只有在跑出自己的日志、commit、checkpoint 和开发集结果后，我才会把“分析”升级为“实现或优化”。

### Q2. 为什么流程必须是 Baseline→SFT→GRPO→Evaluation？

**第一人称 STAR 口播（模板）**

情境上，Base 模型面对的是多轮工具协议，如果直接做 RL，大量 rollout 会浪费在格式错误、历史页面点击和不会终止上，Reward 反映的主要是协议失败而非购物决策。我的任务是让每个阶段回答不同问题。行动上，我先用 Baseline 建立原始能力与失败分布，再用严格通过的教师轨迹做 SFT，让模型获得搜索、核验、选择和购买的 workflow prior；随后用在线 GRPO 优化同题不同轨迹的相对终局结果；最后只在冻结的留出集做一次横向比较。结果上，这条顺序把协议学习、策略优化和泛化验证分开，既降低在线采样成本，也避免用最终测试集选择 checkpoint。

**追问 1：为什么不能只做 SFT？**

情境上，SFT 的目标是复现教师在训练分布中的动作，它能显著改善工具格式和基本流程，但不会直接优化模型在自身分布下遇到的新状态。我的任务是说明 RL 的增量价值而不夸大。行动上，我把 SFT 看成行为先验，把 GRPO 看成在线纠偏：策略自己采样四条轨迹，环境根据真实购买结果给终局 utility，同题内有好坏差异时才产生相对优势。结果上，GRPO理论上能学习教师示范未覆盖的恢复与取舍；但历史 2B 实验从 60.5% 到 62.0% 只有 1.5 个百分点，且来自旧测试集，所以我会说“观察到小幅历史增益”，不会宣称统计显著或普遍有效。

**追问 2：为什么不能直接在 Final-200 上调参？**

情境上，Final-200 Clean 是最终泛化估计，如果拿它挑 prompt、Reward 参数或 checkpoint，模型虽然没有直接做梯度更新，也会通过人工决策间接过拟合测试集。我的任务是保证实验结论仍有可信含义。行动上，我遵循仓库的盲测契约：正式文件只公开 task ID，训练入口用内容哈希和 ID 集合双重检查，开发选择使用 SFT validation 或 GRPO validation，最终测试只做冻结横向比较。结果上，测试成功率才近似反映未见任务表现；任何在 Final-200 上反复试出来的数字，我都会明确降级为开发结果而不是正式 benchmark。

### Q3. Agent 如何和 ShopSimulator 交互？

**第一人称 STAR 口播（模板）**

情境上，模型不能直接操作网页 DOM，而是通过稳定的工具 schema 与环境交互。我的任务是确认从模型输出到环境状态变化的完整边界。行动上，我从 `SHOP_TOOL_SCHEMAS` 追到 `tool_call_to_action`：模型可以搜索、打开当前页商品、查看详情子页、选规格、翻页、返回、购买或合规放弃；每次只执行一个工具，环境返回 observation，Agent 再做下一步。GRPO 中每条异步 trajectory 会绑定独立环境 session 和 coroutine-local runtime state，并在退出路径释放租约。结果上，模型看到的是规范化的 observation v2，不接触隐藏 goal、Reward 或 target ASIN，训练与评测共享同一套执行语义。

**追问 1：为什么强制一次只调用一个工具？**

情境上，多工具并行调用在普通 API 场景可能提高吞吐，但网页购物动作具有严格状态依赖：打开商品以后，可点击按钮和可选规格已经变化，第二个基于旧页面生成的调用可能立即失效。我的任务是维护轨迹的可解释顺序。行动上，rollout 层保留 assistant 的第一个工具调用并记录被丢弃的其余调用，数据验收还会拒绝包含多工具调用的训练消息。结果上，每个 action 都有明确的前置 observation，Guard 可以验证“点击目标来自最新页面”，评测也能稳定生成事件 ID；代价是交互步数增加，但状态一致性和可审计性更重要。

**追问 2：如何确保异常时环境资源不泄漏？**

情境上，GRPO 同时运行多个 worker，任何超时、模型解析错误或 Guard 连续拒绝都可能让环境租约滞留，继而污染后续 rollout。我的任务是把异步训练框架和同步环境客户端隔离。行动上，我使用 session 对象负责 reset、绑定 context-local 环境与 runtime state，并把阻塞网络调用放到线程执行；无论正常完成还是抛出异常，都在清理路径释放租约并重置上下文。结果上，单条轨迹的状态不会串到另一条轨迹，release failure 也能被标记为 infrastructure invalid，而不是伪装成模型失败。

### Q4. Reward v3 是怎样设计的？

**第一人称 STAR 口播（模板）**

情境上，购物任务既有不能违反的硬条件，也有可连续衡量的偏好匹配，单纯按目标 ASIN 相等打分会拒绝合理替代品，单纯按文本相似又难以审计。我的任务是把二者结合成确定性终局 Reward。行动上，我先检查 category 和 budget 两个 hard gates；硬条件通过后，再按 brand 0.35、model 0.25、core functions 0.25、key options 0.15 计算 weighted score 和 evidence coverage。目标 ASIN 且全部满足为 1.0 的 gold purchase，合格替代品为 0.55，部分满足按公式给有限分，错误购买为 -0.85。结果上，Reward 不依赖另一个 LLM，重复执行可得到一致结果，并能输出可审计的类型和证据。

**追问 1：为什么 `reward_unverifiable` 不是普通失败？**

情境上，如果 variant 价格解析失败或硬门槛缺少证据，系统并不知道模型选得对还是错；把这种情况直接记为 0 或负分，会把环境与评分器缺陷错误传播给策略。我的任务是把“策略失败”和“无法判定”分开。行动上，Reward v3 返回 `reward_unverifiable`，设置 `reward_valid=false` 与 `sampling_invalid=true`；runtime 还严格校验只有这一类型可以令 reward_valid 为 false。动态采样看到组内任一无效轨迹就丢弃该组。结果上，无证据样本不会制造伪梯度，同时仍保留诊断信息，便于修复价格解析或数据质量问题。

**追问 2：为什么允许替代商品成功，却把 strict success 限定为 Gold？**

情境上，用户需求可能被多个商品满足，因此 Reward 需要认可合理替代品；但基准测试若把“可能合理”直接等同唯一 Gold，会使不同版本的比较口径漂移。我的任务是同时保留语义合理性与严格可比性。行动上，我让 `valid_alternative_purchase` 计入 purchase success 并获得 0.55，但 strict success 只认完整终局的 `gold_purchase`、`reward_valid=true` 和 `purchase_success=true`。结果上，研究者既能看严格复现目标商品的能力，也能看更宽松的购买成功，不会因替代品定义变化而悄悄抬高主指标。

### Q5. 如何处理长 Observation 和非法动作？

**第一人称 STAR 口播（模板）**

情境上，搜索结果和商品详情很长，直接累积可能超过 24,576 token；粗暴截断又可能删掉 ASIN 或 Buy Now 按钮，让模型看到的动作空间和环境实际动作空间不一致。我的任务是在压缩上下文时保持可执行性。行动上，我按页面类型使用 1,536/4,096/768 token 预算，搜索页对标题字段做二分压缩，详情页保留正文首尾，并完整保留搜索状态、当前页全部 ASIN 和按钮 footer；压缩后再次校验 token、ASIN 和按钮集合。动作守卫再拒绝不在最新 observation 中的目标。结果上，长上下文受控且不会凭空生成可点击目标，任何无法安全投影的页面都会显式失败。

**追问 1：Action Guard 会不会掩盖模型真实能力？**

情境上，Guard 能阻止非法动作污染环境，但它也相当于给模型额外反馈，可能提高系统层面的完成率，因此不能只报告最终成功率。我的任务是把安全脚手架的贡献显式化。行动上，我在轨迹里保存每次 blocked tool call、拒绝原因、是否发生在截断后，并在汇总中统计 Guard 次数和拒绝率；Baseline、SFT、GRPO 使用同一 Guard 与工具 schema，保证横向条件一致。结果上，我可以同时回答“端到端系统是否可靠”和“模型自身是否经常想执行非法动作”，不会把 Guard 修正后的系统表现冒充成裸模型能力。

**追问 2：为什么投影搜索结果时要求保留当前页全部商品？**

情境上，如果投影器只保留它认为最相关的 top-k，而环境仍允许点击被隐藏商品，模型可见信息和动作集合就不等价；若排序器使用目标信息，还可能引入隐性答案泄漏。我的任务是让投影只压缩表述，不替模型做候选选择。行动上，项目限制环境页容量并保留当前页所有 product ASIN，只压缩标题等字段，同时验证可见 ASIN 与可点击商品按钮集合完全一致。结果上，模型仍对当前页完整候选负责，投影不会改变决策问题；代价是预算紧张时标题信息更短，因此还要按截断桶报告成功率和 Guard 行为。

## 4. 中频深入问题

### Q6. SFT 数据为什么只有约 41% 被验收？

**第一人称 STAR 口播（模板）**

情境上，教师模型生成 2,498 条在线轨迹，其中虽然有 1,742 条终局类型看似 gold purchase，但一些轨迹仍包含 Guard 违规、消息结构问题或不可用于训练的动作。我的任务是构造高精度而不是高数量的数据集。行动上，我对每条轨迹检查完整终局、真实 buy_now、Reward v3 版本、gold_purchase、reward_valid、purchase_success、终止原因、单工具调用和逐步动作合法性，并排除 held-out 与重复 task。最终严格通过 1,026 条，接受率 41.07%，固定使用 1,000 条划分 800/200。结果上，SFT 学到的是可回放的正确动作链，而不是只看最终标签的带噪轨迹。

**追问 1：为什么有 1,742 个 gold 结果却只接受 1,026 条？**

情境上，终局 Reward 只说明购买结果满足评分条件，并不能证明整条教师轨迹适合模仿。例如模型可能先产生 Guard 拒绝，再纠正后买对；若把被拒绝调用也作为示范，SFT 会同时学习错误和恢复。我的任务是把结果正确性与行为可训练性分开。行动上，验收器收集所有 rejection reasons，特别剔除 guard violation、多工具并发、工具名或参数与环境 action 不一致等轨迹，而不是只筛 `reward_type`。结果上，716 条包含 Guard 违规的轨迹不会混入训练；这降低了数量，却提高了动作标签的纯度和可解释性。

**追问 2：严格只收成功轨迹有什么副作用？**

情境上，高精度筛选能教会正确流程，但也会造成成功路径偏置：模型看不到错误恢复、合理放弃和困难任务中的局部正确动作，而且同一个教师会降低策略多样性。我的任务是承认这一权衡并设计验证。行动上，我会统计工具序列、搜索改写、候选覆盖与轨迹长度多样性，并比较早中晚 SFT checkpoint 的 validation loss、生成熵和 GRPO 有效 group 比例；若晚期模型四条 rollout 过于相同，就考虑增加多教师或包含经审计的恢复片段。结果上，数据质量不再等同“越严格越好”，而是以协议正确和在线可塑性的平衡为目标。

### Q7. SFT 为什么只计算 assistant token 的 Loss？

**第一人称 STAR 口播（模板）**

情境上，一条工具轨迹同时包含 system 规则、user 需求、assistant 决策和 tool observation。如果对整段文本计算 causal LM loss，模型会被训练去复述用户和环境，而不是专注生成下一步动作。我的任务是准确构造监督区间。行动上，我先用 tokenizer 的 chat template 渲染完整轨迹，labels 默认全部设为 -100；对每个 assistant 回合，分别渲染前缀 generation prompt 与包含该回合的文本，通过 token 公共前缀定位新增区间，只开放 assistant 自然语言和 tool-call token。结果上，输入上下文完整保留，但梯度只来自模型负责的输出，同时避免手写 Qwen 特殊 token 带来的模板漂移。

**追问 1：为什么不用字符位置直接切标签？**

情境上，字符边界与 tokenizer 边界并不一一对应，chat template 还会插入角色标记、工具定义和特殊 token；按字符串长度切分很容易出现一个 token 被跨界、起始标记漏训或模型版本更换后错位。我的任务是让标签构造跟实际模型输入完全一致。行动上，我使用同一个 chat template 生成前缀和带当前 assistant 回合的 token 序列，再求最长公共前缀作为 start，end 取后一序列长度，并校验该 token 前缀与完整输入一致。结果上，标签边界由真实 tokenization 决定，若模板出现不兼容就直接报错，而不是静默训练错误标签。

**追问 2：Tool observation 完全不训练，会不会浪费信息？**

情境上，tool observation 对决策非常重要，但它的角色是条件输入，不是模型要生成的目标；不计算 loss 并不等于模型看不到它。我的任务是让模型学会“读环境后行动”，而不是“预测环境会返回什么”。行动上，我把 tool token 保留在 input_ids 与 attention mask 中，使后续 assistant token 的隐状态能利用价格、属性、按钮和错误反馈，只把对应 labels 设为 -100。结果上，模型仍能从 observation 获取因果条件，同时梯度不会奖励它背诵环境文本；如果要验证，我会比较工具内容改变时下一动作是否同步变化，而不是查看 tool token loss。

### Q8. 项目如何防止训练集与评测集泄漏？

**第一人称 STAR 口播（模板）**

情境上，Agent 数据不是普通 IID 文本；同一 task 的不同 rollout、同一目标商品的改写任务都可能让测试表现虚高。我的任务是先保证最基本且可执行的 task-ID 隔离。行动上，我在采集入口读取 `data/evaluation/tasks.jsonl` 的 held-out IDs，在 SFT 构建时再次排除并去除重复 task；GRPO 数据元信息也声明排除当前 SFT 与冻结 200 题。训练入口调用 blind guard，既校验 Final-200 文件内容 SHA-256，也扫描任意输入产物中的 task ID。结果上，即使测试文件被改名或嵌套在 extra_info 中，重叠仍会被拒绝；当前 metadata 记录 training overlap 为 0。

**追问 1：task-ID 零重叠就完全没有泄漏了吗？**

情境上，task-ID 隔离只能证明没有完全相同的任务标识，不能排除相似 Query、同一目标 ASIN、同型号商品或模板化需求跨集合出现。我的任务是准确界定现有保护的能力。行动上，我会把 ID guard 称为必要条件而不是充分条件，并建议增加 Query 近重复聚类、目标商品/品牌/型号交叉统计、品类约束分布和语义相似度审计；这些分析不能用隐藏 Gold 反向调模型，只能用于数据集治理。结果上，面试时我不会把“overlap=0”夸成绝对无泄漏，而会说明下一层污染风险与可验证方案。

**追问 2：为什么 blind guard 要同时检查哈希和 ID？**

情境上，只检查文件路径很容易被复制改名绕过；只检查整体哈希又会漏掉从测试集抽取部分行后生成的新文件。我的任务是覆盖完整复制和部分混入两种风险。行动上，guard 先验证打包资源中的测试契约和任务集合，再对每个输入文件计算 SHA-256，同时解析 JSONL 顶层、extra_info 或 normalized trajectory 中的 task_id，与冻结 ID 集合求交。结果上，原样复制会命中内容哈希，抽样或改名会命中 ID overlap；遇到任何一种都 fail closed，除非显式进入允许评测的路径。

### Q9. GRPO 在这个项目里怎样工作？

**第一人称 STAR 口播（模板）**

情境上，SFT 后策略已经会使用工具，但它仍可能在候选比较、规格核验和终止决策上犯错。我的任务是用真实环境反馈做在线策略优化，同时控制长轨迹训练成本。行动上，我用 veRL 0.8 和 vLLM 异步 rollout，对每个 prompt 采样 4 条轨迹，温度 0.7、top-p 0.9；环境正常终止后读取 Reward v3 terminal utility，按同题 group 构造相对优势，再用 LoRA rank 16、学习率 1e-6 做 clipped policy update，KL reward/loss 均关闭。结果上，训练信号来自真实购物结果而非学习型 Reward Model，诊断中还能跟踪有效 group、熵、clip fraction、长度和 Guard 行为。

**追问 1：GRPO 相比 PPO 的关键区别是什么？**

情境上，标准 PPO 常用一个 learned critic 估计 value，但长程工具轨迹训练 critic 也需要显存与稳定目标。我的任务是解释为什么这里选择 GRPO。行动上，我对同一 prompt 生成多条 rollout，用组内 Reward 的相对位置估计 advantage，从而省去独立 value model；策略更新仍保留概率比裁剪等 PPO 风格约束。结果上，系统结构更简单、显存压力更低，但代价是每题必须多次采样，而且组内结果若完全相同就没有有效比较信号，所以项目还需要动态采样和有效 group 监控。

**追问 2：为什么配置里 KL 关闭，不担心策略漂移吗？**

情境上，KL 正则能约束策略偏离参考模型，但过强也可能抑制对新行为的探索；当前项目选择依赖小学习率、LoRA 和 PPO clip 控制更新幅度。我的任务不是声称关闭 KL 一定最优，而是说明当前实验配方。行动上，我会监控 approximate KL、clip fraction、entropy、响应长度和评测性能，并在消融中保持其余变量一致地比较 KL 系数。结果上，现有配置的事实是 `use_kl_in_reward=false`、`use_kl_loss=false`，但是否应开启需要开发集和稳定性证据，不能仅凭常识下结论。

### Q10. 动态采样为什么重要，怎样避免无限重采样？

**第一人称 STAR 口播（模板）**

情境上，GRPO 需要同题 rollout 之间有终局 utility 差异。如果四条全是 1.0、全是负分或完全同分，组内相对优势没有信息；在 SFT 较强或任务过难时，这类常量组会很多。我的任务是提高每次 optimizer update 的有效信息量，又不能无限等待“理想 group”。行动上，我按 uid 分组，只保留 utility 最大值与最小值差超过 1e-8 且无 sampling-invalid 的组；不足时最多补采 3 个 generation batch，连续跳过更新最多 10 次，并把 keep/drop 原因写入诊断。结果上，常量组不会稀释梯度，无效 Reward 不会参与学习，采样成本又有明确上界。

**追问 1：为什么只要组里有一条 invalid 就丢整组？**

情境上，组内 advantage 是相对比较，一条轨迹若因环境错误、Reward 不可验证或硬截断而得到无意义 utility，它不仅污染自身，还会改变其他三条轨迹的相对基线。我的任务是保证组内数值具有同一语义。行动上，我从 runtime 提取 infrastructure_invalid、reward_unverifiable、overlong 和 reward sampling-invalid 原因，任一出现就把整个 uid group 标为 sampling invalid，并保留原因计数用于诊断。结果上，更新只比较可判定的真实策略结果；代价是有效样本减少，所以需要同时监控 invalid rate 并优先修复环境问题，而不是单纯扩大补采次数。

**追问 2：动态采样会不会造成分布偏差？**

情境上，只保留有奖励差异的任务，确实会相对放大“当前策略处于学习边界”的 prompt，并弱化始终成功或始终失败的任务。我的任务是把它视为优化采样策略而非无偏数据抽样。行动上，我会记录被丢弃组中的 all-success、no-success、all-zero 和 invalid 比例，并在固定 validation/Final-200 上评估，而不以训练 batch Reward 代替泛化表现；还可做关闭动态采样的对照。结果上，若最终测试提升且失败桶没有明显恶化，偏置是可接受的效率权衡；否则应调整 group size、温度或课程分布。

## 5. 压力与系统设计问题

### Q11. 评测为什么设计成四个面板，而不是一个总分？

**第一人称 STAR 口播（模板）**

情境上，一个总分会把“买错商品”“需求满足但非 Gold”“搜索策略差”和“服务超时”混在一起，模型排名虽然简单，却无法指导优化。我的任务是让结果既可比较又可诊断。行动上，我把每题拆成四个互不覆盖的面板：Reward 与终局记录严格购买事实；Query Rubric 逐条判断硬软需求；Trajectory Judge 评搜索、候选利用、证据核验、决策和终止；确定性行为与基础设施记录工具、重复、Guard、截断和错误。结果上，Reward 与 Rubric 冲突时保留 disagreement，不让任何一方覆盖另一方，最终再按 task ID 做配对比较而不是合成主观总分。

**追问 1：LLM Judge 如何避免看到答案后倒推？**

情境上，如果 Judge 看到 Reward=1、Gold 商品字段或 Actor 未见候选，它很容易把结果合理化，轨迹评分就失去独立性。我的任务是构造 judge-safe 输入。行动上，我只给 Judge 原始 Query、冻结 Rubric、assistant 文本、工具参数、Actor 当时看到的投影 observation、Guard/step error、稳定 event ID，以及中性的 done/over 和白名单行为指标；明确移除 raw observation、Gold、Reward 分项、strict success 和其他模型结果。结果上，Judge 必须引用真实 event ID 说明证据不足或满足，代码还校验 rubric/event IDs 和 JSON schema，禁止输出综合总分。

**追问 2：Rubric Curator 也是 LLM，会不会乱编要求？**

情境上，让 LLM 自由从目标商品生成评分标准会把用户没有说的属性偷加为要求，造成评测泄漏。我的任务是把模型能力限制在整理而不是创造。行动上，代码先从 Query 与结构化事实生成带 candidate ID、字段、操作符、期望值和原文 span 的候选超集；Curator 只能选择、去重、描述和标 hard/soft，不能新增底层值。返回后代码重新校验 ID、quote、hash 和 schema，并从原候选复制实际操作符。结果上，LLM 提供语言理解，决定权仍被候选空间和确定性校验约束。

### Q12. 严格成功率是怎样计算的？

**第一人称 STAR 口播（模板）**

情境上，很多评测只在成功返回的样本上算比例，会把超时、崩溃和缺失任务排除，导致结果虚高。我的任务是定义一个端到端、固定分母的主指标。行动上，我要求 reward version 为 v3、轨迹 status 为 done、轨迹和 terminal result 的 done/over 均为 true、reward type 与 termination reason 都为 gold_purchase、reward_valid=true 且 purchase_success=true；分母始终是预期任务集合长度 200。结果上，未完成、not judged 或基础设施错误都不会悄悄消失，严格成功代表“完整、可验证地买到 Gold”，比单看 final reward 或 done rate 更难被误读。

**追问 1：为什么 mean reward 的分母看起来和 success rate 不完全相同？**

情境上，代码的严格成功率明确用 expected task 数作分母，而当前 `summary.py` 中 mean final reward 是对实际进入 `by_task` 的轨迹求均值；若存在 missing task，两者口径可能不同。我的任务是发现并说明这种统计边界。行动上，我会同时报告 expected、completed、missing、总 Reward 和均值，并在正式结果中优先保证 200 条完整；如果要强化一致性，可以将缺失任务按预先冻结规则计入零或负效用，但必须版本化协议。结果上，我不会只摘一个 mean reward，而会先检查覆盖率和分母，避免基础设施缺失制造好看的均值。

**追问 2：done rate 高为什么不等于成功率高？**

情境上，环境结束只说明轨迹到达终态，终态可能是错误购买、部分替代、重复循环、最大步数或合规放弃。我的任务是把“执行完成”和“任务完成”拆开。行动上，我分别统计 done、purchase success、gold purchase、reward valid 与 reward type distribution；例如历史 SFT 的 done rate 为 96.5%，strict success 只有 60.5%，剩余包含 partial alternative、wrong purchase、repeat loop、max steps 和 unverifiable。结果上，done rate 主要反映协议和终止能力，strict success 才反映目标购买质量，两者结合才能定位问题。

### Q13. 你怎样解读仓库里的实验结果？

**第一人称 STAR 口播（模板）**

情境上，仓库同时存在当前 Final-200 Clean、旧 Final-200 历史实验和更新后的 SFT 数据，若直接把数字排成一张表会造成错误结论。我的任务是先按数据集哈希和配方分层。行动上，我把 Qwen3.8-27B 的 73.0%/0.6354 标为当前 Clean 上的贡献者复现；把 2B Baseline 0%、SFT 60.5%、GRPO step100 62.0% 标为旧 Final-200 归档；把当前 800/200 SFT 数据与历史 379/49 配方分开。结果上，我只能说历史实验显示 SFT 建立了强 workflow prior、GRPO 有小幅净增，不能把三组数据直接比较，也不能认领任何公开结果为个人指标。

**追问 1：60.5% 到 62.0% 能证明 GRPO 有效吗？**

情境上，1.5 个百分点在 200 题上只对应净增 3 题，单次确定性 rollout 仍可能受任务构成和 checkpoint 选择影响。我的任务是给出不超出证据的结论。行动上，我会看 SFT→GRPO 的 task-level 转移，而不只看边际比例；仓库研究记录提到 12 个 fail→success、9 个 success→fail，exact McNemar p 约 0.664，因此现有样本不足以确认显著提升。结果上，合理表述是“历史结果观察到小幅净增并减少部分 Guard/循环问题”，后续需要多 seed、配对置信区间或更大评测才能证明稳定收益。

**追问 2：Base 0% 是否说明模型完全不会工具调用？**

情境上，严格成功 0% 只说明没有一题满足完整 Gold 契约，并不等于模型没有任何局部能力。历史 Baseline 有 18% done rate、平均 5.875 步和大量 Guard 拒绝，说明它可能会搜索或尝试动作，但工具协议与长程终止不稳定。我的任务是避免用单一指标抹掉过程信息。行动上，我会结合合法工具率、候选打开、证据查看、reward type 和错误 taxonomy 分析。结果上，更准确的结论是 Base 缺乏可靠的端到端 workflow prior，而不是“完全不懂购物”或“语言理解为零”。

### Q14. 这个项目目前有哪些局限？

**第一人称 STAR 口播（模板）**

情境上，一个可信的项目介绍必须主动说明外推边界。我的任务是区分已经验证的工程事实与尚未验证的研究主张。行动上，我总结四类局限：当前 Clean metadata 标记 evaluated=false，除贡献者 27B 外没有同协议的 Base/SFT/GRPO 完整横向结果；历史 GRPO 增益小且统计证据不足；严格成功数据筛选可能降低行为多样性；task-ID 零重叠不能排除语义近重复与同商品污染。此外，Reward v3 依赖结构化匹配，未必覆盖所有合理替代。结果上，我会把这些转成具体实验，而不是用“未来优化”一笔带过。

**追问 1：你会优先做哪个改进？**

情境上，在算力昂贵的情况下，直接再跑 500 step GRPO 很可能无法解释结果变化来自哪里。我的任务是选择信息增益最高的下一步。行动上，我会先做失败图谱和 group 信息量审计：在非盲开发集统计检索不可达、候选忽略、证据不足、规格错误、循环、基础设施无效的互斥分布，同时统计常量 group、invalid group、补采次数和 rollout 行为多样性。结果上，这个低成本诊断能判断瓶颈是环境/检索、Reward 区分度还是 SFT 低熵，再决定改数据、增大 group、调采样温度或做 turn-level credit，而不是盲目堆训练步数。

**追问 2：会不会引入 TRACE 解决长程信用分配？**

情境上，购物 Agent 的终局 Reward 会把整条失败轨迹一起惩罚，即使前面搜索和核验是正确的，turn-level credit 确实有吸引力；仓库也已有 TRACE 相关代码，但 canonical 配置默认关闭。我的任务是避免把实验性能力说成已验证主线。行动上，我会先离线检查冻结参考模型对 canonical 购买目标的 prefix log-prob 是否随有效进展上升，并与终局结果相关；只有 proxy 成立，再做小规模 GRPO 对照。结果上，TRACE 可以作为研究方向，但不能替代当前正式的 outcome-only GRPO，也不能在没有配对实验时宣称有效。

### Q15. 如果让你从零复现，你会怎样保证结果可信？

**第一人称 STAR 口播（模板）**

情境上，这个项目训练一次需要较高显存和数小时，若版本、数据或分母不固定，跑出数字也无法复核。我的任务是建立从配置到结果的证据链。行动上，我会先记录代码 commit、Python/veRL/vLLM/torch 版本、环境 manifest 和四个协议版本；校验 SFT、GRPO、Final-200 的 task ID 与 SHA-256；先跑单元测试和非盲单题，再运行小样本 Baseline/SFT/GRPO；checkpoint 只依据 validation 选择；最终 200 题保存每题轨迹、Reward detail、Guard、投影、Judge request 和 summary。结果上，每个简历指标都能回指配置、日志与任务级产物，别人可以判断它是模型提升还是数据、环境或统计口径变化。

**追问 1：最小可交付复现是什么？**

情境上，我不需要一开始就承诺完整 500 step 和 200 题评测，尤其用户尚未授权执行重任务。我的任务是用最小成本证明链路理解是真实的。行动上，我会先在非 Final-200 开发任务中完成三个层次：单题合法工具链与 Reward 回放；小批量 SFT 数据的 assistant-only mask 可视化和一次短训练；少量 prompt 的四路 rollout 与动态 group keep/drop 诊断。结果上，最小交付应包含可重复命令、环境版本、输入 hash、至少一个真实轨迹、测试结果和失败说明；它不能替代正式指标，但足以支撑“我完成了端到端复现”。

**追问 2：面试官要求现场证明，你展示什么？**

情境上，口头背诵很容易伪装，最有说服力的是从一个任务追到源码和产物。我的任务是准备五分钟可核验演示。行动上，我会打开一条非盲轨迹，指出 user query、每个 assistant tool call、投影后的 observation、Guard 判定、最终 reward type；随后跳到 `acceptance_reasons`、`build_supervised_example`、`select_reward_varying_groups` 和 `_is_strict_success` 四个函数，解释同一条数据在采集、SFT、GRPO 和评测中的变化。结果上，面试官能看到我不仅知道概念，还能定位实现、构造反例并说明指标边界；若某项未运行，我会直接展示待办而不是伪造日志。

## 6. 快问快答

| 问题 | 一句话答案 |
|---|---|
| 严格成功是什么？ | 完整正常终止的 `gold_purchase`，且 `reward_valid=true`、`purchase_success=true`。 |
| Reward Model 是否启用？ | 未启用；主训练信号是确定性的 ShopSimulator Reward v3。 |
| 每个 GRPO prompt 采几条？ | 4 条，训练采样温度 0.7、top-p 0.9。 |
| 为什么常量组被丢弃？ | 组内没有 terminal utility 差异，无法提供相对优势信息。 |
| 最大环境步数？ | 35。 |
| 最大模型上下文？ | 24,576 token；每回合生成预留 512 token。 |
| 搜索/详情/通用 observation 预算？ | 1,536 / 4,096 / 768 token。 |
| SFT 学哪些 token？ | 只学 assistant 文本与工具调用；system/user/tool labels 为 -100。 |
| 当前 SFT 数据？ | 2,498 raw、1,026 accepted、固定 1,000、800/200。 |
| 当前 Final-200 Clean 是否已有本地评测？ | metadata 为 `evaluated=false`；不能声称本地已跑。 |
| 73.0% 是谁的？ | 当前 Clean 上 Qwen3.8-27B 的公开贡献者复现，不是个人结果。 |
| 60.5%→62.0% 是什么？ | 旧 Final-200 上 2B SFT→GRPO step100 的历史归档结果。 |

## 7. 源码证据索引

| Claim | 源码或配置 |
|---|---|
| 工具 schema 与 action 映射 | `src/shopping_grpo/environment/tools.py:21`、`:101` |
| 当前页动作守卫 | `src/shopping_grpo/environment/actions.py:31` |
| 隐藏字段不能进入 observation | `src/shopping_grpo/environment/observation.py:41` |
| 安全 Observation 投影 | `src/shopping_grpo/environment/projection.py:56` |
| 单次只保留一个工具调用 | `src/shopping_grpo/evaluation/rollout.py:550` |
| Reward v3 权重与终局值 | `environments/ShopSimulator/shop_env/web_agent_site/engine/reward.py:32` |
| 购买评分分支 | `environments/ShopSimulator/shop_env/web_agent_site/engine/reward.py:353` |
| 合规放弃资格 | `environments/ShopSimulator/shop_env/web_agent_site/engine/reward.py:462` |
| SFT 验收 | `src/shopping_grpo/collection/sft.py:32` |
| SFT 数据构造与去审计字段 | `src/shopping_grpo/collection/sft.py:83` |
| Assistant-only labels | `src/shopping_grpo/training/sft/dataset.py:56` |
| 按 task-ID 稳定划分 | `src/shopping_grpo/training/sft/dataset.py:203` |
| GRPO 动态 group 过滤 | `src/shopping_grpo/training/grpo/dynamic_sampling.py:231` |
| Reward 运行时校验 | `src/shopping_grpo/training/grpo/adapter/runtime.py:113` |
| 环境 session 租约 | `src/shopping_grpo/training/grpo/adapter/session.py:17` |
| GRPO 主配置 | `configs/grpo.yaml:1` |
| Final-200 盲测保护 | `src/shopping_grpo/evaluation/blind_guard.py:135` |
| Rubric 候选与哈希 | `src/shopping_grpo/evaluation/rubric.py:322` |
| Judge schema 与禁用总分 | `src/shopping_grpo/evaluation/contracts.py:230` |
| 固定分母汇总 | `src/shopping_grpo/evaluation/summary.py:20` |
| 严格成功实现 | `src/shopping_grpo/evaluation/summary.py:212` |
| task-ID 配对比较 | `src/shopping_grpo/evaluation/comparison.py:76` |
| 当前 SFT 数量与哈希 | `data/sft/metadata.json` |
| 当前 Clean 契约与哈希 | `data/evaluation/metadata.json` |
| 历史实验口径 | `experiments/*/run_config.json`、`experiments/*/summary.json` |

## 8. 高风险 Claim 清单

以下说法在没有个人产物前不要使用：

- “我设计并实现了 Shopping GRPO 系统。”——目前只能证明你在研读/复现。
- “我把成功率从 0% 提升到 62%。”——这是旧 Final-200 的仓库历史结果。
- “我在 Final-200 Clean 上做到 73%。”——这是 Qwen3.8-27B 的贡献者复现。
- “我用 800 条数据训练得到 60.5%。”——60.5% 对应旧 379/49 配方，不是当前 800/200。
- “GRPO 显著优于 SFT。”——历史净增仅 1.5pp，现有证据不足以支持显著性。
- “完全没有数据泄漏。”——当前能证明 task-ID 与内容哈希保护，近重复污染仍需审计。
- “Reward 能识别所有合理替代品。”——Reward v3 是结构化、确定性规则，存在覆盖边界。
- “TRACE 已经提升效果。”——canonical 配置默认关闭，当前只是研究/实现方向。
- “我完成了 500 step 训练或 200 题评测。”——完成后必须提供日志、配置、hash 与任务级结果。

## 9. 面试前自测标准

每个主问题练三遍：第一遍照稿，第二遍压缩到 90 秒，第三遍不看稿并接受两个追问。口播时必须能主动说出一个代码位置、一个失败反例和一个指标边界。如果面试官问个人贡献，先说真实身份与已完成动作，再说下一步，不要用“我们”模糊所有权。

