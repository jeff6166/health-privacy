# 让 Google 授权「永久有效」（约 10 分钟，做一次即可）

## 为什么需要做这一步
Google 的 OAuth 应用若停留在**「测试」**状态，refresh token **每 7 天过期**（所以要每周重新授权一次）。
把应用**发布到生产模式**后，这个 7 天限制就被解除 → 以后不用再每周点授权链接。

> ⚠️ 关键区分：**「发布」≠「提交验证」**
> - **发布**：只要求填「应用名称 / 支持邮箱 / 首页网址 / 隐私权政策网址」四项 —— 需要公司名称或产品介绍 ❌
> - **提交验证**：那是另一个流程（要用途说明、演示视频等），只在你想去掉「未验证应用」警告时才需要 —— **个人应用不需要**
> - 发布后仍会显示「Google 未验证的应用」→ 点「高级 → 继续前往」照样能过（跟现在一样）
> - 未验证应用有 100 用户上限 —— 本应用只有 1 个用户（你本人），无影响

## 要做的事：给这两个网址找一个公开托管
Google 要求首页与隐私政策**公开可访问**（不能需要登录），且**必须同域**。
本目录两个文件即可：`index.html`（首页）+ `privacy.html`（隐私政策）

- 首页网址：`https://<你的GitHub用户名>.github.io/health-privacy/`
- 隐私权政策网址：`https://<你的GitHub用户名>.github.io/health-privacy/privacy.html`

> 关于 `github.io`：按公共后缀表规则，`<用户名>.github.io` 视作一个可注册域，通常能被 Google 接受；若被拒 → 改用 Cloudflare Pages（`xxx.pages.dev`）或自己的域名，步骤类似。`noindex` 已加好（不被搜索引擎收录，但链接可访问 ✓ 满足 Google 要求）。

### 方式 A：网页操作（不需要命令行）
1. 打开 https://github.com/new
2. Repository name 填 `health-privacy`，选 **Public**，勾选 "Add a README file"，Create
3. 进入仓库 → **Add file → Upload files** → 上传 `index.html` 和 `privacy.html` → Commit
4. 仓库 → **Settings → Pages** → Source 选 `Deploy from a branch` → Branch `main` + `/ (root)` → Save
5. 等 1–2 分钟，访问 `https://<用户名>.github.io/health-privacy/` 确认能打开

### 方式 B：命令行（本机 git 已就绪）
```bash
cd ~/privacy-pages
git init -b main
git add index.html privacy.html README-部署步骤.md
git commit -m "Add app homepage and privacy policy for Google OAuth consent screen"
# 先在 GitHub 上建好空仓库 health-privacy（Public，不要勾 README）
git remote add origin https://github.com/<你的用户名>/health-privacy.git
git push -u origin main
# 然后到仓库 Settings → Pages → Source: main / (root) → Save
```

### 方式 C：零账号应急（Netlify Drop）
打开 https://app.netlify.com/drop → 把 `~/privacy-pages` 文件夹拖进去 → 立即得到一个公开网址。
（可先用它验证流程；但为了长期稳定，建议用 GitHub Pages 或 Cloudflare Pages 并保留账号。）

## 填回 Google Cloud → 发布
1. 打开品牌页 https://console.cloud.google.com/auth/branding?project=my-health-508411
2. **应用首页** 填 `https://<用户名>.github.io/health-privacy/`
3. **应用隐私权政策链接** 填 `https://<用户名>.github.io/health-privacy/privacy.html`
4. 若要求 **已获授权的网域**：填 `<用户名>.github.io`
5. 保存 → 打开「受众」页 https://console.cloud.google.com/auth/audience → 点 **发布应用** → 确认

> 若「发布应用」仍是灰色：检查 ① 应用名称非空（且与首页标题一致即可）② 支持邮箱 ③ 首页网址 ④ 隐私权政策网址 是否都已保存。四项齐了按钮就会变亮。

## 如何验证已经生效
1. 发布后**再授权一次**（`bash ~/.hermes/scripts/google-health/start-auth-wait.sh --all-scopes`）
2. 记录授权日期 → **8 天后**运行 `python3 ~/.hermes/scripts/google-health/gh_auth.py --check`
   → 显示 `AUTHENTICATED`（不再是 `AUTH_FAILED: refresh token 已失效`）就说明 7 天限制已解除 ✓

## 备注
- 浏览器设好了还要记得 `--all-scopes`（否则会丢掉 ECG / 饮食 / 位置 / 资料 4 个权限）
- 即使不做这一步，也已经有保底：每晚 21:00 的授权提醒会自动把新链接推给你（只在失效时发），加上脚本里的自动救援 —— 不会漏数据
