# Casdoor 学习指南

> 一个开源的统一身份认证和访问管理平台(IAM)

---

## 目录

1. [Casdoor 简介](#1-casdoor-简介)
2. [快速开始](#2-快速开始)
3. [核心概念](#3-核心概念)
4. [架构设计](#4-架构设计)
5. [功能特性](#5-功能特性)
6. [API 使用](#6-api-使用)
7. [配置说明](#7-配置说明)
8. [部署指南](#8-部署指南)
9. [最佳实践](#9-最佳实践)
10. [常见问题](#10-常见问题)

---

## 1. Casdoor 简介

### 1.1 是什么

Casdoor 是一个 **UI 优先的统一身份认证和访问管理平台 (IAM)**,提供现代化的 Web 界面。

**核心能力:**
- 用户注册、登录、找回密码
- 单点登录 (SSO)
- 多因素认证 (MFA)
- 权限管理 (RBAC)

### 1.2 核心特性

| 特性 | 说明 |
|------|------|
| **多协议支持** | OAuth 2.0、OIDC、SAML、LDAP、CAS、SCIM、WebAuthn、RADIUS |
| **社交登录** | 支持 30+ 社交平台(微信、QQ、GitHub、Google 等) |
| **多因素认证** | TOTP、SMS、Email、WebAuthn、Face ID |
| **多租户架构** | 支持多个组织和应用程序独立管理 |
| **可视化界面** | 提供完整的管理后台,无需编程即可配置 |

### 1.3 适用场景

- **企业身份管理**: 统一管理企业内部所有应用的用户认证
- **SaaS 应用**: 为 SaaS 产品提供专业的认证系统
- **单点登录**: 多个应用一次登录,处处通行
- **API 网关认证**: 保护 API 接口
- **移动应用认证**: 为移动 App 提供安全认证

---

## 2. 快速开始

### 2.1 Docker 部署(推荐)

```bash
# 1. 拉取镜像
docker pull casbin/casdoor

# 2. 运行(需要先准备 MySQL 数据库)
docker run -d -p 8000:8000 \
  -e dataSourceName="root:password@tcp(host.docker.internal:3306)/casdoor" \
  casbin/casdoor

# 3. 访问 http://localhost:8000
# 默认账号: admin / admin
```

### 2.2 Docker Compose

```yaml
version: '3.8'
services:
  casdoor:
    image: casbin/casdoor:latest
    ports:
      - "8000:8000"
    environment:
      - dataSourceName=root:123456@tcp(db:3306)/casdoor?charset=utf8mb4&parseTime=True&loc=Local
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=casdoor
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

启动: `docker-compose up -d`

### 2.3 从源码运行

```bash
# 克隆代码
git clone https://github.com/casdoor/casdoor.git
cd casdoor

# 安装依赖
go mod download
cd web && npm install && cd ..

# 初始化数据库
CREATE DATABASE casdoor CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
go run object/init_data.go

# 启动后端(默认端口 8000)
go run main.go

# 启动前端(新终端,默认端口 3001)
cd web && npm run start
```

---

## 3. 核心概念

### 3.1 组织 (Organization)

最高层级的管理单元,类似于企业中的"公司"或"部门"。

- 每个组织可以管理自己的用户、应用、角色
- 组织之间数据隔离
- 支持层级结构(父组织-子组织)

### 3.2 用户 (User)

认证的主体,包含:
- **基本信息**: 姓名、头像、邮箱、手机号
- **认证信息**: 密码(加密)、社交账号绑定
- **状态信息**: 是否禁用、是否删除
- **扩展属性**: 自定义字段

**关键属性:**
```
Owner: 所属组织
Name: 用户名(全局唯一)
DisplayName: 显示名称
Email: 邮箱(可用于登录)
Phone: 手机号(可用于登录)
```

### 3.3 应用 (Application)

代表使用 Casdoor 进行认证的具体系统(Web 网站、移动 App、后台管理系统)

**关键配置:**
```
Client ID / Client Secret: OAuth 客户端凭证
Redirect URIs: 回调地址(白名单)
Grant Types: 授权方式(授权码、密码、客户端凭证等)
Providers: 启用的认证方式(密码登录、社交登录等)
```

### 3.4 角色 (Role) 和权限 (Permission)

采用 **RBAC(基于角色的访问控制)** 模型:

```
用户 → 角色 → 权限 → 资源
```

**示例:**
```
用户: 张三
角色: 产品经理
权限: 查看产品列表、编辑产品、删除产品
```

### 3.5 提供商 (Provider)

外部身份源,包括:
- **社交登录**: GitHub、Google、微信、QQ 等
- **企业认证**: LDAP、RADIUS、SAML
- **存储服务**: 阿里云 OSS、AWS S3
- **邮件服务**: SMTP、SendGrid
- **短信服务**: 阿里云短信、腾讯云短信

---

## 4. 架构设计

### 4.1 技术栈

**后端:**
- Go 1.18+
- Beego v2 (Web 框架)
- XORM (ORM 框架)
- Casbin (权限引擎)
- MySQL/PostgreSQL

**前端:**
- React 18.2.0
- Ant Design 5.x
- CodeMirror (代码编辑器)
- ECharts (图表库)

### 4.2 项目结构

```
casdoor/
├── main.go              # 程序入口
├── conf/               # 配置管理
├── controllers/        # 控制器层
├── object/            # 数据模型层
├── routers/           # 路由配置
├── idp/              # 身份提供商
├── ldap/             # LDAP 集成
├── storage/          # 存储服务
└── web/              # 前端代码
```

### 4.3 请求流程

```
用户请求 → 路由层 → 中间件 → 控制器 → 业务逻辑 → 数据模型 → 数据库
```

---

## 5. 功能特性

### 5.1 认证功能

**密码登录:**
- 支持多种加密算法: bcrypt(推荐)、argon2id、PBKDF2

**社交登录:**
- 国内: 微信、QQ、支付宝、抖音、百度、新浪微博、钉钉
- 国外: Google、GitHub、Facebook、Twitter、Azure AD、Slack

**多因素认证 (MFA):**
- TOTP: 时间动态密码(Google Authenticator)
- SMS: 短信验证码
- Email: 邮箱验证码
- WebAuthn: 指纹、Face ID

### 5.2 授权功能

采用 **RBAC 模型**,支持:
- 灵活的角色和权限配置
- 细粒度的资源访问控制
- 权限继承和组合

### 5.3 用户管理

- 丰富的用户属性支持
- 自定义字段
- 批量导入/导出(Excel)

---

## 6. API 使用

### 6.1 核心认证 API

**用户登录:**
```http
POST /api/login
Content-Type: application/json

{
  "organization": "admin",
  "username": "admin",
  "password": "admin"
}

响应:
{
  "status": "ok",
  "data": {
    "user": {...},
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**获取用户信息:**
```http
GET /api/userinfo
Authorization: Bearer <token>
```

### 6.2 OAuth 2.0 流程

**授权码模式:**
```
1. 用户访问应用
2. 应用重定向到 Casdoor 授权页面
3. 用户登录并授权
4. Casdoor 回调应用,携带 code
5. 应用用 code 换取 token
6. 应用使用 token 访问 API
```

**示例:**
```javascript
// 1. 跳转到授权页面
window.location.href =
  `https://door.casdoor.com/login/oauth/authorize?` +
  `client_id=${client_id}&` +
  `redirect_uri=${redirect_uri}&` +
  `response_type=code&` +
  `scope=openid profile`;

// 2. 用 code 换取 token
const response = await fetch('/api/login/oauth/access_token', {
  method: 'POST',
  body: JSON.stringify({
    grant_type: 'authorization_code',
    client_id: clientId,
    client_secret: clientSecret,
    code: code,
  }),
});
```

### 6.3 SDK 使用

**JavaScript SDK:**
```javascript
import CasdoorSDK from 'casdoor-js-sdk';

const sdk = new CasdoorSDK({
  serverUrl: 'http://localhost:8000',
  clientId: 'your-client-id',
  appName: 'app-built-in',
  organizationName: 'admin',
  redirectPath: '/callback',
});

// 获取登录 URL
const url = sdk.getSigninUrl();
window.location.href = url;
```

---

## 7. 配置说明

### 7.1 主配置文件 (conf/app.conf)

```ini
appname = casdoor
httpport = 8000
runmode = dev

# 数据库配置
dataSourceName = root:123456@tcp(localhost:3306)/casdoor?charset=utf8mb4&parseTime=True&loc=Local

# Redis 配置(可选)
redisEndpoint = localhost:6379

# 静态资源配置
staticBaseUrl = https://cdn.casbin.org
```

### 7.2 应用配置

在管理后台配置应用:
```json
{
  "clientId": "客户端 ID",
  "clientSecret": "客户端密钥",
  "redirectUris": ["http://localhost:8080/callback"],
  "enableSignUp": true,
  "enablePassword": true,
  "providers": ["GitHub", "Google"]
}
```

---

## 8. 部署指南

### 8.1 宝塔面板部署(推荐新手)

**优势:** 图形化界面、零基础部署、可视化管理

**快速步骤:**
1. 安装宝塔面板
2. 软件商店安装: Nginx、MySQL、PM2
3. 创建数据库
4. 下载 Casdoor 二进制文件
5. 配置 app.conf
6. PM2 管理进程
7. 配置 Nginx 反向代理和 SSL

**Docker 部署(更简单):**
```bash
# 1. 安装 Docker
# 2. 创建 docker-compose.yml(见快速开始)
# 3. 启动: docker-compose up -d
```

### 8.2 Kubernetes 部署

```bash
# 添加 Helm 仓库
helm repo add casdoor https://casdoor.github.io/casdoor-helm-chart

# 安装
helm install casdoor casdoor/casdoor
```

### 8.3 Nginx 反向代理

```nginx
server {
    listen 443 ssl http2;
    server_name auth.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 9. 最佳实践

### 9.1 安全建议

1. **使用 HTTPS**: 生产环境必须配置 SSL 证书
2. **密码策略**: 设置密码复杂度要求
3. **启用 MFA**: 对管理员账号启用多因素认证
4. **定期备份**: 定期备份数据库和配置文件
5. **访问控制**: 遵循最小权限原则

### 9.2 性能优化

1. **数据库索引**:
   ```sql
   CREATE INDEX idx_user_owner_name ON user(owner, name);
   CREATE INDEX idx_token_user ON token(user);
   ```

2. **启用 Redis 缓存**:
   ```ini
   redisEndpoint = localhost:6379
   ```

3. **CDN 加速**:
   ```ini
   staticBaseUrl = https://cdn.example.com
   ```

---

## 10. 常见问题

### Q: 忘记管理员密码怎么办?

```sql
UPDATE user SET password = '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy'
WHERE name = 'admin';
-- 密码重置为: admin
```

### Q: 如何配置 LDAP 集成?

1. 在 "Providers" 中添加 LDAP 提供商
2. 配置连接信息(Host、Port、Base DN、Bind DN)
3. 在应用中启用 LDAP 登录

### Q: 如何实现单点登出?

```javascript
// 调用登出 API
fetch('/api/logout', { method: 'POST' });
// 清除本地 token
localStorage.removeItem('token');
```

### Q: 数据库迁移注意事项?

1. 先备份数据库
2. 使用相同的 Casdoor 版本
3. 确保数据库字符集为 utf8mb4
4. 检查时区设置

---

## 相关资源

- **官方网站**: https://casdoor.org
- **GitHub 仓库**: https://github.com/casdoor/casdoor
- **官方文档**: https://casdoor.org/docs
- **在线示例**: https://door.casdoor.com

---

**文档版本**: v2.0 (精简版)
**更新时间**: 2026-01-23
**适用于 Casdoor 版本**: v1.x

> 本文档从原版 2190 行精简至约 800 行,保留核心内容,删除冗余说明,聚焦快速上手和关键知识点。
