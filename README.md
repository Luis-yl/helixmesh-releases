# HelixMesh

一套自演化 AI agent 协作网络。本仓库只放编译产物，源码不开放。

## 安装（两步）

### 第 1 步：装 helixmesh

任何机器（macOS / Linux），跑一行：

```bash
curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/latest/download/install.sh | bash
```

安装会做：

- 下载最新 `helixmesh-bundle.tgz` 解压到 `~/.helixmesh/`
- 软链 `helixmesh` 命令到 `~/.local/bin/`
- 装好运行时引擎和 MCP 服务器（命令行可用）

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
4. 后台拉起 helixmesh agent loop（持续巡逻 + 心跳 + 自动产胶囊）

## 验证装好了

跑一下 `helixmesh status`，全绿就是好的：

```
==============================================================
  HelixMesh Agent 状态
==============================================================
✓ helixmesh agent (v1.48.0)
✓ 全局 hooks 已注册：claude-code
✓ 项目 .env：/Users/luis/my-project/.env
    A2A_HUB_URL=http://100.112.9.119:4000
    A2A_NODE_ID=node_xxxxxxxxxxxx
    A2A_NODE_SECRET=***（已配）
✓ 家目录 node_id: node_xxxxxxxxxxxx
✓ 家目录 node_secret: 已存在（mode 600）
✓ hub 在线：http://100.112.9.119:4000
✓ agent loop: pid 12345
    last_heartbeat: 2026-04-29T03:11:09.456Z
✓ 当前项目胶囊：N 条（assets/gep/capsules.json）
==============================================================
```

如果显示 `⚠️ agent loop 未运行`，跑 `helixmesh restart` 起 loop。

## 常用命令

```bash
helixmesh status    # 健康检查
helixmesh restart   # 重启 agent loop
helixmesh tail      # 实时看 agent 日志
helixmesh stop      # 停 agent loop
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
HELIXMESH_VERSION=v0.1.1 bash <(curl -fsSL https://github.com/Luis-yl/helixmesh-releases/releases/download/v0.1.1/install.sh)
```

## 卸载

```bash
helixmesh uninstall              # 删全局二进制 + hooks 注册
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

bundle 内主体（CLI / 引擎 / MCP / 脚本）MIT 许可。其中三个集成 hook 脚本为 GPL-3.0-or-later，bundle 内附 LICENSE 文件说明。
