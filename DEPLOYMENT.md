# 部署文档 · 从零部署到 Cloudflare Workers

本项目基于 **Cloudflare Workers + Durable Objects**，全程跑在 Cloudflare 免费计划上，无需服务器、无需数据库。

> 完整功能介绍见 [README.md](./README.md)，日常使用方法见 [USAGE.md](./USAGE.md)。

---

## 前置要求

- 一个 Cloudflare 账号（免费计划即可，Durable Objects 的 SQLite 后端在免费计划可用）
- Node.js 18 及以上
- 已安装 npm

## 第一步：获取代码

```bash
git clone https://github.com/lovexw/wechat-chat-room.git
cd wechat-chat-room
npm install
```

## 第二步：本地登录 Cloudflare

```bash
npx wrangler login
```

执行后会打开浏览器完成授权。可以用以下命令确认登录状态：

```bash
npx wrangler whoami
```

## 第三步：本地试运行（可选但推荐）

```bash
npm run dev     # 打开 http://localhost:8787
```

本地开发需要管理员功能时，在项目根目录创建 `.dev.vars`（该文件已被 `.gitignore` 忽略，不会提交）：

```
ADMIN_USERNAME=你的账号
ADMIN_PASSWORD=你的密码
ADMIN_SECRET=任意随机长字符串
```

生成随机 `ADMIN_SECRET` 的方法：

```bash
openssl rand -hex 32
```

## 第四步：部署上线

```bash
npm run deploy
```

部署成功后终端会输出访问地址，形如：

```
https://wechat-chat-room.<你的子域>.workers.dev
```

首次部署 Durable Objects 会自动执行 `wrangler.jsonc` 中的 migration（创建 SQLite 后端的 `ChatRoom` 类），无需手动操作。

## 第五步：配置管理员账号（必需）

线上环境的管理员账号密码通过 **Cloudflare Secrets** 注入，仓库中不保留任何明文口令。**必须完成本步，否则管理员登录会一直失败（这是预期行为）。**

```bash
npx wrangler secret put ADMIN_USERNAME   # 按提示输入管理员账号，回车确认
npx wrangler secret put ADMIN_PASSWORD   # 按提示输入管理员密码，回车确认
npx wrangler secret put ADMIN_SECRET     # 建议设置：令牌签名密钥，用 openssl rand -hex 32 生成
```

- 修改 Secret 后立即生效，无需重新部署
- 如怀疑密码泄露，重新执行 `npx wrangler secret put ADMIN_PASSWORD` 即可轮换；轮换后已签发的管理员令牌立即失效，需重新登录
- 在 Cloudflare 控制台查看/管理 Secret：Workers & Pages → wechat-chat-room → Settings → Variables and Secrets

## 第六步：验证部署

1. 打开 `https://wechat-chat-room.<你的子域>.workers.dev`
2. 页面顶部应显示「已连接」和在线人数
3. 发一条消息，确认能正常显示
4. 另开一个无痕窗口，点右上角「管理员登录」，输入你配置的账号密码
5. 登录成功后测试「关闭群聊」开关，确认普通窗口出现 🔒 提示且无法发言
6. 重新打开开关，恢复正常聊天

## 常见部署问题

**部署报错 `Durable Objects ... need a paid plan?`**
确认 `wrangler.jsonc` 中 `migrations` 使用 `new_sqlite_classes`（本项目已配置）。SQLite 后端的 Durable Objects 在免费计划可用；如仍报错，检查账号是否登录了正确的 Cloudflare 账号（`npx wrangler whoami`）。

**部署成功但打开页面 404**
确认 `wrangler.jsonc` 中 `assets.directory` 指向 `./public`，且 `public/` 目录下有 `index.html`。

**管理员登录一直提示「账号或密码错误」**
说明 Secrets 未配置或配置的和输入的不一致。用 `npx wrangler secret list` 检查已配置的 Secret 名称，重新执行第五步。

**自定义域名（可选）**
在 Cloudflare 控制台：Workers & Pages → wechat-chat-room → Settings → Domains & Routes → Add → Custom domain，绑定一个已在同一账号托管的域名即可，证书自动签发。

**中国大陆访问**
`*.workers.dev` 域名在部分地区可能不稳定，绑定自定义域名通常可以改善访问体验。
