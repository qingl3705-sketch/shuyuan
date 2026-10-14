# 用 AI(MCP) 逆向游戏、建 IDA 库、做 Mod——工具选型指南

> 结论先行：**主流程度排名 = IDA Pro > Ghidra >> Binary Ninja**；
> 但"AI 通过 MCP 操作逆向工具"这件事，**目前最受欢迎的开源桥是 mrexodia/ida-pro-mcp（12.8k★），最推荐的却已经是 Hex-Rays 官方的 ida-mcp**。
> 对你的用途（AI 逆向游戏 → 结果存进 IDA 库 → 做 Mod），推荐路线是 **IDA 9.4+（Home 或 Pro）+ 官方 IDA MCP**，纯白嫖路线是 **Ghidra + GhidraMCP**。

---

## 0. 先分清两层东西，不然会看晕

| 层 | 是什么 | 代表 |
|---|---|---|
| ① 逆向器本体 | 真正做反汇编 / 反编译的软件 | IDA、Ghidra、Binary Ninja、radare2 |
| ② MCP 桥 | 把 ① 的能力暴露给 AI（Claude Code / Codex / Cursor / VS Code / Trae…） | HexRaysSA/ida-mcp、mrexodia/ida-pro-mcp、GhidraMCP |

你说的"这几个 ida"，通常混了两件事：**IDA 的版本档位**（Free / Home / Pro）和**给它接 AI 的 MCP 插件**。下面分开讲。

---

## 1. IDA 各档位对比（官方定价页，2026 年当前）

| | IDA Free | IDA Home | IDA Pro Essential | IDA Pro Expert |
|---|---|---|---|---|
| 价格 | 免费 | **$365/年** | **€1,099/年** | 从 €4,999/年（2 个本地反编译器） |
| 商用 | ❌ | ❌（个人非商业） | ✅ | ✅ |
| 反汇编架构 | 仅 x86-32/64 | 任选 1 个家族：x86 / ARM / MIPS / PPC / RISC-V | 60+ 全支持 | 60+ 全支持 |
| 反编译器 | 云端 x86-32/64 | 云端 2 个（跟着你选的架构） | 云端 2 个 | **本地** 2~6 个 |
| 调试器 | 本地 x86/x64 | 本地 + GDB Server | 本地 + 远程 | 本地 + 远程 |
| 脚本 / SDK | Python + C++ SDK | Python + C++ SDK | 同左 | 同左 |
| **idalib（无头模式）** | ❌ | ✅ | ✅ | ✅ |
| 团队协作 Teams | ❌ | ❌ | 加购 €999/年 | 加购 |

几个关键点：

1. **"$399 买断版 IDA Home"已经没有了**，现在是年订阅（$365/年）。
2. **Home / Free 的反编译是"云"的**——断网就用不了反编译；想要**本地反编译**得从 IDA Pro Expert（€4,999/年）起步。个人玩 Mod，云端反编译通常够用，但要接受"分析内容过一遍 Hex-Rays 的云"这件事。
3. **想做 AI + MCP，必须看 idalib 那一行。** IDA Free 没有 idalib：
   - 官方 IDA MCP 明确要求 **IDA 9.4+ 且带 idalib** → Free 用不了（Home/Pro/Classroom/OEM 可以）。
   - mrexodia/ida-pro-mcp 明确写着 **IDA Free is not supported**。
   - 所以：**想玩 AI 逆向，最低门槛是 IDA Home（$365/年）。**

---

## 2. MCP 桥：谁更火、谁更好用

| 项目 | Star | 定位 | 优点 | 缺点 |
|---|---|---|---|---|
| **[HexRaysSA/ida-mcp](https://github.com/HexRaysSA/ida-mcp)**（官方） | ~1.1k | 官方 MCP Server | **Code Mode** 架构比社区方案省约 20% token；模型无关、可完全本地/内网；随 IDA 同步发布（几乎每周更新）；底层 ida-nexus 支持多 agent 同时开多个 IDB；一条命令装好 | 2026 年才发布，生态刚起步；要求 IDA 9.4+ |
| **[mrexodia/ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp)** | **12.8k** | 社区事实标准 | 客户端支持最广（Claude / Codex / Cursor / VS Code / Kimi / Trae 等 28+）；MIT；加新工具只要写个 `@tool` 函数；支持 headless/SSE/stdio/TCP | **作者本人已在 README 顶部推荐改用官方版**；要求 IDA Pro 8.3+（不支持 Free）；token 消耗略高 |
| **[LaurieWired/GhidraMCP](https://github.com/LaurieWired/GhidraMCP)** | 11.2k | Ghidra 免费路线 | 完全免费；Ghidra 是 NSA 开源、86.8k★ 的行业第二大工具 | 反编译质量普遍被认为弱于 IDA；大型游戏二进制分析体验差；没有 FLIRT/Lumina |
| [bethington/ghidra-mcp](https://github.com/bethington/ghidra-mcp) | 5.6k | Ghidra 免费路线（进阶） | 200+ 工具、懒加载、Docker 部署、Ghidra Server 集成 | 配置比 IDA 路线复杂 |

**一句话选型：**
- **能接受 $365/年 → IDA Home + 官方 ida-mcp（首选）。**
- **不想花钱 → Ghidra + GhidraMCP（能跑，但 AI 分析效果打折）。**
- **手上是 IDA 8.3~9.3 → 用 mrexodia/ida-pro-mcp。**

---

## 3. 游戏逆向：IDA 只是其中一环

只靠 IDA 去"看懂一个游戏"效率很低。真正好用的链条是这样的（按引擎分）：

### Unity（市面上绝大多数手游 / 独立游戏的 Mod 对象）

| 打包方式 | 怎么看代码 | 备注 |
|---|---|---|
| Mono（`Assembly-CSharp.dll`） | **dnSpyEx**（11.3k★）或 ILSpy，**直接就是 C# 源码** | 基本不需要 IDA |
| IL2CPP（`GameAssembly.dll` / `libil2cpp.so` + `global-metadata.dat`） | **Il2CppDumper**（9.5k★）或 **Cpp2IL**（3.0k★） | ← **这才是 IDA 出场的地方** |

**IL2CPP 的关键工作流（正好对应你说的"把逆向存进 IDA 库"）：**

1. `Il2CppDumper` 读 `GameAssembly.dll` + `global-metadata.dat`；
2. 它输出 `script.json`、`il2cpp.h`、以及 **`ida.py` / `ida_with_struct.py`** 脚本；
3. 在 IDA 里打开 `GameAssembly.dll`，`File → Script file` 跑这个脚本 →
   **原本的 `sub_180ABC10` 一键变成 `Player$$TakeDamage`，结构体也一起导入**；
4. 这些函数名 / 注释 / 类型全部落在 `.i64` 数据库文件里（这就是"IDA 库"）；
5. 之后 AI 通过 MCP 读到的就是可读的人类命名，分析效率和准确率天差地别。

> 所以**正确顺序**是：先 Dumper 恢复符号 → 再让 AI 用 MCP 分析 → 结果自动写回 IDB。
> 直接让 AI 去啃一个没有符号的 300MB `GameAssembly.dll`，是浪费 token。

### 运行期改 / 做 Mod 本体
- **BepInEx**（8.8k★）+ Harmony：Unity 打 Mod 的事实标准，写 C# 插件挂钩子。
- **Cheat Engine**（19.3k★）：数值扫描、"什么改写了这个地址"，用来找结构体最快。
- **x64dbg** / **Frida**（22.3k★）：原生调试与 Hook（Android 上 Frida 几乎是必备）。
- IDA 的角色：**搞清"要 Hook 哪个函数、参数是什么、结构体怎么布局"**，然后回到上面这些工具去实现。

### 现实中的拦路虎
- `global-metadata.dat` 经常被加密（AES/XOR 变体），需要先解密再喂给 Dumper；
- 有反调试 / 反篡改（EAC、BattlEye 等），硬碰风险极高；
- 安卓要分清 `arm64-v8a` / `armeabi-v7a`，别拿错 so。

---

## 4. 给你的具体路线（按预算）

**A. 完全免费试水（推荐先走这条）**
1. Ghidra + GhidraMCP，或者 IDA Free（只能 x86/x64、不能接 MCP）；
2. Il2CppDumper + dnSpyEx，先感受一遍"恢复符号 + 看书"的流程；
3. 确认自己真的会长期玩，再考虑掏钱。

**B. 个人主力（性价比最高）**
1. **IDA Home，$365/年**（选 ARM 或 x86 家族，看你的目标游戏是 PC 还是安卓）；
2. 升级到 **IDA 9.4+**，装 **官方 ida-mcp**：`uvx ida-hcli mcp install`；
3. MCP 客户端用 Claude Code / Codex / Cursor / VS Code 都行，官方插件里直接有对应 marketplace；
4. 配 Il2CppDumper 恢复符号 → 让 AI 在 IDB 里改名、加注释、追交叉引用 → 存库 → 按库内容写 BepInEx Mod。

**C. 要商业化（公司、卖 Mod、接单）**
- IDA Pro Essential €1,099/年起（可商用 + 60+ 架构）；需要本地反编译就上 Expert；
- 或者 Ghidra（免费可商用）+ 自建流水线，用官方 MCP 的 headless 模式批量跑。

**⚠️ 两点提醒**
- **IDA Home 明确禁止商业使用**；Mod 本身多是灰色地带，收费 Mod / 绕过线上经济系统风险很大，别碰线上反作弊。
- **AI 的账单独立算**：反编译一个长函数一次就可能吐几千 token。优先用支持"跑代码/写文件"的 **Code Mode** 客户端（官方 ida-mcp 就是为此设计的），不要手动把伪代码整段贴进对话。

---

## 5. 一句话总结

| 问题 | 答案 |
|---|---|
| 哪个最受欢迎？ | **IDA Pro**（商业标准）；开源界是 **Ghidra**（86.8k★）；MCP 桥里 star 最多的是 **mrexodia/ida-pro-mcp**（12.8k★） |
| 哪个更好用？ | **IDA + 官方 Hex-Rays ida-mcp**（IDA 9.4+，$365/年 Home 起步），省 token、更新快、多 agent 协作 |
| 免费怎么办？ | **Ghidra + GhidraMCP**，能干活但效果打折；IDA Free **接不了 MCP** |
| 做 Mod 只靠 IDA 够吗？ | 不够。**Il2CppDumper（恢复符号）→ IDA（理解逻辑）→ BepInEx/Frida（实现修改）** 才是完整链条 |

*数据来源：Hex-Rays 官网定价页与博客、GitHub API（star 数为本文写作时快照）。*
