# pi-web-ui Community Plugin Registry

> `pi-web-ui` 官方社区第三方插件清单仓库，供独立开发者与贡献者提交插件、GitHub Actions 自动化安全审查（CI Audit）与自动合并（Auto-Merge）。
>
> 💡 **说明**：`pi-web-ui` 项目自带的官方内置插件（如 notes, vscode-editor, browser-use, tts 等）随主仓库直接分发维护，不需要且不在此仓库提交。本仓库专供第三方社区独立插件提交与收录。

---

## 🌟 仓库特性

- **独立元数据管理**：每个插件对应独立目录 `plugins/<id>/plugin.json`，彻底消除 PR 并发合并冲突。
- **CI 自动化安全审查**：
  - PR 范围校验（单 PR 仅允许修改单一插件）。
  - JSON Schema 格式合法性校验。
  - 远端仓库真实性、开源协议与 `manifest.json` 一致性校验。
  - SVG 图标 XSS 过滤与权限风险评级（LOW / MEDIUM / HIGH）。
- **无人值守自动合并**：低风险（LOW）插件 PR 自动审查通过后，机器人自动 Approve 并合并。
- **自动化发布与同步**：合并后自动构建 `dist/catalog.json`，并发布至 GitHub Pages，供所有 `pi-web-ui` 客户端订阅。

---

## 📦 如何在 pi-web-ui 中订阅本清单

> ✅ **本清单已是 `pi-web-ui` 的官方默认清单**：服务端启动时若未配置 `PI_WEB_PLUGIN_CATALOG_URL`，
> 会自动拉取 `https://xing-shuyin.github.io/pi-web-ui-plugins/catalog.json` 并同步进插件市场，无需任何配置。

### 方式 1：覆盖默认清单（可选）

如需改用自定义清单或本地文件，启动 `pi-web-ui` 时显式指定：

```bash
export PI_WEB_PLUGIN_CATALOG_URL="https://your-own-catalog.example/catalog.json"
pi-web-ui
```

### 方式 2：关闭默认同步（可选）

显式设为空串或 `off` / `0` / `false` / `no` 即可关闭开机预同步：

```bash
export PI_WEB_PLUGIN_CATALOG_URL="off"
pi-web-ui
```

### 方式 3：客户端动态同步

在 `pi-web-ui` 界面中，通过支持目录同步的插件或控制台直接调用：

```javascript
window.__piWebUiHost.reloadCatalog(
  "https://xing-shuyin.github.io/pi-web-ui-plugins/catalog.json",
  { replace: false },
);
```

---

## 🛠️ 本地开发与测试

```bash
# 1. 安装依赖
npm install

# 2. 全量审计现有插件
npm test

# 3. 针对特定插件进行审计测试
node scripts/audit.mjs --target pi-web-multigit

# 4. 构建聚合目录
npm run build
```

---

## 🚀 仓库创建与部署指南（维护者）

如果你是维护者，准备将本仓库推送到 GitHub：

1. **新建 GitHub 仓库**：
   在 GitHub 上创建一个公开仓库，例如 `pi-web-ui-plugins`。
2. **推送代码**：
   ```bash
   cd plugin-registry
   git init
   git branch -M main
   git remote add origin git@github.com:xing-shuyin/pi-web-ui-plugins.git
   git add .
   git commit -m "feat: initial commit of plugin registry with automated CI audit"
   git push -u origin main
   ```
3. **开启 GitHub Pages**：
   - 仓库设置 -> **Settings** -> **Pages**。
   - Source 选择 **GitHub Actions**。
4. **配置 PR 自动合并权限**：
   - 仓库设置 -> **Settings** -> **General** -> 勾选 **Allow auto-merge**。
   - 仓库设置 -> **Settings** -> **Actions** -> **General** -> **Workflow permissions** -> 选择 **Read and write permissions**，并勾选 **Allow GitHub Actions to create and approve pull requests**。

---

## 🤝 插件提交与自动化审查说明

本清单支持 **第三方开发者全自动提交插件** 与 **持续自动化安全审查**。只要你的开源插件满足基础规范，从提交到合并上架全流程无需人工介入！

### ⚡ 核心提交流程（第三方全自动上线）

```text
Fork 本仓库 -> 添加 plugins/<id>/plugin.json -> 发起 Pull Request -> CI 机器人自动化安全审查 -> 自动 Approve & 合并上线
```

1. **准备你的插件开源仓库**：
   - 拥有公开的 GitHub 仓库，并包含标准开源协议（如 MIT、Apache-2.0 等）；
   - 仓库根目录或子目录中包含合法的 `manifest.json`。
2. **Fork 本仓库并添加元数据文件**：
   在 `plugins/<你的插件ID>/` 目录下新建 `plugin.json`（可直接在 GitHub 网页端操作）：
   ```json
   {
     "id": "my-plugin",
     "name": "插件显示名称",
     "source": "github用户名/插件仓库名",
     "author": "github用户名",
     "description": "这里填写插件的中文功能描述（必填，不少于 5 个字）",
     "descriptionEn": "English description of the plugin (optional)",
     "icon": "🚀",
     "homepage": "https://github.com/github用户名/插件仓库名"
   }
   ```
3. **发起 Pull Request 并享受全自动审查**：
   - **单 PR 单插件**：每个 PR 仅添加或修改属于你自己的单一 `plugins/<id>/plugin.json`；
   - **每次修改严格自动审查**：提交 PR 以及后续针对该 PR 的**每一次修改与 Push**，CI 都会自动触发安全审计，包括：
     - JSON Schema 合规性校验；
     - 远端仓库真实性、开源协议与 `manifest.json` 一致性核对；
     - SVG 图标 XSS 脚本过滤与权限安全扫描；
     - 对已有插件的修改核验提交者与原作者一致性（防篡改保护）。
   - **无人值守自动合并（Auto-Merge）**：对于无高危敏感权限的普通工具/视图插件（LOW 风险），审查通过后机器人将**自动 Approve 并自动合并**，无需等待官方维护者手动审批。
4. **极速同步上架**：
   - PR 合并后，GitHub Actions 会在 1 分钟内自动重新构建 `dist/catalog.json` 并发布；
   - 全球所有启动中的 `pi-web-ui` 客户端即可在插件市场中直接搜索、浏览并一键安装你的插件！

> 💡 **官方内置插件说明**：随 `pi-web-ui` 核心程序发布的官方内置插件（如 notes, vscode-editor, browser-use, tts, tunnel 等）直接在 [主仓库 (xing-shuyin/pi-web-ui)](https://github.com/xing-shuyin/pi-web-ui) 的 `plugins/` 目录下维护，无需在本清单重复提交。本仓库专为第三方独立开源插件生态提供全自动收录通道。

更详细的字段定义与规范请参阅 [CONTRIBUTING.md](./CONTRIBUTING.md)。
