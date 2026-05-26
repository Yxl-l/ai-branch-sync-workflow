# 🔄 AI Branch Sync Workflow

> 面向多分支、多平台项目的 AI 辅助代码同步流程。
>
> 适用于以下场景：
>
> * `develop-xx` 分支包含新的业务改动，需要同步。
> * `fusion-xx` 分支需要接收改动，但存在平台差异或特有代码。
> * 两个分支整体结构相似，但因平台或部署环境存在差异。

---

# 📌 核心目标

本流程旨在：

* 仅同步“业务逻辑改动”
* 保留目标分支的平台特有代码
* 降低 AI 自动覆盖风险
* 通过逐文件、逐方法校验提高同步可靠性
* 建立可审计、可回溯的 AI 同步流程

---

# 🧠 推荐使用模型

建议使用支持长上下文与代码推理能力较强的大模型，例如：

* GPT-5.x
* Claude Sonnet / Opus
* Gemini Pro

---

# 📂 推荐目录结构

```text
.
├── README.md
├── prompt-sync-file.md
├── prompt-audit-method.md
└── examples/
    ├── sample-diff.patch
    └── sample-analysis.md
```

---

# ⚠️ 同步原则（非常重要）

## 必须遵守

1. 保留 fusion 分支中的平台专属逻辑
2. 不允许直接覆盖整个文件
3. 不同步历史遗留差异
4. 仅同步指定时间范围内的业务改动
5. 必须逐文件处理
6. 修改后必须编译 + 测试
7. 核心方法必须进行二次逻辑审计

---

# 🛡️ Safe File Sync Prompt

建议搭配 `prompt-sync-file.md` 使用。

该 Prompt 用于：

* 对比 develop 与 fusion 同名文件
* 自动过滤平台代码
* 输出安全 patch
* 生成变更分析与风险提示

适用于：

* `OrderService.java`
* `PaymentValidator.kt`
* `XXXServiceImpl.java`
* 核心业务服务类

---

# 🚀 标准执行流程

---

# 第一步：锁定范围，整理差异清单

## 🎯 目标

基于时间范围筛选 develop 分支改动。

避免：

* 历史遗留差异被误同步
* AI 将旧逻辑误判为新改动

---

## ✅ 操作步骤

1. 切换到源分支（如 `develop-xx`）
2. 确定需要同步的时间范围
3. 让 AI 生成去重后的改动文件清单

---

## 💬 Prompt

```text
请总结 develop-xx 分支从 2026.4.22 15:00 到 2026.5.6 16:14 之间的所有提交。
请列出该范围内所有改动过的文件（去重），并提供一份包含“文件名”和“简单概述修改内容”的清单。
```

---

# 第二步：明确规则，分析目标差异

## 🎯 目标

让 AI 理解 fusion 分支中的“不可修改区域”。

建立同步边界。

---

## ✅ 操作步骤

1. 切换到目标分支（如 `fusion-xx`）
2. 将第一步生成的文件清单提供给 AI
3. 明确说明 fusion 分支的特殊逻辑
4. 要求 AI 后续逐文件同步

---

## 💬 Prompt

```text
这是 develop-xx 的改动清单。我现在需要把这些改动同步到当前的 fusion-xx 分支。
请先整体分析一遍差异，随后我将一个文件一个文件地让你执行同步。

核心规则：
1. 目标分支 fusion-xx 有独有的代码逻辑，同步时必须保留，严禁覆盖。
2. 不要同步 develop-xx 中非修改业务的差异。
3. 请按我的指令逐个文件处理。
```

---

# 第三步：逐文件智能同步

## 🎯 目标

精细化合并业务逻辑。

确保：

* develop 新业务逻辑被迁移
* fusion 平台逻辑被保留

---

## ✅ 操作步骤

1. 从清单中取出一个文件
2. 提供 develop 与 fusion 两个版本
3. 让 AI 输出安全 patch

---

## 💬 Prompt

```text
请同步文件：ReportOfPassRateServiceImpl.java

执行要求：
1. 对比 develop-xx 和 fusion-xx 分支中该文件的差异。
2. 识别出 develop-xx 在指定范围内的业务改动。
3. 将这些改动合并到 fusion-xx 的当前文件中，务必保留 fusion-xx 原有的差异代码。
```

---

# 第四步：修复测试文件

## 🎯 目标

修复因业务签名变化导致的测试失败。

---

## 常见问题

* 方法参数类型变化
* 新依赖未 Mock
* Mock 返回值缺失
* 构造函数变化
* Bean 初始化失败

---

## 💬 Prompt

```text
请修复所有测试文件中对应的调用错误。

常见修复点：
- 方法参数类型变更（例如：List<Long> 变为 Long）。
- 新增的依赖类需要添加 @Mock 注解。
- 新调用的方法需要补充 Mock 返回值。
- 构造函数参数变更导致的实例化错误。
```

---

# 第五步：编译与测试验证

## 🎯 目标

建立快速反馈循环。

及时发现：

* 漏同步
* Mock 缺失
* import 错误
* Bean 依赖错误
* API 签名不一致

---

## ✅ 操作建议

每同步 1~3 个文件：

```bash
mvn test
```

或者：

```bash
gradle test
```

出现错误时：

直接将编译日志投喂给 AI 修复。

---

# 第六步：逐接口 / 方法逻辑核对

## 🎯 目标

防止 AI：

* 漏掉某段逻辑
* 错误合并 if/else
* 修改边界条件
* 引入隐藏回归

这是最重要的一步。

---

## ✅ 建议核查内容

优先检查：

* ServiceImpl
* Validator
* Rule Engine
* Strategy
* 核心聚合逻辑
* 复杂条件分支

---

## 💬 Prompt

```text
请对比 develop-xx 和 fusion-xx 分支中 XXXServiceImpl 类的 methodName 方法。
请逐行 Diff，确认两个分支的逻辑是否完全一致（排除平台差异代码后）。
```

---

# 第七步：配置一致性验证

## 🎯 目标

确保配置、枚举、常量没有同步错乱。

避免：

* 枚举值错位
* 常量含义不一致
* feature flag 失效
* 配置遗漏

---

## 💬 Prompt

```text
请确认修改过的配置类、常量定义、枚举值等在两个分支中的定义和用法是否保持一致。
```

---

# 🔒 Safe Sync Rules（推荐固定给 AI）

```text
1. NEVER modify platform-specific code.
2. NEVER overwrite fusion-only logic.
3. NEVER sync unrelated formatting changes.
4. NEVER change public APIs unless explicitly required.
5. Only output patch + rationale.
6. If uncertain, output:
⚠️ MANUAL REVIEW REQUIRED
```

---

# 📋 推荐 AI 输出格式

建议 AI 始终输出：

## 1. Change Analysis

| Method         | Changed | Risk | Notes             |
| -------------- | ------- | ---- | ----------------- |
| calculateTotal | Yes     | Low  | Tax logic updated |

---

## 2. Proposed Patch

统一 diff patch。

---

## 3. Rationale

解释为什么安全。

---

## 4. Risk Flags

列出：

* 平台边界
* API 修改
* 配置修改
* 不确定逻辑

---

# 🧪 推荐验证顺序

```text
1. Compile
2. Unit Test
3. Integration Test
4. Core Service Audit
5. Config Validation
6. Smoke Test
```

---

# 📌 关键注意事项

## 1. 不要一次同步太多文件

推荐：

* 每次 1~3 个文件
* 每次不超过 300 行核心逻辑

否则 AI 极易遗漏。

---

## 2. 平台代码必须提前标记

推荐添加：

```java
// PLATFORM: ANDROID
```

或：

```java
@PlatformSpecific
```

否则 AI 无法稳定识别。

---

## 3. AI 不可信，必须人工 Review

AI 的职责：

* 加速 diff
* 识别业务改动
* 生成 patch

人的职责：

* 判断业务意图
* 确认平台边界
* 审核关键逻辑

---

## 4. Low Confidence 禁止直接合并

如果 AI 输出：

```text
Confidence: Low
```

必须：

* 重新分析
* 缩小同步范围
* 人工 review

---

# 🧩 推荐搭配使用

| 文件                       | 用途      |
| ------------------------ | ------- |
| `prompt-sync-file.md`    | 安全同步单文件 |
| `prompt-audit-method.md` | 核心方法审计  |
| `README.md`              | 流程说明    |

---

# ✅ 最终目标

通过本流程，实现：

* AI 辅助安全同步
* 保留平台差异
* 降低人工 diff 成本
* 提升多分支维护效率
* 减少回归风险

---

# 🏗️ Architecture

```text
develop branch
    ↓
diff extraction
    ↓
AI semantic analysis
    ↓
platform filtering
    ↓
safe patch generation
    ↓
human review
    ↓
compile & test
    ↓
audit
    ↓
merge
```

---

# ❓ Why Not Direct Merge?

很多团队会问：

```text
为什么不直接 git merge / cherry-pick？
```

原因是：

在多平台、多部署、多定制代码库中：

* fusion 分支通常包含平台专属逻辑
* develop 分支可能存在大量历史遗留差异
* 直接 merge 极易覆盖平台代码
* cherry-pick 无法理解“业务逻辑”和“平台逻辑”的区别
* AI 需要的是“语义级同步”，而不是纯文本 merge

---

## ✅ 适用场景

* Android / iOS 多平台代码
* SaaS / 私有化部署
* 国内版 / 海外版
* OEM 定制版本
* 长生命周期维护分支
* 多租户业务版本

---

## ❌ 不适用场景

以下情况建议直接使用：

* git merge
* rebase
* cherry-pick

而不是本 workflow：

* 普通 feature branch
* 短生命周期开发分支
* 无平台差异项目
* 无定制逻辑项目

---

# 🧪 Example Workflow

## 场景示例

### develop 分支

```java
public BigDecimal calculateTotal(Order order) {
    return order.getAmount().multiply(new BigDecimal("1.13"));
}
```

新增：

* 税费计算逻辑

---

### fusion 分支

```java
public BigDecimal calculateTotal(Order order) {
    // PLATFORM: ANDROID
    if (isAndroidChannel(order)) {
        return androidCalculator.calculate(order);
    }

    return order.getAmount();
}
```

存在：

* Android 平台特殊逻辑

---

### AI 输出结果

```diff
 public BigDecimal calculateTotal(Order order) {
     // PLATFORM: ANDROID
     if (isAndroidChannel(order)) {
         return androidCalculator.calculate(order);
     }
 
-    return order.getAmount();
+    return order.getAmount().multiply(new BigDecimal("1.13"));
 }
```

---

## ✅ 最终效果

* 保留 Android 平台逻辑
* 同步 develop 的税费计算
* 不覆盖 fusion 特殊代码

---

# ❌ Common AI Failure Cases

## Case 1：覆盖平台代码

### 错误行为

AI 直接使用 develop 文件覆盖 fusion 文件。

### 风险

* 平台逻辑丢失
* 线上行为异常
* 编译失败

### 解决方案

* 提前标记平台代码
* 强制 AI 只输出 patch
* 禁止 whole-file overwrite

---

## Case 2：误同步历史遗留差异

### 错误行为

AI 将旧 diff 识别为本次业务修改。

### 风险

* 引入脏代码
* 产生未知行为变化

### 解决方案

* 必须限定时间范围
* 必须先生成 commit diff 清单

---

## Case 3：误改 Public API

### 错误行为

AI 修改：

* 方法签名
* DTO 字段
* 接口返回值

### 风险

* 编译失败
* 调用方崩溃

### 解决方案

固定规则：

```text
NEVER change public APIs.
```

---

## Case 4：删除 Import

### 错误行为

AI 自动整理 import。

### 风险

* 编译失败
* 注解丢失

### 解决方案

要求：

```text
Preserve import order.
```

---

## Case 5：错误合并条件逻辑

### 错误行为

AI 修改：

* if/else
* switch
* feature flag
* fallback 逻辑

### 风险

* 隐性业务回归
* 条件边界错误

### 解决方案

必须执行：

* method audit
* line-by-line diff
* 核心逻辑 review

---

# 📑 Standard Output Template

建议 AI 固定输出以下结构：

```markdown
## 1. Change Analysis

## 2. Proposed Patch

## 3. Rationale & Confidence

## 4. Risk Flags
```

这样可以：

* 稳定输出结构
* 方便 review
* 方便审计
* 方便后续自动化

---

# 👨‍⚖️ Human-in-the-Loop Principle

## 核心原则

```text
AI only proposes patches.
Human reviewers make the final merge decision.
```

AI 的职责：

* 提取业务 diff
* 识别风险
* 生成 patch
* 提供审计建议

人的职责：

* 判断业务意图
* 审核平台边界
* 最终 merge 决策
* 线上风险控制

---

# 📂 Recommended Examples Structure

```text
examples/
 ├── before/
 ├── after/
 ├── prompts/
 └── patches/
```

建议包含：

* develop 原始文件
* fusion 原始文件
* AI prompt
* AI patch
* 最终结果

---

# 📂 Recommended Templates Structure

```text
templates/
 ├── sync-analysis.md
 ├── patch-output.md
 └── audit-output.md
```

---

# 📄 License

MIT
