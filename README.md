# dsh-manager

DeepSeek Harness 的能力清单仓库。**只放清单，不放代码。**

每台机器都可以从这里读走一份清单，然后在自己的面板里逐项选择要装的东西；
反过来，任何一台机器都可以把自己现在的清单发布到这里。

## 里面有什么

| 文件 | 作用 |
| --- | --- |
| [`dsh-manifest.json`](dsh-manifest.json) | 清单本体：插件 / MCP / 技能三类，每项带平台与外部依赖 |

清单不含任何密钥、绝对路径或机器名。发布时由 `dsh-inventory` 脱敏，
面板会先给出脱敏报告再提交。

## 清单格式

```jsonc
{
  "manifestVersion": 1,          // 只有 1，读到别的版本会直接报错而不是猜
  "generatedAt": 1793445583956,  // 生成时间戳，仅供人看
  "repo": "owner/repo",          // 发布方自报家门
  "ref": "main",                 // 生成时的分支
  "plugins": [
    {
      "kind": "plugin",
      "name": "dsh-git-bash-tool",
      "version": "0.1.0",
      "source": "repo",          // repo | npm | git | unknown
      "platform": "win32",       // any | win32 | linux | darwin
      "requires": ["officecli"]  // 可选：本机必须有的外部命令，或 "gpu"
    }
  ],
  "mcp":   [{ "kind": "mcp",   "name": "context7", "transport": "stdio", "platform": "any" }],
  "skills":[{ "kind": "skill", "name": "ponytail", "platform": "any" }]
}
```

### 字段说明

- **`platform`** — 这一项在哪些系统上能用。`win32` 的项在 Linux 机器上会被标成
  `blocked`，不会被当成"缺失"，也不会出现在可勾选列表里。
- **`requires`** — 外部前置条件。写命令名（`officecli`）表示本机 `PATH` 里要有它；
  写 `gpu` 表示要有显卡。缺任何一个同样归为 `blocked`。
- **`source`** — 告诉拉取方从哪里取：`repo` 指本仓库（`DSH-plugins`）里的包，
  `npm` 指 npm registry，`git` 指任意 git 地址。`unknown` 表示发布方也不知道，
  拉取时会失败并说明原因。

## 拉取方看到的差异

| 状态 | 含义 | 能勾选吗 |
| --- | --- | --- |
| `missing` | 清单有，本机没有 | 是 |
| `outdated` | 都有，版本不同 | 是 |
| `blocked` | 本机不适用（平台不符 / 缺依赖 / 没 GPU） | 否，只显示原因 |
| `disabled` | 装了但被关掉，或已停用的桩 | 否 |
| `current` | 一致 | 否 |
| `local-only` | 本机有，清单没有（可以发布上去） | 否 |

**没有"全量同步"。** 每一项都要人点头。

## 怎么用

在 DSH 面板 → 工具管理 → 同步：

1. 填清单仓库地址（`owner/repo` 或完整 URL）和 token。token 只用来**写**这个仓库，
   存在 `$DSH_HOME/plugin-sync.json`，不回显、不发布。
2. 点"读取清单"，看差异列表。
3. 勾要装的，点"安装选中"。
4. 本机有想分享的东西，点"发布本机清单"。

## 边界

- 这个仓库**不存插件代码**。插件代码在 [`DSH-plugins`](https://github.com/kurumi1ksllq/DSH-plugins)。
- 清单是**意图**，不是事实。机器实际装了什么由各自的 `dsh-inventory` 现场扫描得出。
- 拉取方不会因为你删了某一项就卸载它。清单不是白名单。
