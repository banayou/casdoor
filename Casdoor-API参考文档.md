# Casdoor API 参考文档

> 完整的 API 接口说明、参数详解和代码示例

---

## 目录

1. [API 概述](#1-api-概述)
2. [认证方式](#2-认证方式)
3. [认证 API](#3-认证-api)
4. [用户管理 API](#4-用户管理-api)
5. [应用管理 API](#5-应用管理-api)
6. [组织管理 API](#6-组织管理-api)
7. [角色权限 API](#7-角色权限-api)
8. [资源管理 API](#8-资源管理-api)
9. [提供商 API](#9-提供商-api)
10. [令牌 API](#10-令牌-api)
11. [SDK 使用](#11-sdk-使用)
12. [错误码](#12-错误码)

---

## 1. API 概述

### 1.1 基础信息

**Base URL:**
```
http://localhost:8000/api
```

**协议:** HTTP/HTTPS

**数据格式:** JSON

**字符编码:** UTF-8

### 1.2 通用响应格式

所有 API 响应遵循统一格式:

**成功响应:**
```json
{
  "status": "ok",
  "data": { ... },
  "msg": "操作成功"
}
```

**错误响应:**
```json
{
  "status": "error",
  "msg": "错误描述",
  "code": "错误码"
}
```

### 1.3 通用请求头

```http
Content-Type: application/json
Authorization: Bearer <token>
```

---

## 2. 认证方式

### 2.1 Bearer Token 认证

需要在请求头中携带访问令牌:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**获取 Token:** 通过登录接口或 OAuth 2.0 流程获取

### 2.2 OAuth 2.0 认证

Casdoor 支持 OAuth 2.0 标准认证流程:

- **Authorization Code**: 授权码模式(推荐)
- **Password**: 密码模式
- **Client Credentials**: 客户端凭证模式
- **Token Exchange**: 令牌交换模式

---

## 3. 认证 API

### 3.1 用户登录

登录并获取访问令牌。

**接口地址:**
```http
POST /api/login
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| organization | string | 是 | 组织名称 |
| username | string | 是 | 用户名或邮箱 |
| password | string | 是 | 密码 |
| type | string | 是 | 认证类型,固定值: "password" |
| application | string | 否 | 应用名称,默认: "app-built-in" |
| code | string | 否 | 验证码(MFA 时需要) |
| phonePrefix | string | 否 | 手机号国家代码 |
| autoSignin | boolean | 否 | 是否自动登录,默认: false |

**请求示例:**
```json
{
  "organization": "admin",
  "username": "admin",
  "password": "admin",
  "type": "password"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": {
    "user": {
      "owner": "admin",
      "name": "admin",
      "displayName": "管理员",
      "email": "admin@example.com",
      "phone": "+8613800138000",
      "avatar": "https://example.com/avatar.png",
      "createdTime": "2026-01-01T00:00:00Z",
      "modifiedTime": "2026-01-23T00:00:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
}
```

---

### 3.2 用户注册

创建新用户账号。

**接口地址:**
```http
POST /api/signup
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| organization | string | 是 | 组织名称 |
| username | string | 是 | 用户名(全局唯一) |
| password | string | 是 | 密码 |
| email | string | 条件必填 | 邮箱(应用要求时必填) |
| phone | string | 条件必填 | 手机号(应用要求时必填) |
| displayName | string | 否 | 显示名称 |
| firstName | string | 否 | 名 |
| lastName | string | 否 | 姓 |
| address | array | 否 | 地址信息 |
| affiliation | string | 否 | 所属机构 |
| title | string | 否 | 职位 |
| idCard | string | 否 | 身份证号 |
| homepage | string | 否 | 个人主页 |
| bio | string | 否 | 个人简介 |
| tag | string | 否 | 标签 |
| region | string | 否 | 地区 |
| language | string | 否 | 语言 |
| gender | string | 否 | 性别 |
| birthday | string | 否 | 生日(格式: YYYY-MM-DD) |
| education | string | 否 | 学历 |
| score | int | 否 | 积分 |
| ranking | int | 否 | 排名 |
| isDefaultAvatar | boolean | 否 | 是否默认头像 |
| isOnline | boolean | 否 | 是否在线 |
| isAdmin | boolean | 否 | 是否管理员 |
| isGlobalAdmin | boolean | 否 | 是否全局管理员 |
| isForbidden | boolean | 否 | 是否被禁用 |
| isDeleted | boolean | 否 | 是否被删除 |
| signupApplication | string | 否 | 注册应用名称 |
| hash | string | 否 | 哈希值 |
| preHash | string | 否 | 之前的哈希值 |

**请求示例:**
```json
{
  "organization": "admin",
  "username": "newuser",
  "password": "Password123!",
  "email": "user@example.com",
  "displayName": "新用户",
  "phone": "+8613800138000"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "创建成功",
  "sub": "admin/newuser"
}
```

---

### 3.3 获取用户信息 (OIDC)

根据访问令牌获取用户信息(OIDC 标准)。

**接口地址:**
```http
GET /api/userinfo
```

**请求头:**
```http
Authorization: Bearer <access_token>
```

**响应示例:**
```json
{
  "sub": "admin",
  "name": "Admin",
  "preferred_username": "admin",
  "given_name": "Admin",
  "family_name": "User",
  "email": "admin@example.com",
  "phone_number": "+8613800138000",
  "picture": "http://example.com/avatar.png",
  "address": {
    "country": "CN",
    "region": "Beijing"
  },
  "locale": "zh-CN"
}
```

---

### 3.4 刷新令牌

使用刷新令牌获取新的访问令牌。

**接口地址:**
```http
POST /api/oauth/refresh_token
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| grant_type | string | 是 | 固定值: "refresh_token" |
| refresh_token | string | 是 | 刷新令牌 |
| client_id | string | 是 | 客户端 ID |
| client_secret | string | 是 | 客户端密钥 |

**请求示例:**
```json
{
  "grant_type": "refresh_token",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "client_id": "your-client-id",
  "client_secret": "your-client-secret"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
  }
}
```

---

### 3.5 用户登出

登出并使令牌失效。

**接口地址:**
```http
POST /api/logout
```

**请求头:**
```http
Authorization: Bearer <token>
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| user | string | 否 | 用户标识(格式: owner/name) |
| application | string | 否 | 应用名称 |

**请求示例:**
```json
{
  "user": "admin/admin",
  "application": "app-built-in"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "msg": "登出成功"
}
```

---

## 4. 用户管理 API

### 4.1 获取用户列表

分页获取用户列表。

**接口地址:**
```http
GET /api/get-users
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| page | int | 否 | 页码,默认: 1 |
| pageSize | int | 否 | 每页数量,默认: 10 |
| field | string | 否 | 排序字段 |
| value | string | 否 | 搜索值(模糊匹配) |
| sortField | string | 否 | 排序字段 |
| sortOrder | string | 否 | 排序方向: "asc" 或 "desc" |

**请求示例:**
```http
GET /api/get-users?owner=admin&page=1&pageSize=20&sortField=name&sortOrder=asc
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "admin",
      "displayName": "管理员",
      "email": "admin@example.com",
      "phone": "+8613800138000",
      "avatar": "https://example.com/avatar.png",
      "createdTime": "2026-01-01T00:00:00Z",
      "modifiedTime": "2026-01-23T00:00:00Z",
      "isAdmin": true,
      "isForbidden": false,
      "isDeleted": false
    },
    {
      "owner": "admin",
      "name": "user1",
      "displayName": "用户1",
      "email": "user1@example.com",
      "createdTime": "2026-01-02T00:00:00Z"
    }
  ]
}
```

---

### 4.2 获取用户详情

获取指定用户的详细信息。

**接口地址:**
```http
GET /api/get-user
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | string | 是 | 用户 ID(格式: owner/name) |

**请求示例:**
```http
GET /api/get-user?id=admin/admin
```

**响应示例:**
```json
{
  "status": "ok",
  "data": {
    "owner": "admin",
    "name": "admin",
    "createdTime": "2026-01-01T00:00:00Z",
    "updatedTime": "2026-01-23T12:00:00Z",
    "displayName": "管理员",
    "avatar": "https://example.com/avatar.png",
    "email": "admin@example.com",
    "phone": "+8613800138000",
    "address": [
      "中国",
      "北京市",
      "朝阳区"
    ],
    "affiliation": "示例公司",
    "title": "技术总监",
    "homepage": "https://example.com",
    "bio": "这是个人简介",
    "tag": "vip",
    "region": "北京",
    "language": "zh",
    "gender": "M",
    "birthday": "1990-01-01",
    "education": "博士",
    "score": 100,
    "ranking": 1,
    "isDefaultAvatar": false,
    "isOnline": true,
    "isAdmin": true,
    "isGlobalAdmin": false,
    "isForbidden": false,
    "isDeleted": false,
    "roles": ["admin"],
    "permissions": [
      {
        "name": "Permission 1",
        "displayName": "权限1"
      }
    ]
  }
}
```

---

### 4.3 添加用户

创建新用户。

**接口地址:**
```http
POST /api/add-user
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:** 同 [用户注册](#32-用户注册)

**请求示例:**
```json
{
  "owner": "admin",
  "name": "testuser",
  "password": "Password123!",
  "displayName": "测试用户",
  "email": "test@example.com",
  "phone": "+8613800138000",
  "isAdmin": false
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "admin/testuser"
}
```

---

### 4.4 更新用户

更新用户信息。

**接口地址:**
```http
POST /api/update-user
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:** 同 [用户注册](#32-用户注册),只需提供要更新的字段

**请求示例:**
```json
{
  "owner": "admin",
  "name": "testuser",
  "displayName": "测试用户(已更新)",
  "email": "newemail@example.com",
  "title": "产品经理"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "admin/testuser"
}
```

---

### 4.5 删除用户

删除指定用户。

**接口地址:**
```http
POST /api/delete-user
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| name | string | 是 | 用户名 |

**请求示例:**
```json
{
  "owner": "admin",
  "name": "testuser"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "已删除"
}
```

---

### 4.6 批量删除用户

批量删除多个用户。

**接口地址:**
```http
POST /api/delete-users
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userObjects | array | 是 | 用户对象数组 |

**请求示例:**
```json
{
  "userObjects": [
    {
      "owner": "admin",
      "name": "user1"
    },
    {
      "owner": "admin",
      "name": "user2"
    }
  ]
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "已删除 2 个用户"
}
```

---

### 4.7 修改密码

修改用户密码。

**接口地址:**
```http
POST /api/set-password
```

**请求头:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| userOwner | string | 是 | 用户所属组织 |
| userName | string | 是 | 用户名 |
| oldPassword | string | 是 | 旧密码 |
| newPassword | string | 是 | 新密码 |

**请求示例:**
```json
{
  "userOwner": "admin",
  "userName": "testuser",
  "oldPassword": "oldPassword123",
  "newPassword": "newPassword456"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "msg": "密码修改成功"
}
```

---

## 5. 应用管理 API

### 5.1 获取应用列表

获取指定组织的应用列表。

**接口地址:**
```http
GET /api/get-applications
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**请求示例:**
```http
GET /api/get-applications?owner=admin&page=1&pageSize=10
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "app-built-in",
      "displayName": "内置应用",
      "description": "Casdoor 默认应用",
      "logo": "https://cdn.casbin.org/img/casdoor-logo_1185x256.png",
      "homepage": "https://casdoor.org",
      "organization": "admin",
      "enablePassword": true,
      "enableSignUp": true,
      "enableSigninSession": true,
      "enableCodeSignin": false,
      "enableWebAuthn": true,
      "providers": [],
      "clientSecret": "b14c68d770de44d5ab5bb24e4fb6c4bde5162a99",
      "redirectURIs": ["http://localhost:8080/callback"],
      "grantTypes": ["authorization_code", "password"],
      "tokenFormat": "JWT",
      "signupItems": [],
      "signinHtml": "",
      "signUpHtml": ""
    }
  ]
}
```

---

### 5.2 获取应用详情

获取指定应用的详细信息。

**接口地址:**
```http
GET /api/get-application
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | string | 是 | 应用 ID(格式: owner/name) |

**请求示例:**
```http
GET /api/get-application?id=admin/app-built-in
```

**响应示例:**
```json
{
  "status": "ok",
  "data": {
    "owner": "admin",
    "name": "app-built-in",
    "createdTime": "2026-01-01T00:00:00Z",
    "displayName": "内置应用",
    "logo": "https://cdn.casbin.org/img/casdoor-logo_1185x256.png",
    "homepage": "https://casdoor.org",
    "description": "Casdoor 默认应用",
    "organization": "admin",
    "clientIp": "0.0.0.0",
    "enablePassword": true,
    "enableSignUp": true,
    "enableSigninSession": true,
    "enableCodeSignin": false,
    "enableWebAuthn": true,
    "providers": [
      "provider-client-id",
      "provider-provider-name"
    ],
    "redirectURIs": [
      "http://localhost:8080/callback"
    ],
    "grantTypes": [
      "authorization_code",
      "password"
    ],
    "tokenFormat": "JWT",
    "expireInHours": 24 * 7,
    "refreshExpireInHours": 24 * 30,
    "signupItems": [
      {
        "name": "Username",
        "type": "Username",
        "required": true,
        "rule": "None"
      },
      {
        "name": "Password",
        "type": "Password",
        "required": true,
        "rule": "None"
      },
      {
        "name": "Email",
        "type": "Email",
        "required": false,
        "rule": "None"
      },
      {
        "name": "Phone",
        "type": "Phone",
        "required": false,
        "rule": "None"
      }
    ],
    "signinHtml": "<div class='login-container'>...</div>",
    "signUpHtml": "<div class='signup-container'>...</div>",
    "themeData": {
      "primaryColor": "#1890ff",
      "borderRadius": "4px",
      "fontFamily": "Arial, sans-serif"
    }
  }
}
```

---

### 5.3 添加应用

创建新应用。

**接口地址:**
```http
POST /api/add-application
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| name | string | 是 | 应用名称(全局唯一) |
| displayName | string | 是 | 显示名称 |
| description | string | 否 | 应用描述 |
| logo | string | 否 | 应用 Logo URL |
| homepage | string | 否 | 应用主页 URL |
| organization | string | 是 | 所属组织 |
| redirectURIs | array | 是 | 回调地址列表 |
| grantTypes | array | 是 | 授权类型列表 |
| enablePassword | boolean | 否 | 是否启用密码登录 |
| enableSignUp | boolean | 否 | 是否开放注册 |
| enableCodeSignin | boolean | 否 | 是否启用验证码登录 |
| enableWebAuthn | boolean | 否 | 是否启用 WebAuthn |

**请求示例:**
```json
{
  "owner": "admin",
  "name": "my-app",
  "displayName": "我的应用",
  "description": "这是一个示例应用",
  "organization": "admin",
  "redirectURIs": [
    "http://localhost:8080/callback"
  ],
  "grantTypes": [
    "authorization_code",
    "password"
  ],
  "enablePassword": true,
  "enableSignUp": true,
  "enableWebAuthn": false
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "admin/my-app"
}
```

---

### 5.4 更新应用

更新应用信息。

**接口地址:**
```http
POST /api/update-application
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:** 同 [添加应用](#53-添加应用)

**请求示例:**
```json
{
  "owner": "admin",
  "name": "my-app",
  "displayName": "我的应用(已更新)",
  "description": "更新后的描述",
  "redirectURIs": [
    "http://localhost:8080/callback",
    "https://example.com/callback"
  ]
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "admin/my-app"
}
```

---

### 5.5 删除应用

删除指定应用。

**接口地址:**
```http
POST /api/delete-application
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| name | string | 是 | 应用名称 |

**请求示例:**
```json
{
  "owner": "admin",
  "name": "my-app"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "已删除"
}
```

---

## 6. 组织管理 API

### 6.1 获取组织列表

获取组织列表。

**接口地址:**
```http
GET /api/get-organizations
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 否 | 所有者(可选) |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**请求示例:**
```http
GET /api/get-organizations?page=1&pageSize=10
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "admin",
      "displayName": "默认组织",
      "website": "https://casdoor.org",
      "favicon": "https://cdn.casbin.org/img/casdoor-logo_1185x256.png",
      "createdTime": "2026-01-01T00:00:00Z",
      "passwordType": "plain",
      "phonePrefix": "86",
      "defaultAvatar": "https://casbin.org/img/casbin.svg",
      "masterPassword": "",
      "enableSoftDeletion": false,
      "isProfilePublic": true
    }
  ]
}
```

---

### 6.2 获取组织详情

获取指定组织的详细信息。

**接口地址:**
```http
GET /api/get-organization
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | string | 是 | 组织 ID(格式: owner/name) |

**请求示例:**
```http
GET /api/get-organization?id=admin/admin
```

**响应示例:**
```json
{
  "status": "ok",
  "data": {
    "owner": "admin",
    "name": "admin",
    "createdTime": "2026-01-01T00:00:00Z",
    "displayName": "默认组织",
    "website": "https://casdoor.org",
    "favicon": "https://cdn.casbin.org/img/casdoor-logo_1185x256.png",
    "passwordType": "plain",
    "phonePrefix": "86",
    "defaultAvatar": "https://casbin.org/img/casbin.svg",
    "masterPassword": "",
    "enableSoftDeletion": false,
    "isProfilePublic": true
  }
}
```

---

### 6.3 添加组织

创建新组织。

**接口地址:**
```http
POST /api/add-organization
```

**请求头:**
```http
Authorization: Bearer <admin_token>
Content-Type: application/json
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 所有者 |
| name | string | 是 | 组织名称(全局唯一) |
| displayName | string | 是 | 显示名称 |
| website | string | 否 | 网站 URL |
| favicon | string | 否 | Favicon URL |
| passwordType | string | 否 | 密码类型: plain/bcrypt/argon2id |
| phonePrefix | string | 否 | 手机号国家代码 |
| defaultAvatar | string | 否 | 默认头像 URL |

**请求示例:**
```json
{
  "owner": "admin",
  "name": "my-org",
  "displayName": "我的组织",
  "website": "https://example.com",
  "passwordType": "bcrypt",
  "phonePrefix": "86"
}
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "admin/my-org"
}
```

---

## 7. 角色权限 API

### 7.1 获取角色列表

获取角色列表。

**接口地址:**
```http
GET /api/get-roles
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**请求示例:**
```http
GET /api/get-roles?owner=admin&page=1&pageSize=10
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "admin",
      "displayName": "管理员",
      "description": "系统管理员角色",
      "createdTime": "2026-01-01T00:00:00Z",
      "users": ["admin"]
    }
  ]
}
```

---

### 7.2 获取权限列表

获取权限列表。

**接口地址:**
```http
GET /api/get-permissions
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**请求示例:**
```http
GET /api/get-permissions?owner=admin&page=1&pageSize=10
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "permission-1",
      "displayName": "权限1",
      "description": "示例权限",
      "createdTime": "2026-01-01T00:00:00Z",
      "effect": "allow",
      "users": ["admin"],
      "roles": ["admin"],
      "resourceType": "Application",
      "resource": "app-built-in",
      "action": "Read",
      "isEnabled": true
    }
  ]
}
```

---

## 8. 资源管理 API

### 8.1 上传文件

上传文件到 Casdoor。

**接口地址:**
```http
POST /api/upload-resource
```

**请求头:**
```http
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| file | file | 是 | 要上传的文件 |
| owner | string | 是 | 组织名称 |
| application | string | 否 | 应用名称 |
| tag | string | 否 | 文件标签 |

**请求示例 (cURL):**
```bash
curl -X POST http://localhost:8000/api/upload-resource \
  -H "Authorization: Bearer <token>" \
  -F "file=@/path/to/file.jpg" \
  -F "owner=admin" \
  -F "tag=avatar"
```

**响应示例:**
```json
{
  "status": "ok",
  "data": "https://cdn.example.com/files/file.jpg"
}
```

---

## 9. 提供商 API

### 9.1 获取提供商列表

获取可用的身份提供商列表。

**接口地址:**
```http
GET /api/get-providers
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| owner | string | 是 | 组织名称 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页数量 |

**请求示例:**
```http
GET /api/get-providers?owner=admin&page=1&pageSize=10
```

**响应示例:**
```json
{
  "status": "ok",
  "data": [
    {
      "owner": "admin",
      "name": "provider-github",
      "displayName": "GitHub",
      "category": "OAuth",
      "type": "GitHub",
      "clientId": "your-github-client-id",
      "clientSecret": "******",
      "host": "https://github.com",
      "port": 443,
      "createdTime": "2026-01-01T00:00:00Z"
    },
    {
      "owner": "admin",
      "name": "provider-google",
      "displayName": "Google",
      "category": "OAuth",
      "type": "Google",
      "clientId": "your-google-client-id",
      "clientSecret": "******"
    }
  ]
}
```

---

## 10. 令牌 API

### 10.1 OAuth 授权

OAuth 2.0 授权端点。

**接口地址:**
```http
GET /api/oauth/authorize
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| client_id | string | 是 | 客户端 ID |
| response_type | string | 是 | 响应类型: code/token |
| redirect_uri | string | 是 | 回调地址 |
| scope | string | 否 | 授权范围 |
| state | string | 是 | 状态参数(防 CSRF) |
| nonce | string | 否 | 随机数(ID token 时需要) |

**请求示例:**
```http
GET /api/oauth/authorize?
    client_id=your-client-id&
    redirect_uri=http://localhost:8080/callback&
    response_type=code&
    scope=openid profile&
    state=random-state-123
```

**响应:**
重定向到 redirect_uri，携带授权码

---

### 10.2 获取访问令牌

用授权码换取访问令牌。

**接口地址:**
```http
POST /api/oauth/token
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| grant_type | string | 是 | 授权类型: authorization_code/password |
| code | string | 条件必填 | 授权码(authorization_code 时必填) |
| client_id | string | 是 | 客户端 ID |
| client_secret | string | 是 | 客户端密钥 |
| redirect_uri | string | 条件必填 | 回调地址(需与授权时一致) |
| username | string | 条件必填 | 用户名(password 时必填) |
| password | string | 条件必填 | 密码(password 时必填) |
| scope | string | 否 | 授权范围 |

**请求示例 (Authorization Code):**
```json
{
  "grant_type": "authorization_code",
  "client_id": "your-client-id",
  "client_secret": "your-client-secret",
  "code": "authorization-code-from-previous-step",
  "redirect_uri": "http://localhost:8080/callback"
}
```

**请求示例 (Password):**
```json
{
  "grant_type": "password",
  "client_id": "your-client-id",
  "client_secret": "your-client-secret",
  "username": "admin",
  "password": "admin",
  "scope": "openid profile"
}
```

**响应示例:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "scope": "openid profile"
}
```

---

### 10.3 验证令牌

验证访问令牌是否有效。

**接口地址:**
```http
POST /api/oauth/introspect
```

**请求参数:**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| token | string | 是 | 访问令牌 |
| client_id | string | 是 | 客户端 ID |
| client_secret | string | 是 | 客户端密钥 |

**请求示例:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "client_id": "your-client-id",
  "client_secret": "your-client-secret"
}
```

**响应示例:**
```json
{
  "active": true,
  "client_id": "your-client-id",
  "username": "admin",
  "scope": "openid profile",
  "exp": 1706055600
}
```

---

## 11. SDK 使用

### 11.1 Go SDK

**安装:**
```bash
go get github.com/casdoor/casdoor-go-sdk
```

**初始化:**
```go
package main

import (
    "fmt"
    "github.com/casdoor/casdoor-go-sdk/casdoorsdk"
)

func main() {
    // 初始化 SDK
    casdoorsdk.InitConfig("http://localhost:8000",
        "admin",
        "admin",
        "admin",
        "b14c68d770de44d5ab5bb24e4fb6c4bde5162a99")

    // 获取用户信息
    user, err := casdoorsdk.GetUser("admin/admin")
    if err != nil {
        panic(err)
    }
    fmt.Printf("用户: %s\n", user.DisplayName)

    // 获取用户列表
    users, err := casdoorsdk.GetUsers("admin")
    if err != nil {
        panic(err)
    }
    fmt.Printf("用户数: %d\n", len(users))

    // 创建用户
    newUser := &casdoorsdk.User{
        Owner:       "admin",
        Name:        "newuser",
        DisplayName: "新用户",
        Email:       "user@example.com",
        Type:        "normal-user",
    }
    affectedRows, err := casdoorsdk.AddUser(newUser)
    if err != nil {
        panic(err)
    }
    fmt.Printf("影响行数: %d\n", affectedRows)
}
```

---

### 11.2 JavaScript SDK

**安装:**
```bash
npm install casdoor-js-sdk
# 或
yarn add casdoor-js-sdk
```

**使用:**
```javascript
import CasdoorSDK from 'casdoor-js-sdk';

// 初始化 SDK
const sdk = new CasdoorSDK({
  serverUrl: 'http://localhost:8000',
  clientId: 'your-client-id',
  appName: 'app-built-in',
  organizationName: 'admin',
  redirectPath: '/callback',
});

// 1. 获取登录 URL
function login() {
  const url = sdk.getSigninUrl();
  window.location.href = url;
}

// 2. 处理回调
async function handleCallback() {
  const code = new URLSearchParams(window.location.search).get('code');
  if (code) {
    try {
      // 换取 token
      await sdk.exchangeForToken();
      console.log('登录成功');

      // 获取用户信息
      const user = await sdk.getUserInfo();
      console.log('用户信息:', user);

      // 保存 token
      localStorage.setItem('token', sdk.token);

      // 跳转到主页
      window.location.href = '/';
    } catch (error) {
      console.error('登录失败:', error);
    }
  }
}

// 3. 调用 API
async function callApi() {
  const token = localStorage.getItem('token');
  const response = await fetch('/api/user/profile', {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });
  return await response.json();
}

// 4. 登出
function logout() {
  sdk.logout();
  localStorage.removeItem('token');
  window.location.href = '/login';
}
```

---

### 11.3 Python SDK

**安装:**
```bash
pip install casdoor
```

**使用:**
```python
from casdoor import CasdoorSDK

# 初始化 SDK
sdk = CasdoorSDK(
    endpoint="http://localhost:8000",
    client_id="your-client-id",
    client_secret="your-client-secret",
    certificate="your-certificate",
    org_name="admin",
    application_name="app-built-in"
)

# 获取用户信息
user = sdk.get_user("admin/admin")
print(f"用户: {user.display_name}")

# 获取用户列表
users = sdk.get_users("admin")
print(f"用户数: {len(users)}")

# 创建用户
from casdoor import User

new_user = User(
    owner="admin",
    name="newuser",
    display_name="新用户",
    email="user@example.com",
    type="normal-user"
)
sdk.add_user(new_user)

# 删除用户
sdk.delete_user(new_user)
```

---

### 11.4 Java SDK

**Maven 依赖:**
```xml
<dependency>
    <groupId>org.casbin</groupId>
    <artifactId>casdoor-java-sdk</artifactId>
    <version>1.x.x</version>
</dependency>
```

**使用:**
```java
import org.casbin.casdoor.sdk.CasdoorSDK;

public class CasdoorExample {
    public static void main(String[] args) {
        // 初始化 SDK
        CasdoorSDK sdk = new CasdoorSDK(
            "http://localhost:8000",
            "your-client-id",
            "your-client-secret",
            "your-certificate",
            "admin",
            "app-built-in"
        );

        // 获取用户信息
        User user = sdk.getUser("admin/admin");
        System.out.println("用户: " + user.getDisplayName());

        // 获取用户列表
        List<User> users = sdk.getUsers("admin");
        System.out.println("用户数: " + users.size());

        // 创建用户
        User newUser = new User();
        newUser.setOwner("admin");
        newUser.setName("newuser");
        newUser.setDisplayName("新用户");
        newUser.setEmail("user@example.com");
        sdk.addUser(newUser);

        // 删除用户
        sdk.deleteUser(newUser);
    }
}
```

---

### 11.5 PHP SDK

**安装:**
```bash
composer require casdoor/casdoor-php-sdk
```

**使用:**
```php
<?php
require 'vendor/autoload.php';

use Casdoor\Auth\User;
use Casdoor\Casdoor;

// 初始化 SDK
$casdoor = new Casdoor(
    "http://localhost:8000",
    "client-id",
    "client-secret",
    "certificate",
    "admin",
    "app-built-in"
);

// 获取用户信息
$user = $casdoor->getUser("admin/admin");
echo "用户: " . $user->displayName . "\n";

// 获取用户列表
$users = $casdoor->getUsers("admin");
echo "用户数: " . count($users) . "\n";

// 创建用户
$newUser = new User();
$newUser->owner = "admin";
$newUser->name = "newuser";
$newUser->displayName = "新用户";
$newUser->email = "user@example.com";
$casdoor->addUser($newUser);

// 删除用户
$casdoor->deleteUser($newUser);
?>
```

---

## 12. 错误码

### 12.1 错误响应格式

```json
{
  "status": "error",
  "msg": "错误描述信息",
  "code": "ERROR_CODE"
}
```

### 12.2 常见错误码

| 错误码 | 说明 |
|--------|------|
| **BadRequest** | 请求参数错误 |
| **Unauthorized** | 未授权或令牌无效 |
| **Forbidden** | 无权限访问 |
| **NotFound** | 资源不存在 |
| **UserNotFound** | 用户不存在 |
| **ApplicationNotFound** | 应用不存在 |
| **OrganizationNotFound** | 组织不存在 |
| **InvalidCredential** | 用户名或密码错误 |
| **DuplicateUser** | 用户已存在 |
| **DuplicateOrganization** | 组织已存在 |
| **DuplicateApplication** | 应用已存在 |
| **InvalidToken** | 令牌无效或已过期 |
| **MissingParameter** | 缺少必要参数 |
| **InvalidParameter** | 参数格式错误 |
| **EmailNotFound** | 邮箱不存在 |
| **PhoneNotFound** | 手机号不存在 |
| **VerificationCodeFailed** | 验证码错误 |
| **VerificationCodeExpired** | 验证码已过期 |
| **MFARequired** | 需要多因素认证 |
| **MFACodeFailed** | MFA 验证码错误 |
| **PasswordTooWeak** | 密码强度不足 |
| **PasswordSameAsOld** | 新密码不能与旧密码相同 |
| **OAuthError** | OAuth 认证错误 |
| **OAuthInvalidClient** | OAuth 客户端 ID 或密钥错误 |
| **OAuthInvalidCode** | OAuth 授权码无效 |
| **OAuthInvalidRedirectURI** | OAuth 回调地址不匹配 |
| **OAuthExpiredToken** | OAuth 令牌已过期 |
| **DatabaseError** | 数据库错误 |
| **NetworkError** | 网络错误 |
| **ServerError** | 服务器内部错误 |

### 12.3 错误处理示例

**JavaScript:**
```javascript
async function handleApiCall() {
  try {
    const response = await fetch('/api/get-user?id=admin/admin');
    const result = await response.json();

    if (result.status === 'ok') {
      console.log('成功:', result.data);
    } else {
      console.error('错误:', result.msg, result.code);
      handleError(result.code);
    }
  } catch (error) {
    console.error('请求失败:', error);
  }
}

function handleError(code) {
  switch (code) {
    case 'Unauthorized':
      // 跳转到登录页
      window.location.href = '/login';
      break;
    case 'Forbidden':
      alert('您没有权限访问此资源');
      break;
    case 'InvalidToken':
      // 清除令牌并跳转登录
      localStorage.removeItem('token');
      window.location.href = '/login';
      break;
    default:
      alert('操作失败: ' + code);
  }
}
```

**Go:**
```go
package main

import (
    "fmt"
    "github.com/casdoor/casdoor-go-sdk/casdoorsdk"
)

func handleApiError(err error) {
    if err == nil {
        return
    }

    // 类型断言获取详细错误
    if casdoorErr, ok := err.(*casdoorsdk.Error); ok {
        fmt.Printf("错误码: %s, 错误信息: %s\n",
            casdoorErr.Code, casdoorErr.Message)

        switch casdoorErr.Code {
        case "Unauthorized":
            fmt.Println("未授权,需要重新登录")
        case "Forbidden":
            fmt.Println("无权限访问")
        case "InvalidToken":
            fmt.Println("令牌无效,需要刷新")
        default:
            fmt.Printf("未知错误: %s\n", casdoorErr.Code)
        }
    } else {
        fmt.Printf("请求失败: %v\n", err)
    }
}
```

---

## 附录

### A. OAuth 2.0 作用域 (Scopes)

| Scope | 说明 |
|-------|------|
| **openid** | 启用 OIDC 流程 |
| **profile** | 获取用户基本信息 |
| **email** | 获取用户邮箱 |
| **phone** | 获取用户手机号 |
| **address** | 获取用户地址 |
| **roles** | 获取用户角色 |

### B. 令牌格式

Casdoor 使用 JWT (JSON Web Token) 格式:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyIjoiYWRtaW4iLCJvcmdhbml6YXRpb24iOiJhZG1pbiIsImV4cCI6MTcwNjA1NTYwMH0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**JWT 包含三个部分:**
1. **Header**: 算法和令牌类型
2. **Payload**: 用户信息和权限
3. **Signature**: 签名(验证令牌完整性)

### C. 相关资源

- **官方文档**: https://casdoor.org/docs
- **API 文档**: https://casdoor.org/docs/api/overview
- **GitHub**: https://github.com/casdoor/casdoor
- **SDK 仓库**: https://github.com/casdoor/sdks

---

**文档版本**: v1.0
**更新时间**: 2026-01-23
**适用于 Casdoor 版本**: v1.x

> 本文档提供完整的 API 参考,包括所有端点、参数、响应格式和错误码。
