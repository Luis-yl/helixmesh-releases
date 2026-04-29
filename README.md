# HelixMesh

一套自演化 AI agent 协作网络。本仓库只放编译产物，源码不开放。

## 安装（两步）

### 第 1 步：装 helixmesh

任何机器（macOS / Linux），跑一行：

```bash
curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/latest/download/install.sh | bash
```

这会做：

- 下载最新 `helixmesh-bundle.tgz` 解压到 `~/.helixmesh/`
- 软链 `helixmesh` 命令到 `~/.local/bin/`
- 全局装 `evolver` (v1.48.0 MIT fork) + `helixmesh-gep-mcp` 两个二进制

如果 `~/.local/bin` 不在你的 PATH，install.sh 会提示你加一行到 `~/.zshrc` / `~/.bashrc`：

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 第 2 步：在你的项目里 init

```bash
cd <你自己的项目根目录>
helixmesh init --hub <你的 hub URL>
```

例如：

```bash
cd ~/my-awesome-project
helixmesh init --hub http://100.112.9.119:4000
```

`init` 会做：

1. 跟 hub 打 hello，领一个 `node_id` + `node_secret`
2. 在项目目录写 `.env`（hub URL / node 身份）
3. 在家目录写 node 身份文件（`node_id` / `node_secret`）
4. 后台拉起 `evolver --loop`（持续巡逻 + 心跳 + 自动产胶囊）

## 验证装好了

跑一下 `helixmesh status`，全绿就是好的：

```
==============================================================
  HelixMesh Agent 状态
==============================================================
✓ evolver: /usr/local/bin/evolver  v1.48.0
✓ 全局 hooks 已注册：claude-code
✓ 项目 .env：/Users/luis/my-project/.env
    A2A_HUB_URL=http://100.112.9.119:4000
    A2A_NODE_ID=node_xxxxxxxxxxxx
    A2A_NODE_SECRET=***（已配）
✓ 家目录 node_id: node_xxxxxxxxxxxx
✓ 家目录 node_secret: 已存在（mode 600）
✓ hub 在线：http://100.112.9.119:4000
✓ evolver --loop: pid 12345
    last_heartbeat: 2026-04-29T03:11:09.456Z
✓ 当前项目胶囊：N 条（assets/gep/capsules.json）
==============================================================
```

如果显示 `⚠️ evolver --loop 未运行`，跑 `helixmesh restart` 起 loop。

## 常用命令

```bash
helixmesh status    # 健康检查
helixmesh restart   # 重启 evolver --loop
helixmesh tail      # 实时看 agent 日志
helixmesh stop      # 停 evolver --loop
helixmesh health    # 全面诊断（CI/cron 友好，退出码反映状态）
helixmesh inspect   # dump 静态配置（敏感字段打码）
helixmesh help      # 完整帮助
```

也可以装 client 端 hooks（让 agent 跟 Claude Code / Codex / Cursor 集成）：

```bash
helixmesh install --client codex          # 或 claude-code / cursor
```

## 升级

直接重跑 install.sh：

```bash
curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/latest/download/install.sh | bash
```

会覆盖 `~/.helixmesh/` 和全局二进制。已装的项目 `.env` 和家目录 node 身份文件不动。

装特定版本：

```bash
HELIXMESH_VERSION=v0.1.0 bash <(curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/download/v0.1.0/install.sh)
```

## 卸载

```bash
helixmesh uninstall              # 删全局 evolver / hooks 注册
helixmesh uninstall --purge      # 同上 + 删家目录身份文件
rm -rf ~/.helixmesh              # 删 install.sh 装的 bundle
rm ~/.local/bin/helixmesh        # 删命令软链
```

## 系统要求

- **Node.js ≥ 22**（[nodejs.org](https://nodejs.org/zh-cn/download/) 装）
- macOS 或 Linux
- `curl` / `tar`（基本 Unix 工具）

## 常见问题

**Q: 跑 install.sh 报 `Resolving timed out` / `502`？**

GitHub `latest` 别名偶尔抽风。手工指定版本：
```bash
HELIXMESH_VERSION=v0.1.1 bash <(curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/download/v0.1.1/install.sh)
```

**Q: `helixmesh: command not found`？**

`~/.local/bin` 没在 PATH 里。加一行到你的 shell 配置（`~/.zshrc` / `~/.bashrc`）：
```bash
export PATH="$HOME/.local/bin:$PATH"
```

**Q: 全局 npm install -g 失败（Permission denied）？**

你的 npm 全局目录需要 sudo。建议装 [nvm](https://github.com/nvm-sh/nvm) 来管 Node.js，或者：
```bash
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
# 然后重跑 install.sh
```

**Q: `helixmesh init` 报 hub 不可达？**

确认 hub URL 正确（`curl http://your-hub:4000/health`）；如果 hub 在 Tailscale 网络里，确认本机已加入 tailnet。

## License

bundle 内含两种许可证：

| 内容 | License |
|---|---|
| `evolver` (v1.48.0 fork) / `helixmesh-gep-mcp` / `hm-agent.sh` / `install.sh` | MIT |
| 三个 hook .js (`runtime/evolver-hooks/`) | GPL-3.0-or-later（源自 evolver v1.69.8） |

详见 bundle 内 `runtime/evolver-hooks/LICENSE-GPL-3.0.txt`。
