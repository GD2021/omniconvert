# 万象转换 (OmniConvert) 资产与部署文档

- **项目名称**：万象转换 OmniConvert（纯浏览器端本地文件格式转换工具：PDF/图片/音频/文档 24+ 款工具，100% 本地运算，零数据外发）
- **官方仓库**：https://github.com/Fatallove101/omniconvert
- **GitHub 维护仓库**：https://github.com/GD2021/omniconvert
- **生产访问自定义域名**：https://omniconvert.lucky.cloudns.biz
- **边缘 Workers 默认域名**：https://omniconvert.ak6469515961668.workers.dev
- **架构特点**：采用 Cloudflare Workers + Static Assets 边缘架构。前端静态资源经构建后由 Cloudflare 全球 CDN 静态网络高速分发，通过 Cloudflare Workers Custom Domain 直接绑定 `omniconvert.lucky.cloudns.biz`，全链路 HTTPS，无需购买服务器，零维护成本。

---

## 规范命名资产清单

| 资源类型 | Cloudflare 资源名称 / 标识 | 归属 Zone / 绑定 | 费用与状态 | 说明 |
|---|---|---|---|---|
| **Worker 服务** | `omniconvert` | Account: `Waka` (`fc6f79e77dca61be461e2dedb82244f5`) | Free Plan（已部署） | 运行 Cloudflare Workers 静态资产托管引擎，分发 Web 应用 |
| **静态资产 (Assets)** | `dist` 目录 (48 个核心文件，~9 MB) | `assets.directory: "./dist"` | Free Plan（已同步） | 包含 WebAssembly (qpdf, sql.js)、JS 解码器、CSS、Icons 等 |
| **自定义域名** | `omniconvert.lucky.cloudns.biz` | Zone: `lucky.cloudns.biz` (`4fb4ad18a6c1f968afac4c6ed816d010`) | Free Plan（已生效） | Worker Custom Domain 自动生成只读 AAAA 解析（`100::`）并提供 Cloudflare 托管 SSL |
| **可观测性** | `observability.enabled = true` | Worker 级别实时指标与日志 | Free Plan（已开启） | 监控边缘访问状态与网络异常 |

---

## 自动化同步与部署配置

- **Workflow 文件**：`.github/workflows/sync-and-deploy.yml`
- **定时触发**：每天北京时间凌晨 **04:07**（换算 UTC 时间 `20:07`，避免整点高峰调度延迟）自动检查官方上游更新
- **手动触发入口**：支持在 GitHub 仓库 Actions 页面手动点击 `Run workflow` 触发（支持 `force_deploy` 参数）
- **执行逻辑**：
  1. 检查官方上游 `Fatallove101/omniconvert:main` 分支的新提交与 Tags；
  2. 若无上游更新且未指定强制部署，正常结束工作流，不消耗 CI 与构建资源；
  3. 若检测到上游更新，记录提交差异，并在保护本地定制配置（`wrangler.jsonc`）的前提下执行自动合并；
  4. 执行前端代码验证与测试（`test-docs.mjs` 与 `test-check-music.mjs`）；
  5. 执行静态打包构建（`node scripts/build-web.mjs`）并校验构建产物完整性；
  6. 验证通过后将合并分支推送到 `origin main`，并执行 `npx wrangler deploy` 发布至生产环境；
  7. 部署后执行端到端健康检查（访问 `https://omniconvert.lucky.cloudns.biz/` 验证 HTTP 200）；
  8. 若出现合并冲突或构建测试失败，自动中断发布并回滚，保留当前线上正常运行的版本。
- **仓库 Secrets 配置**：
  - `CLOUDFLARE_ACCOUNT_ID`：`fc6f79e77dca61be461e2dedb82244f5`（已配置到 GitHub Secrets）
  - `CLOUDFLARE_API_TOKEN`：待配置（已由本地环境完成首次上线验证；如需 GitHub Actions 每天在云端全自动触发无人工干预部署，需在 GitHub 仓库添加具有 Workers 部署权限的 API Token）。

---

## 本地日常维护指引

进入本地项目目录：
```bash
cd /Users/jack/Desktop/运维/Cloudflare_项目/omniconvert
```

- **本地打包 Web 产物**：
  ```bash
  node scripts/build-web.mjs
  ```
- **本地即时部署到 Cloudflare**：
  ```bash
  npx wrangler deploy
  ```
- **查看线上实时日志**：
  ```bash
  npx wrangler tail
  ```
- **手工同步上游更新**：
  ```bash
  git fetch upstream main
  git merge upstream/main
  git push origin main
  ```
