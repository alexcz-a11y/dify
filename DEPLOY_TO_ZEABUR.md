# 在 Zeabur 上部署 Dify

本指南将帮助你在 Zeabur 平台上手动部署 Dify，而不使用模板方式。

## 前置要求

1. 拥有 Zeabur 账户（需要升级到 Developer Plan，$5/月）
2. 已克隆 Dify 官方仓库到本地
3. 了解基本的 Git 操作

## 部署步骤

### 1. 准备项目

确保你的项目已经还原到官方原版：
```bash
git status
# 应该显示 "Your branch is up to date with 'origin/main'"
```

### 2. 推送配置到 GitHub

将包含 Zeabur 配置的项目推送到你的 GitHub 仓库：
```bash
git add .
git commit -m "Add Zeabur deployment configuration"
git push origin main
```

### 3. 在 Zeabur 上创建项目

1. 登录 [Zeabur Dashboard](https://zeabur.com/dashboard)
2. 点击 "Create Project"
3. 选择 "Deploy from Git"
4. 选择你的 GitHub 仓库
5. 选择 "main" 分支

### 4. 解决检测问题

如果 Zeabur 将项目识别为"静态 HTML5 站点"：

1. **检查配置文件**：确保根目录有 `zeabur.toml` 文件
2. **重新部署**：点击"配置"按钮，然后重新部署
3. **手动配置**：如果自动检测失败，可以手动配置服务

### 5. 配置服务

Zeabur 会自动检测到 `zeabur.toml` 配置文件，并创建以下服务：

#### 数据库服务
- **dify-postgres**: PostgreSQL 数据库（版本 16）
- **dify-redis**: Redis 缓存（版本 7）
- **dify-weaviate**: 向量数据库（版本 1.22.4）

#### 应用服务
- **dify-api**: Python Flask API 服务（端口 5001）
- **dify-web**: Next.js Web 前端服务（端口 3000）

### 6. 环境变量配置

在 Zeabur Dashboard 中，为每个服务配置必要的环境变量：

#### API 服务环境变量
```
FLASK_APP=app.py
EDITION=SELF_HOSTED
DEPLOY_ENV=PRODUCTION
POSTGRES_USER=postgres
POSTGRES_PASSWORD=difyai123456
POSTGRES_DB=dify
POSTGRES_HOST={{services.dify-postgres.internalHost}}
POSTGRES_PORT={{services.dify-postgres.port}}
REDIS_HOST={{services.dify-redis.internalHost}}
REDIS_PORT={{services.dify-redis.port}}
REDIS_PASSWORD=difyai123456
WEAVIATE_HOST={{services.dify-weaviate.internalHost}}
WEAVIATE_PORT={{services.dify-weaviate.port}}
CONSOLE_API_URL=https://{{services.dify-api.domain}}
CONSOLE_WEB_URL=https://{{services.dify-web.domain}}
SERVICE_API_URL=https://{{services.dify-api.domain}}
APP_WEB_URL=https://{{services.dify-web.domain}}
```

#### Web 服务环境变量
```
NODE_ENV=production
EDITION=SELF_HOSTED
DEPLOY_ENV=PRODUCTION
NEXT_TELEMETRY_DISABLED=1
CONSOLE_API_URL=https://{{services.dify-api.domain}}
APP_API_URL=https://{{services.dify-api.domain}}
MARKETPLACE_API_URL=https://marketplace.dify.ai
MARKETPLACE_URL=https://marketplace.dify.ai
```

### 7. 部署和启动

1. 点击 "部署" 开始部署
2. 等待所有服务构建完成
3. 检查服务状态，确保所有服务都正常运行

### 8. 域名配置

1. 为 API 和 Web 服务分配自定义域名
2. 更新环境变量中的 URL 配置
3. 重新部署服务以应用新的域名

## 故障排除

### 常见问题

1. **Zeabur 检测为静态站点**
   - 确保根目录有 `zeabur.toml` 文件
   - 检查配置文件格式是否正确
   - 尝试重新部署或手动配置

2. **构建失败**
   - 检查 `zeabur.toml` 配置是否正确
   - 确保所有依赖都已正确安装
   - 查看构建日志获取详细错误信息

3. **服务无法启动**
   - 检查环境变量配置
   - 查看服务日志以获取错误信息
   - 确保数据库服务已启动

4. **数据库连接失败**
   - 确保数据库服务已启动
   - 检查连接字符串和凭据
   - 验证服务间网络连接

### 日志查看

在 Zeabur Dashboard 中：
1. 选择相应的服务
2. 点击 "Logs" 标签
3. 查看实时日志以诊断问题

## 维护

### 更新部署

当 Dify 发布新版本时：
1. 拉取最新代码：`git pull upstream main`
2. 推送更新：`git push origin main`
3. 在 Zeabur 中重新部署项目

### 备份

定期备份数据库：
1. 在 Zeabur Dashboard 中导出 PostgreSQL 数据
2. 保存配置文件和环境变量

## 成本估算

- **Developer Plan**: $5/月（必需）
- **服务资源**: 根据使用量计费
- **域名**: 免费（Zeabur 提供）

## 支持

如果遇到问题：
1. 查看 [Zeabur 文档](https://docs.zeabur.com/)
2. 查看 [Dify 文档](https://docs.dify.ai/)
3. 在 [Zeabur Discord](https://discord.gg/zeabur) 寻求帮助 