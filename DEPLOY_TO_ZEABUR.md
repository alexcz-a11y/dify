# 在 Zeabur 上部署 Dify - 手动配置指南

本指南将帮助你在 Zeabur 平台上手动部署 Dify，解决自动检测问题。

## 问题分析

Zeabur 将项目识别为"静态 HTML5 站点"的原因：
1. Zeabur 无法自动检测多服务项目
2. 需要手动配置每个服务
3. 配置文件格式可能不被支持

## 解决方案：手动配置部署

### 1. 准备项目

确保你的项目已经还原到官方原版：
```bash
git status
# 应该显示 "Your branch is up to date with 'origin/main'"
```

### 2. 推送配置到 GitHub

```bash
git add .
git commit -m "Add Zeabur YAML configuration"
git push origin main
```

### 3. 在 Zeabur 上创建项目

1. 登录 [Zeabur Dashboard](https://zeabur.com/dashboard)
2. 点击 "Create Project"
3. 选择 "Deploy from Git"
4. 选择你的 GitHub 仓库：`alexcz-a11y/dify`
5. 选择 "main" 分支

### 4. 手动配置服务

由于 Zeabur 无法自动检测多服务项目，需要手动创建每个服务：

#### 步骤 1：创建数据库服务

1. **创建 PostgreSQL 服务**：
   - 点击 "Add Service"
   - 选择 "PostgreSQL"
   - 版本选择 "16"
   - 服务名称：`dify-postgres`

2. **创建 Redis 服务**：
   - 点击 "Add Service"
   - 选择 "Redis"
   - 版本选择 "7"
   - 服务名称：`dify-redis`

3. **创建 Weaviate 服务**：
   - 点击 "Add Service"
   - 选择 "Weaviate"
   - 版本选择 "1.22.4"
   - 服务名称：`dify-weaviate`

#### 步骤 2：创建 API 服务

1. 点击 "Add Service"
2. 选择 "Deploy from Git"
3. 选择同一个仓库：`alexcz-a11y/dify`
4. 在配置中设置：
   - **Service Name**: `dify-api`
   - **Source Directory**: `api`
   - **Build Command**: `pip install uv && uv sync --locked`
   - **Start Command**: `python app.py`
   - **Port**: `5001`

#### 步骤 3：创建 Web 服务

1. 点击 "Add Service"
2. 选择 "Deploy from Git"
3. 选择同一个仓库：`alexcz-a11y/dify`
4. 在配置中设置：
   - **Service Name**: `dify-web`
   - **Source Directory**: `web`
   - **Build Command**: `npm install -g pnpm && pnpm install --frozen-lockfile && pnpm build`
   - **Start Command**: `pnpm start`
   - **Port**: `3000`

### 5. 配置环境变量

#### API 服务环境变量

在 `dify-api` 服务的设置中添加以下环境变量：

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

在 `dify-web` 服务的设置中添加以下环境变量：

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

### 6. 部署和启动

1. 确保所有服务都已创建
2. 点击每个服务的 "Deploy" 按钮
3. 等待所有服务构建完成
4. 检查服务状态，确保所有服务都正常运行

### 7. 域名配置

1. 为 API 和 Web 服务分配自定义域名
2. 更新环境变量中的 URL 配置
3. 重新部署服务以应用新的域名

## 替代方案：使用 Zeabur CLI

如果手动配置太复杂，可以尝试使用 Zeabur CLI：

```bash
# 安装 Zeabur CLI
npm install -g @zeabur/cli

# 登录
zeabur login

# 部署项目
zeabur deploy
```

## 故障排除

### 常见问题

1. **服务无法启动**
   - 检查环境变量配置
   - 查看服务日志以获取错误信息
   - 确保数据库服务已启动

2. **构建失败**
   - 检查构建命令是否正确
   - 确保所有依赖都已正确安装
   - 查看构建日志获取详细错误信息

3. **数据库连接失败**
   - 确保数据库服务已启动
   - 检查连接字符串和凭据
   - 验证服务间网络连接

4. **环境变量未生效**
   - 确保环境变量格式正确
   - 重新部署服务以应用新的环境变量
   - 检查服务间引用格式

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
3. 在 Zeabur 中重新部署相关服务

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