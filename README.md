# dist-frontend 发布分支

GitHub Pages 源分支，由 script/dev_bash/build-frontend-publish.sh 生成并推送，服务两个独立部署的前端 SPA（子目录自动识别部署根，无环境变量/注入）。

- CNAME：自定义域名（lot-frontend-b.leiyanhui.com）
- 404.html：SPA 兜底重定向页（自动定位部署根）
- ops-ui/：运维控制台 SPA
- web-ui/：业务 WebUI SPA（含 download/ 与 robots.txt）
