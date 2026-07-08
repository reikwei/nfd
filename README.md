# nfd

Telegram 反诈机器人项目（基于 Cloudflare Workers）。

## 手机端编辑 Git 仓库

可以通过 **Cursor Cloud Agent**（手机 App 发起）直接编辑本仓库代码。流程如下：

1. 在手机 Cursor 中描述要改什么
2. Agent 在云端克隆仓库、修改代码、运行测试
3. 自动创建分支、提交、推送，并开 Pull Request
4. 在 GitHub 上审查 PR 后合并到 `main`

> 代码不会直接静默写入 `main`，一般会走分支 + PR，便于审查。

### 主要文件

| 文件 | 说明 |
|------|------|
| `worker.js` | Telegram Bot 主逻辑（Cloudflare Workers） |
| `fraud.db` | 反诈数据 |
| `startMessage.md` | 机器人启动说明 |
| `shuoming.txt` / `caidanshuoming.txt` | 功能说明文档 |
