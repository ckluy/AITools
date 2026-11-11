# 纯前端 AI 工具箱

一个单文件 `index.html`，可以直接部署到 GitHub Pages。内置 9 个工具：摘要、翻译、改写、JSON 抽取、测验生成、回复起草、代码解释、FAQ 生成、文案校对。

支持以下 AI 服务：

- OneTag AI（推荐，纯静态网页可直接调用）
- 通义千问（DashScope OpenAI 兼容接口）
- 豆包（火山方舟 OpenAI 兼容接口）
- Google Gemini
- 任意 OpenAI 兼容接口

## 本地预览

直接双击 `index.html` 即可，也可以启动一个本地静态服务器：

```powershell
python -m http.server 8000
```

然后打开 `http://localhost:8000/`。

## 部署到 GitHub Pages

### 方式一：网页上传（最简单）

1. 登录 GitHub，点击 **New repository**，填写仓库名，选择 **Public**，创建仓库。
2. 进入仓库，点击 **Add file → Upload files**，上传 `index.html`，然后点击 **Commit changes**。
3. 打开仓库的 **Settings → Pages**。
4. 在 **Build and deployment** 下，Source 选择 **Deploy from a branch**。
5. Branch 选择 `main`，文件夹选择 `/(root)`，点击 **Save**。
6. 等待 1-2 分钟，访问：

```text
https://你的用户名.github.io/仓库名/
```

如果仓库名是 `你的用户名.github.io`，则访问地址是：

```text
https://你的用户名.github.io/
```

### 方式二：Git 命令行

先在 GitHub 创建一个空仓库（不要勾选自动生成 README），然后在本目录执行：

```powershell
git init
git add .
git commit -m "deploy pure frontend AI toolbox"
git branch -M main
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main
```

推送成功后，仍然按上面的 **Settings → Pages** 步骤，选择 `main` 分支和 `/(root)` 目录。

以后更新页面只需要：

```powershell
git add index.html
git commit -m "update AI toolbox"
git push
```

GitHub Pages 会自动重新部署。

## 接入真实 AI

打开页面顶部的 **AI 设置**，选择服务、填写 Key，点击 **保存并使用**。Key 默认只保存在当前浏览器的 `localStorage`，不会写进 `index.html`。

### OneTag AI（推荐）

1. 到 [OneTag AI 领取免费 Key](https://onetagai.com/#get-key)，用邮箱注册。
2. 页面里选择 **OneTag AI**，粘贴 Key（通常以 `ot_` 开头），保存。
3. 第一次调用工具时，页面会按需加载 `https://onetagai.com/v1.js`，然后调用：

```js
const answer = await onetag.ask(prompt, '', {
  system: '你是一个专业的AI助手',
  maxTokens: 3000
});
```

OneTag 的免费 Key 按设计可以在静态页面中使用，适合个人开发和初期上线。具体额度和规则以官网为准。

### 通义千问 / 豆包 / Gemini

在 AI 设置里切换到对应服务：

- 通义千问：填 DashScope API Key，默认接口是 `https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions`，模型默认 `qwen-plus`。
- 豆包：填火山方舟 API Key，模型通常填推理接入点 ID（`ep-...`）或模型 ID。
- Gemini：填 Gemini API Key，默认模型 `gemini-2.5-flash`。Gemini 在中国大陆通常需要自行解决网络访问问题。
- 自定义：填任意 OpenAI 兼容的 `/chat/completions` 接口地址、模型和 Key。

## 密钥安全说明

GitHub Pages 是纯静态托管，没有后端，**不要指望把服务端私钥藏在前端代码里**。任何写进 HTML 或浏览器能读到的 Key，都可能被访问者看到。

- 本页面的设置面板会把 Key 存在当前浏览器 `localStorage`，适合“每个用户填自己的 Key”。
- OneTag 的 Key 本来就是面向静态网页设计的公开 Key，可以直接用于纯前端页面；免费额度仍然可能被他人消耗，公开上线前请评估。
- 如果你要做一个所有访客共用额度的公开工具，并且使用通义千问、豆包、Gemini 等私有 Key，请增加一个后端代理（Cloudflare Workers、Netlify Functions、Vercel Functions 等），前端只调用你的代理，不直接接触真实 Key。

## 广告位配置

页面预留了 3 个响应式广告位：

- `top`：AI 设置下方、工具列表上方
- `inFeed`：前 3 个工具卡片之后
- `bottom`：工具列表下方、页脚上方

配置集中在 `index.html` 的 `AD_CONFIG`，搜索 `===== 广告位配置 =====` 即可找到。默认显示“广告位预览”，不会加载任何广告脚本。

### 接入 Google AdSense

1. 在 AdSense 后台添加站点并通过审核。
2. 创建广告单元，拿到 `ca-pub-...` 和每个广告位的 `data-ad-slot` ID。
3. 修改 `AD_CONFIG`：

```js
const AD_CONFIG = {
    enabled: true,
    provider: 'adsense',
    showPlaceholders: false,
    adsense: {
        client: 'ca-pub-1234567890123456',
        slots: {
            top: '1111111111',
            inFeed: '2222222222',
            bottom: '3333333333'
        }
    },
    customHtml: { top: '', inFeed: '', bottom: '' },
    labels: {
        top: '顶部横幅广告位 · 970 × 90',
        inFeed: '内容中插广告位 · 自适应',
        bottom: '页底横幅广告位 · 970 × 90'
    }
};
```

页面会在有可用槽位时按需加载 `adsbygoogle.js`，没配置的广告位继续显示预览框或自动隐藏。

### 接入百度联盟或其他广告

把 `provider` 改成 `'custom'`，将广告代码粘贴到对应位置的 `customHtml`：

```js
provider: 'custom',
customHtml: {
    top: '<你的百度联盟广告代码>',
    inFeed: '',
    bottom: ''
}
```

自定义 HTML 里的 `<script>` 会被重新创建并执行，因此可以兼容常见的联盟广告代码。正式上线前请确认广告平台允许你的域名、页面内容和展示方式。

### 广告合规

- 不要诱导点击广告，也不要给广告位加遮挡或自动点击逻辑。
- 如果使用 Google AdSense，建议补充隐私政策页面，并按平台要求配置 `ads.txt`。
- GitHub Pages 默认域名可以用于测试，正式商业化建议绑定自己的域名。

## 常见问题

**打开 GitHub Pages 是 404**

确认仓库根目录有 `index.html`，并且 Pages 配置为 `main` 分支的 `/(root)`。如果是第一次部署，等待 1-2 分钟。

**改了文件但线上没变化**

确认已经 `git push`，然后查看仓库的 **Actions / Deployments** 是否成功；浏览器可以强制刷新。

**OneTag 调用失败**

检查 Key、网络、广告拦截器，以及 OneTag 的免费额度是否用完。错误提示会显示 `no_key`、`bad_key`、`quota_exceeded`、`rate_limited` 等具体原因。

**通义千问或豆包报跨域错误**

它们的 OpenAI 兼容接口目前通常允许浏览器跨域，但服务商策略可能变化。遇到 CORS 问题时，请改用 OneTag，或增加自己的后端代理。

