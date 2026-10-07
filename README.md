# dist-frontend 发布分支

GitHub Pages 源分支，由 script/dev_bash/build-frontend-publish.sh 生成并推送，服务两个独立部署的前端 SPA（子目录自动识别部署根，无环境变量/注入）。

- CNAME：自定义域名（lot-frontend-b.leiyanhui.com）
- favicon.ico：站点根 favicon（来自 frontend/icon/favicon.ico）
- robots.txt：根目录及 ops-ui/、web-ui/ 子目录各一份（禁止收录）
- 404.html：SPA 兜底加载页（保留路径，自动加载对应部署根部）
- ops-ui/：运维控制台 SPA
- web-ui/：业务 WebUI SPA（含 download/ 与 robots.txt）
