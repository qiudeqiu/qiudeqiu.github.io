# Drawnix for macOS — OAuth 品牌页

Google 把「外部 + 生产」模式下的首页和隐私政策设成了硬性门槛。这个目录就是用来满足它的最小站点：
三个静态页，零依赖、零外部资源，随便扔哪个静态托管都能跑。

```
index.html    应用首页（产品说明 + 隐私政策入口 + 联系方式）
privacy.html  隐私政策（含 Google API 用户数据「有限使用」声明）
terms.html    服务条款
```

## 第一步：检查邮箱（已填好）

三个页面里的联系方式与页脚已经统一填成 **qiudeqiu@gmail.com**，不用再改。想换的话：

```bash
cd google-oauth-site
sed -i '' 's/qiudeqiu@gmail.com/新邮箱@gmail.com/g' index.html privacy.html terms.html
```

首页正文里还有一处 `<a href="https://github.com/plait-board/drawnix">`，如果想把「开源」链接指到自己的仓库，一并改掉。

> 品牌页里的「用户支持邮箱」填同一个 `qiudeqiu@gmail.com`，Google 会核对首页上的联系方式能否收信。

## 第二步：托管（二选一）

### 方案 1：GitHub Pages（免费，5 分钟）

1. GitHub 上新建一个**公开**仓库，比如 `drawnix-site`；
2. 把这三个 html 放进仓库根目录，`git push`；
3. 仓库 Settings → Pages → Source 选 `Deploy from a branch`，分支 `main`、目录 `/ (root)`，保存；
4. 等 1–2 分钟，地址是：

```
https://<你的GitHub用户名>.github.io/drawnix-site/
```

> 如果你把仓库命名为 `<你的GitHub用户名>.github.io`，地址会直接是根路径，域名也更好看，推荐这种。

### 方案 2：Cloudflare Pages（免费，可绑自定义域名）

新建项目 → 直连该仓库 → 构建命令留空、输出目录填 `/`。拿到的地址形如 `https://drawnix-site.pages.dev`。

## 第三步：填进 Google 品牌页

到 [Google Auth Platform](https://console.cloud.google.com/auth/branding) → **品牌信息**：

| 字段 | 填什么（以 GitHub Pages 为例） |
| --- | --- |
| 应用名称 | `Drawnix for macOS`（必须和首页标题一致） |
| 用户支持邮箱 | 你的 Gmail |
| 应用徽标 | **留空**（上传徽标等于承诺要做应用验证，别碰） |
| 应用首页 | `https://<用户名>.github.io/drawnix-site/` |
| 应用隐私权政策链接 | `https://<用户名>.github.io/drawnix-site/privacy.html` |
| 应用服务条款链接 | `https://<用户名>.github.io/drawnix-site/terms.html` |
| 已获授权的网域 | `<用户名>.github.io` |

保存后回 **受众（Audience）→ 发布应用（Publish app）**，状态应从「测试」变为「正式版」。

## 如果报「必须是有效的顶级私有域名」

Google 用公共后缀表（PSL）校验域名。`github.io`、`pages.dev`、`vercel.app` 本身是公共后缀，
但 `<用户名>.github.io` 在 PSL 里是一个可注册的私有域，通常能通过；如果控制台仍然拒绝，就走自定义域名：

1. 买个域名（`.com` / `.dev` 一年几十块，Cloudflare Registrar 或 Namecheap 都行），不需要备案；
2. Cloudflare Pages 里绑定该域名（一条 CNAME 记录）；
3. 品牌页里的三处链接和「已获授权的网域」相应换成你的域名，隐私政策与首页保持同一域名即可
   （Google 要求隐私政策托管在首页所在域名下）。

## 几个别踩的坑

- **别点「提交验证」（Verification Center）**。`auth/drive` 是受限 scope，正式验证要走 CASA 第三方安全评估，
  要钱要几周。个人自用不验证也能发布，代价只有两点：首次授权多一个「Google 尚未验证此应用」页面
  （点「高级 → 继续前往」），以及未验证应用最多 100 个用户。一个人用完全够。
- **发布后仍是「未验证」，这不代表失败**。令牌寿命已经变成正常长度，不会再 7 天过期。
- 如果保存时提示「无法访问该网址」，多半是 Pages 还没构建完，等两分钟重试，或在浏览器里确认链接能打开。
- 页面里的隐私政策链接、页脚链接都用相对路径，所以必须把三页放在**同一个路径层级**下。

## 第四步：回到应用重新登录一次

发布成功后，在 Drawnix 里「文件 ▸ Google Drive 同步… → 重新登录」一次，拿到新的长效 refresh token。
以后除非你主动撤权，`DXGoogleDriveAuth.m` 里那条 `invalid_grant` 分支不会因为「7 天」再触发。
