# NodeSeek 自动签到 · 工作流入口

这个仓库**只放 GitHub Actions 工作流**，不放代码、不放 Cookie。

真正的代码在私有库 **`nodeseek-auto`**（那里还有加密后的 Cookie 状态文件）。
为什么这么分：公开库的 Actions **免费、不限分钟**；而代码和 Cookie 不能公开。

```
account-1.yml   → 账号1（每天自动跑）
account-2.yml   → 账号2（填好 Secret 后解除注释即可定时）
account-3.yml   → 账号3（同上）
```

## 它每天做什么

1. 用**部署密钥**把私有库 clone 下来
2. `curl_cffi` 先试（快、省资源），被 Cloudflare 拦就自动升级到 `xvfb` 里的真实 Chrome
3. 先看今天签没签，没签才签 —— **一天最多一次**，绝不反复请求
4. 把刷新后的 Cookie 加密写回**私库**的状态文件
5. 失败/掉线时通过你配的渠道推一条消息

三个工作流共用同一个 `concurrency` 组，**永远串行**，不会并发打站点。

## 需要配置的 Secrets

`Settings → Secrets and variables → Actions`：

| Secret | 必填 | 说明 |
|---|---|---|
| `DEPLOY_KEY` | ✅ | 私库的部署密钥**私钥**（私库 → Settings → Deploy keys，勾 Allow write access） |
| `CODE_REPO` | 可选 | 私库全名，默认 `YYluck06/nodeseek-auto` |
| `NS_COOKIE_1` | ✅ | 账号1 Cookie：`session=xxx; cf_clearance=yyy` |
| `NS_PROXY_1` | 建议 | 账号1 固定代理，如 `http://user:pass@1.2.3.4:8080` |
| `NS_COOKIE_2` / `NS_PROXY_2` | 多账号 | 账号2 |
| `NS_COOKIE_3` / `NS_PROXY_3` | 多账号 | 账号3 |
| `NS_STATE_KEY` | ✅ | 状态文件加密口令（设了别改；改了旧 Cookie 状态就解不开） |
| `NS_USER_1` / `NS_PASS_1` | 建议 | 账号1 用户名/密码：Cookie 失效时脚本会自己登录续期（验证码发到 Telegram） |
| `TG_BOT_TOKEN` / `TG_CHAT_ID` | 建议 | Telegram Bot：推送通知 + 接收登录验证码 |
| `NS_COOKIE_2` / `NS_PROXY_2` / `NS_USER_2` / `NS_PASS_2` | 多账号 | 账号2 |
| `NS_COOKIE_3` / `NS_PROXY_3` / `NS_USER_3` / `NS_PASS_3` | 多账号 | 账号3 |
| `DD_BOT_TOKEN` / `DD_BOT_SECRET` | 可选 | 钉钉（加签） |
| `BARK_URL` / `SERVERCHAN_KEY` / `WEBHOOK_URL` | 可选 | 其他推送 |

> Cookie 属于账号凭据，只放 Secret，**不要**提交到任何文件里。Secret 写进去之后谁都读不出来，只在运行时交给脚本。

## 手动触发

`Actions → 账号1 签到 → Run workflow`

- `force`：忽略本地"今天已签"的记录，再问一次站点（站点自己会说"已经签过"，不会重复签）
- `login`：用账号密码登录一次并抓最新 Cookie。验证码会发到你的 Telegram，**在 TG 里回那 6 位数字**即可；
  抓到的 Cookie 会自动存进私库，之后就一直用 Cookie 签到

## 加一个账号

1. 加 Secret：`NS_COOKIE_4`、`NS_PROXY_4`
2. 复制 `account-3.yml` → `account-4.yml`，把里面的 `3` 全改成 `4`（文件名也建议 `account-4.yml`）
3. 想要每天自动跑就解除 `schedule:` 注释，并把 cron 再错开几分钟

## 安全说明

- 一个号一个代理（`NS_PROXY_N`），出口 IP 固定 —— Cloudflare 的清关凭据和 IP+UA 绑定，换 IP 就得重过一次人机验证
- 只签到，不评论、不加鸡腿、不刷任何东西
- 网络请求之间是 3~9 秒的随机"思考"间隔，启动还有 0~90 秒抖动
- Cookie 失效时工作流会告诉你，重新在本机跑一次 `tools/capture_cookie.py` 抓 Cookie 即可

## 免责

仅供学习自动化运维使用。请遵守站点规则，自行承担使用风险。

## 致谢

思路参考：`wugeng20/NodeSeekSignin`、`HoseaDev/NodeSeek-Daily`、`dragonff/NodeSeek`。
