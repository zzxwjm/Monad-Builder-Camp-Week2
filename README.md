# Moss 新手教程：从零资金运行一次可验证的 Web3 Agent 流程

本教程适合第一次接触 Moss、AI Agent 和 Web3 协议交互的读者。目标不是立刻让你执行真实交易，而是帮助你在**没有私钥、没有资金、不签名、不发送交易**的前提下，理解 Moss 如何发现协议能力、生成交易计划并进行模拟验证。

## 1. 先理解你要运行的东西

Moss 可以看作 AI Agent 与 Web3 协议之间的结构化交互层：

```text
用户意图 → Agent 澄清条件 → Moss 工具 → 协议适配器 → Plan → 模拟器 → 用户/钱包审查
```

Moss 不是聊天机器人，也不是钱包。Agent 负责理解“我想做什么”，Moss 负责把已经明确的目标转换为协议调用计划并检查模拟结果，钱包只有在用户主动确认后才可能参与签名。

本教程围绕四个阶段展开：

```text
discover → load → action → simulate
```

- `discover`：查找当前可用的能力。
- `load`：读取某个能力的参数、约束和风险说明。
- `action`：生成包含未签名交易和预期效果的 Plan。
- `simulate`：在签名之前重放并核对实际效果。

## 2. 准备本地环境

根据 Moss 当前 Getting Started 文档，需要 Node.js 22 或更高版本，以及 pnpm。

```bash
git clone https://github.com/nishuzumi/moss
cd moss
pnpm install
pnpm build
```

先运行离线测试，确认本地工具链没有问题：

```bash
MOSS_SKIP_E2E=1 pnpm test
```

这个环境变量会跳过依赖实时链上 RPC 的端到端测试。之后如果要运行 live mainnet e2e，需要一个支持 `debug_traceCall` 的 RPC。第一次学习建议先完成离线测试和下面的零资金示例。

## 3. 第一次运行：wrap 示例

在仓库根目录执行：

```bash
pnpm --filter @themoss/example-simple-flow wrap
```

这个示例会构建一个把 1.5 MON 包装成 WMON 的未签名计划，并在 Monad 链上进行模拟。它不会要求你提供私钥，也不会替你发送交易。

你应该重点观察四件事：

1. Moss 先发现哪个协议支持 `wrap`。
2. Moss 加载该能力的调用说明。
3. `action` 生成 Plan，而不是生成已经广播的交易。
4. `simulate` 对照 Plan 声明的预期效果和模拟观察到的效果。

如果最后看到：

```text
✓ No warnings — the unsigned txs may be handed to a wallet for review.
```

它的含义是：在这次模拟中，没有发现预期与观察结果之间的差异。它**不等于**未来签名后一定成功，因为价格、流动性、链上状态和合约状态都可能变化。

## 4. 手动拆开四个阶段

你可以打开 `examples/simple-flow/src/play.ts`，或创建一个临时 TypeScript 文件，先注册运行时和协议清单：

```ts
import { Registry } from "@themoss/core";
import { ercManifest } from "@themoss/erc";
import { kuruManifest } from "@themoss/protocol-kuru";
import { monadRuntime, systemManifest } from "@themoss/system";

const runtime = monadRuntime();
const registry = new Registry(runtime);
for (const manifest of [systemManifest, ercManifest, kuruManifest]) {
  registry.use(manifest);
}
```

### 4.1 discover：查能力目录

```ts
console.log(registry.discover({ verb: "swap" }));
```

这里的 `verb` 是用户视角的动作，例如 `swap`、`wrap`、`supply`、`transfer`。它不一定等于协议合约里的函数名：例如 WMON 合约可能使用 `deposit()`，但对用户来说它表达的是 `wrap`。

### 4.2 load：读取调用合同

```ts
console.log(registry.load([{ protocol: "kuru", method: "swap" }]));
```

返回内容会说明参数语义、风险标签和调用方式。例如金额可能要求人类可读的字符串 `"1.5"`，运行时再根据代币精度进行缩放。这里的 `1.5` 是示例输入，不是 Moss 擅自决定要花费 1.5 个代币；真实使用时应由用户明确提供并由 Agent 继续确认。

### 4.3 action：生成 Plan

```ts
const account = "0xCcCccCCCcCCcccCcCccccCcCCCCcccccCcCCcCcC";
const plan = await registry.action("kuru", "swap", account, {
  tokenIn: "MON",
  tokenOut: "USDC",
  amount: "1",
});

console.log(plan.intent, plan.expects, plan.txs);
```

Plan 通常包含：

- 编码好的未签名交易 `txs`；
- 这次操作的意图和风险标签；
- 预期的代币流入、流出和授权范围 `expects`；
- 用于完整性校验的 `planHash`。

它不是签名交易，也不会自动广播。账户字符串只是构建计划时使用的地址值，不代表你需要把私钥交给 Moss。

### 4.4 simulate：签名前验证

```ts
import { createTraceSimulator } from "@themoss/simulator";

const simulator = createTraceSimulator(runtime);
const { results } = await simulator.simulate([plan]);
console.log(results[0]?.effects, results[0]?.warnings);
```

模拟器会使用 RPC 的 `debug_traceCall` 重放交易，并检查实际资产流、授权和收款人是否符合 Plan 的声明。只要出现未声明的差异，就应该把 warning 当作停止信号，先查清原因，而不是直接交给签名者。

## 5. 从示例走向 Agent

Moss 的四步能力也可以通过 MCP Server 暴露给外部 Agent。一个最小的 MCP 配置形态如下：

```json
{
  "mcpServers": {
    "moss": {
      "command": "node",
      "args": ["<path-to-moss>/packages/mcp-server/dist/cli.js"],
      "env": { "MOSS_RPC_URL": "https://rpc.monad.xyz" }
    }
  }
}
```

然后，Agent 可以先询问能力和参数，再生成 Plan，最后请求模拟。注意：当用户只说“帮我换币”时，Agent 仍应追问输入资产、目标资产、数量、滑点和网络；Moss 不会替用户猜测这些关键条件。

## 6. 常见问题

### 必须有资金或私钥吗？

完成本教程的安装、离线测试、Plan 构建和模拟不需要私钥或真实资金。任何需要签名的步骤都应由用户自己的钱包在充分审查后完成。

### 为什么模拟失败？

常见原因包括 RPC 不支持 `debug_traceCall`、网络连接问题或执行条件不满足。先阅读错误信息并更换符合文档要求的 RPC，不要为了“让流程通过”而跳过模拟。

### `No warnings` 是安全保证吗？

不是。它只说明当前模拟中没有发现预期差异。签名前后链上状态可能改变，仍需要用户审查 Plan、风险标签和最终交易。

### `wrap`、`swap`、`supply` 有什么区别？

- `wrap`：把原生资产转换为对应的包装代币。
- `swap`：把一种资产兑换成另一种资产。
- `supply`：把资产存入某个借贷或收益协议。具体规则取决于接入的协议适配器。

## 7. 如何参与 Moss 开源贡献？

第一次贡献不一定要写新的协议适配器。你可以按下面的顺序开始：

1. 阅读 [README](https://github.com/nishuzumi/moss) 和 [Docs](https://github.com/nishuzumi/moss/tree/main/docs)。
2. 用离线测试和零资金示例核对文档描述。
3. 在 Issues 中寻找文档、FAQ、示例或开发者体验问题。
4. Fork 仓库，创建 `codex/` 前缀的分支，做一个小而聚焦的修改。
5. 提交 PR，说明动机、改动范围、验证方法和未解决的限制。
6. 根据维护者 review 修改，不把“PR 已提交”写成“贡献已合并”。

我本周围绕 [Issue #16](https://github.com/nishuzumi/moss/issues/16) 补充了 Getting Started 的新手 FAQ，并提交了 [PR #26](https://github.com/nishuzumi/moss/pull/26)。这次经历说明，文档贡献同样需要核对分支、实际 diff、链接、测试证据和 review 状态。

## 8. 下一步阅读

- [Getting Started](https://github.com/nishuzumi/moss/blob/main/docs/getting-started.md)
- [MCP Tools Reference](https://github.com/nishuzumi/moss/blob/main/docs/mcp-tools.md)
- [Agent Skill Guide](https://github.com/nishuzumi/moss/blob/main/docs/agent-skill.md)
- [Protocol Onboarding](https://github.com/nishuzumi/moss/blob/main/docs/protocol-onboarding.md)
- [Security Model](https://github.com/nishuzumi/moss/blob/main/SECURITY.md)
- [Contributing Guide](https://github.com/nishuzumi/moss/blob/main/CONTRIBUTING.md)

这份教程的核心原则是：先理解能力，再生成计划；先模拟验证，再考虑签名；先做小范围文档贡献，再逐步接触协议适配器和 Agent 集成。
