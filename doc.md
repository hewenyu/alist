# AList API 文档

AList 是一个支持多存储的文件列表程序，使用 Gin 和 Solidjs 开发。本文档列出了所有可用的 API 端点。

## 基础信息

- 基础路径: 配置的 URL 路径（默认为 `/`）
- 所有 API 请求都需要在基础路径后添加 `/api`
- 认证方式: JWT Token（在登录后获取）
- 请求头: 
  - `Authorization`: Bearer {token} (需要认证的接口)
  - `Content-Type`: application/json

## 公共接口

### 健康检查
- `GET /ping`
  - 描述: 检查服务是否正常运行
  - 响应: `pong`

### 认证相关

#### 登录
- `POST /api/auth/login`
  - 描述: 用户登录
  - 请求体:
    ```json
    {
      "username": "string", // 必填，用户名
      "password": "string", // 必填，密码
      "otp_code": "string" // 可选，二次认证码
    }
    ```
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": {
        "token": "string" // JWT token
      }
    }
    ```

#### SSO 登录
- `GET /api/auth/sso`
  - 描述: SSO 登录重定向
- `GET /api/auth/sso_callback`
  - 描述: SSO 登录回调
- `GET /api/auth/get_sso_id`
  - 描述: 获取 SSO ID
- `GET /api/auth/sso_get_token`
  - 描述: 获取 SSO Token

### 公共设置
- `ANY /api/public/settings`
  - 描述: 获取公共设置
- `ANY /api/public/offline_download_tools`
  - 描述: 获取离线下载工具列表
- `ANY /api/public/archive_extensions`
  - 描述: 获取支持的压缩文件扩展名

## 需要认证的接口

### 用户相关

#### 个人信息
- `GET /api/me`
  - 描述: 获取当前用户信息
  - 请求头: 需要 Authorization
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": {
        "username": "string",
        "role": 0,
        "otp": false,
        "base_path": "string",
        "sso_id": "string"
      }
    }
    ```

#### SSH 密钥管理
- `GET /api/me/sshkey/list`
  - 描述: 列出个人 SSH 公钥
- `POST /api/me/sshkey/add`
  - 描述: 添加 SSH 公钥
- `POST /api/me/sshkey/delete`
  - 描述: 删除 SSH 公钥

#### 二次认证
- `POST /api/auth/2fa/generate`
  - 描述: 生成二次认证
  - 请求头: 需要 Authorization
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": {
        "qr": "string", // 二维码图片 base64
        "secret": "string" // 密钥
      }
    }
    ```

- `POST /api/auth/2fa/verify`
  - 描述: 验证二次认证
  - 请求头: 需要 Authorization
  - 请求体:
    ```json
    {
      "code": "string", // 必填，验证码
      "secret": "string" // 必填，密钥
    }
    ```

#### WebAuthn
- `GET /api/authn/webauthn_begin_registration`
  - 描述: 开始 WebAuthn 注册
- `POST /api/authn/webauthn_finish_registration`
  - 描述: 完成 WebAuthn 注册
- `GET /api/authn/webauthn_begin_login`
  - 描述: 开始 WebAuthn 登录
- `POST /api/authn/webauthn_finish_login`
  - 描述: 完成 WebAuthn 登录
- `POST /api/authn/delete_authn`
  - 描述: 删除 WebAuthn 登录
- `GET /api/authn/getcredentials`
  - 描述: 获取 WebAuthn 凭证

### 文件系统操作

#### 基本操作
- `ANY /api/fs/list`
  - 描述: 列出目录内容
  - 请求参数:
    ```json
    {
      "path": "string", // 必填，路径
      "password": "string", // 可选，加密目录的密码
      "page": 1, // 可选，页码，默认 1
      "per_page": 20, // 可选，每页数量，默认 20
      "refresh": false // 可选，是否刷新缓存
    }
    ```
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": {
        "content": [
          {
            "name": "string",
            "size": 0,
            "is_dir": false,
            "modified": "string",
            "created": "string",
            "sign": "string",
            "thumb": "string",
            "type": 0,
            "hashinfo": "string",
            "hash_info": {}
          }
        ],
        "total": 0,
        "readme": "string",
        "header": "string",
        "write": false,
        "provider": "string"
      }
    }
    ```

#### 搜索文件
- `ANY /api/fs/search`
  - 描述: 搜索文件

#### 获取文件信息
- `ANY /api/fs/get`
  - 描述: 获取单个文件或目录的详细信息
  - 请求参数:
    ```json
    {
      "path": "string", // 必填，文件路径
      "password": "string" // 可选，加密目录的密码
    }
    ```
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": {
        "name": "string",
        "size": 0,
        "is_dir": false,
        "modified": "string",
        "created": "string",
        "sign": "string",
        "thumb": "string",
        "type": 0,
        "raw_url": "string",
        "readme": "string",
        "header": "string",
        "provider": "string",
        "related": []
      }
    }
    ```

#### 其他文件操作
- `ANY /api/fs/other`
  - 描述: 其他文件操作

#### 获取目录列表
- `ANY /api/fs/dirs`
  - 描述: 获取目录列表

#### 文件管理
- `POST /api/fs/mkdir`
  - 描述: 创建目录
- `POST /api/fs/rename`
  - 描述: 重命名文件/目录
- `POST /api/fs/batch_rename`
  - 描述: 批量重命名
- `POST /api/fs/regex_rename`
  - 描述: 正则表达式重命名
- `POST /api/fs/move`
  - 描述: 移动文件/目录
- `POST /api/fs/recursive_move`
  - 描述: 递归移动目录
- `POST /api/fs/copy`
  - 描述: 复制文件/目录
- `POST /api/fs/remove`
  - 描述: 删除文件/目录
- `POST /api/fs/remove_empty_directory`
  - 描述: 删除空目录

#### 上传下载
- `PUT /api/fs/put`
  - 描述: 流式上传文件
  - 请求头: 
    - `Authorization`: Bearer {token}
    - `Content-Type`: application/octet-stream
    - `File-Path`: 文件路径（URL 编码）
  - 请求体: 文件二进制内容

- `PUT /api/fs/form`
  - 描述: 表单上传文件
  - 请求头: 
    - `Authorization`: Bearer {token}
    - `Content-Type`: multipart/form-data
  - 表单参数:
    - `file`: 文件
    - `path`: 文件路径

#### 压缩文件操作
- `ANY /api/fs/archive/meta`
  - 描述: 获取压缩文件元信息
- `ANY /api/fs/archive/list`
  - 描述: 列出压缩文件内容
- `POST /api/fs/archive/decompress`
  - 描述: 解压文件

### 管理员接口

#### 元数据管理
- `GET /api/admin/meta/list`
  - 描述: 列出元数据
- `GET /api/admin/meta/get`
  - 描述: 获取元数据
- `POST /api/admin/meta/create`
  - 描述: 创建元数据
- `POST /api/admin/meta/update`
  - 描述: 更新元数据
- `POST /api/admin/meta/delete`
  - 描述: 删除元数据

#### 用户管理
- `GET /api/admin/user/list`
  - 描述: 列出所有用户
  - 请求头: 需要管理员权限
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": [
        {
          "username": "string",
          "role": 0,
          "base_path": "string",
          "sso_id": "string"
        }
      ]
    }
    ```

- `GET /api/admin/user/get`
  - 描述: 获取用户信息
- `POST /api/admin/user/create`
  - 描述: 创建新用户
  - 请求头: 需要管理员权限
  - 请求体:
    ```json
    {
      "username": "string",
      "password": "string",
      "role": 0,
      "base_path": "string"
    }
    ```

- `POST /api/admin/user/update`
  - 描述: 更新用户
- `POST /api/admin/user/cancel_2fa`
  - 描述: 取消用户的二次认证
- `POST /api/admin/user/delete`
  - 描述: 删除用户
- `POST /api/admin/user/del_cache`
  - 描述: 删除用户缓存

#### 存储管理
- `GET /api/admin/storage/list`
  - 描述: 列出所有存储
  - 请求头: 需要管理员权限
  - 响应:
    ```json
    {
      "code": 200,
      "message": "success",
      "data": [
        {
          "id": 0,
          "mount_path": "string",
          "order": 0,
          "driver": "string",
          "status": "string",
          "addition": "string"
        }
      ]
    }
    ```

- `GET /api/admin/storage/get`
  - 描述: 获取存储信息
- `POST /api/admin/storage/create`
  - 描述: 创建新存储
  - 请求头: 需要管理员权限
  - 请求体:
    ```json
    {
      "mount_path": "string",
      "order": 0,
      "driver": "string",
      "status": "string",
      "addition": {}
    }
    ```

- `POST /api/admin/storage/update`
  - 描述: 更新存储
- `POST /api/admin/storage/delete`
  - 描述: 删除存储
- `POST /api/admin/storage/enable`
  - 描述: 启用存储
- `POST /api/admin/storage/disable`
  - 描述: 禁用存储
- `POST /api/admin/storage/load_all`
  - 描述: 加载所有存储

#### 驱动管理
- `GET /api/admin/driver/list`
  - 描述: 列出驱动信息
- `GET /api/admin/driver/names`
  - 描述: 列出驱动名称
- `GET /api/admin/driver/info`
  - 描述: 获取驱动信息

#### 设置管理
- `GET /api/admin/setting/get`
  - 描述: 获取设置
- `GET /api/admin/setting/list`
  - 描述: 列出设置
- `POST /api/admin/setting/save`
  - 描述: 保存设置
- `POST /api/admin/setting/delete`
  - 描述: 删除设置
- `POST /api/admin/setting/reset_token`
  - 描述: 重置 Token

#### 下载工具设置
- `POST /api/admin/setting/set_aria2`
  - 描述: 设置 Aria2
- `POST /api/admin/setting/set_qbit`
  - 描述: 设置 qBittorrent
- `POST /api/admin/setting/set_transmission`
  - 描述: 设置 Transmission
- `POST /api/admin/setting/set_115`
  - 描述: 设置 115 网盘
- `POST /api/admin/setting/set_pikpak`
  - 描述: 设置 PikPak
- `POST /api/admin/setting/set_thunder`
  - 描述: 设置迅雷

#### 索引管理
- `POST /api/admin/index/build`
  - 描述: 构建索引
- `POST /api/admin/index/update`
  - 描述: 更新索引
- `POST /api/admin/index/stop`
  - 描述: 停止索引
- `POST /api/admin/index/clear`
  - 描述: 清除索引
- `GET /api/admin/index/progress`
  - 描述: 获取索引进度

### WebDAV 支持
- 基础路径: `/dav`
  - 描述: WebDAV 协议支持，可用于第三方 WebDAV 客户端访问

### S3 协议支持
- 基础路径: `/s3`
  - 描述: S3 协议支持，可用于 S3 客户端访问

## 错误码说明

- 200: 成功
- 400: 请求参数错误
- 401: 未认证
- 403: 无权限
- 404: 资源不存在
- 500: 服务器内部错误

## 注意事项

1. 所有需要认证的接口都需要在请求头中携带 JWT Token
2. 管理员接口需要管理员权限
3. 上传和下载接口都有速率限制
4. 部分接口支持异步操作，需要通过任务系统查看进度
5. 文件路径使用 URL 编码，特殊字符需要转义
6. 时间格式使用 ISO 8601 标准
7. 文件大小单位为字节

# 错误码说明

- 200: 成功
- 400: 请求参数错误
- 401: 未认证
- 403: 无权限
- 404: 资源不存在
- 500: 服务器内部错误 