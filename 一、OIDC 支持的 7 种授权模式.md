OIDC授权模式说明

## 一、OIDC 支持的 7 种授权模式

### 1. **授权码模式** (Authorization Code Flow) - **最安全，推荐**

### 2. **混合模式** (Hybrid Flow) - **SPA推荐**

### 3. **隐式模式** (Implicit Flow) - **已废弃，不推荐**

### 4. **密码模式** (Resource Owner Password Credentials)

### 5. **客户端凭证模式** (Client Credentials Flow)

### 6. **设备授权模式** (Device Flow)

### 7. **刷新令牌模式** (Refresh Token Flow)

------

## 二、详细流程图解

### 1. **授权码模式 (Authorization Code Flow)**

**特点**：最安全，适用于 Web 应用，支持 PKCE



```markdown
sequenceDiagram
    participant User as 用户/浏览器
    participant Client as 客户端应用
    participant Auth as 授权服务器
    participant Resource as 资源服务器

    Note over User,Resource: 第一步：获取授权码
    User->>Client: 1. 访问应用
    Client->>User: 2. 重定向到授权端点
    User->>Auth: 3. 用户登录并授权
    Auth->>User: 4. 重定向回应用 (带code)
    User->>Client: 5. 传递code给客户端

    Note over Client,Resource: 第二步：令牌交换（后端）
    Client->>Auth: 6. POST /token (code + client_secret)
    Auth->>Client: 7. 返回令牌 (id_token, access_token, refresh_token)

    Note over Client,Resource: 第三步：访问资源
    Client->>Resource: 8. 使用access_token访问API
    Resource->>Client: 9. 返回受保护资源

    Note over Client,Auth: 第四步：令牌刷新
    Client->>Auth: 10. POST /token (refresh_token)
    Auth->>Client: 11. 返回新access_token
```

**PKCE 增强版流程**：



```markdown
flowchart TD
    A[客户端生成 code_verifier] --> B[计算 code_challenge]
    B --> C[授权请求包含 code_challenge]
    C --> D[授权服务器存储 challenge]
    D --> E[返回授权码]
    E --> F[令牌请求包含 code_verifier]
    F --> G[服务器验证 verifier 匹配 challenge]
    G --> H[颁发令牌]
```



------

### 2. **混合模式 (Hybrid Flow)**

**特点**：ID Token 立即返回，Access Token 后端获取



```markdown
sequenceDiagram
    participant User as 用户/浏览器
    participant Client as 客户端
    participant Auth as 授权服务器
    participant API as 资源服务器

    Note over User,Auth: 第一阶段：前端授权
    User->>Client: 1. 访问应用
    Client->>User: 2. 重定向到/authorize<br/>response_type=code id_token
    User->>Auth: 3. 用户登录授权
    Auth->>User: 4. 302重定向回应用<br/>携带: code + id_token
    User->>Client: 5. 前端收到code和id_token

    Note over Client,Auth: 第二阶段：前端验证+后端交换
    Client->>Client: 6. 前端验证id_token<br/>显示用户信息
    Client->>Auth: 7. 后端发送code + client_secret<br/>POST /token
    Auth->>Client: 8. 返回完整令牌集<br/>access_token + refresh_token

    Note over Client,API: 第三阶段：访问资源
    Client->>API: 9. 使用access_token调用API
    API->>Client: 10. 返回数据
```

**数据流向**：

text

```
前端通道（不安全）：
授权服务器 → 浏览器 → 前端
携带：code(授权码) + id_token(JWT)

后端通道（安全）：
前端 → 后端 → 授权服务器
携带：code + client_secret
返回：access_token + refresh_token
```



------

### 3. **隐式模式 (Implicit Flow) - 已废弃**

**特点**：令牌直接返回给前端，不安全





**现代替代方案**：

text

```
✅ 使用授权码模式 + PKCE
✅ 或混合模式
```



------

### 4. **密码模式 (Resource Owner Password Credentials)**

**特点**：用户直接提供用户名密码，适用于受信任的客户端





**使用场景限制**：

text

```
✅ 适用场景：
- 第一方官方应用
- 内部企业应用
- 高度信任的环境

❌ 不适用：
- 第三方应用
- 公开的客户端应用
- 安全性要求高的场景
```



------

### 5. **客户端凭证模式 (Client Credentials Flow)**

**特点**：服务器到服务器，没有用户参与





**令牌内容**：

json

```
{
  "access_token": "服务令牌",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "api:read api:write"
  // 注意：没有id_token，没有refresh_token
}
```



------

### 6. **设备授权模式 (Device Flow)**

**特点**：适用于输入受限的设备（IoT、电视、命令行工具）





**典型应用**：

text

```
📺 智能电视登录：
1. 电视显示：请在手机访问 xx.com，输入 1234-5678
2. 用户手机授权
3. 电视自动登录

🖥️ 命令行工具：
1. 工具显示：请访问 http://localhost:8080
2. 浏览器打开完成授权
3. 工具获得令牌
```



------

### 7. **刷新令牌模式 (Refresh Token Flow)**

**特点**：不独立的授权流程，是其他流程的扩展





**刷新策略**：

csharp

```
// C# 刷新令牌实现
public async Task<string> RefreshAccessToken()
{
    var refreshToken = await _secureStorage.GetAsync("refresh_token");
    
    var response = await _httpClient.RequestRefreshTokenAsync(new RefreshTokenRequest
    {
        Address = "https://auth.example.com/token",
        ClientId = _clientId,
        ClientSecret = _clientSecret,
        RefreshToken = refreshToken,
        
        // 可选：限制新令牌的范围
        Scope = "openid profile email api.read"
    });
    
    if (!response.IsError)
    {
        // 更新存储的令牌
        await _secureStorage.SetAsync("access_token", response.AccessToken);
        
        // 如果返回了新refresh_token，更新
        if (!string.IsNullOrEmpty(response.RefreshToken))
        {
            await _secureStorage.SetAsync("refresh_token", response.RefreshToken);
        }
        
        return response.AccessToken;
    }
    else
    {
        // 刷新失败，需要重新登录
        await Logout();
        throw new Exception("令牌刷新失败");
    }
}
```



------

## 三、模式选择决策图





------

## 四、各模式对比总结表

| 模式           | 响应类型                                           | 令牌位置   | 安全性 | 适用场景          | 现代推荐   |
| :------------- | :------------------------------------------------- | :--------- | :----- | :---------------- | :--------- |
| **授权码**     | `code`                                             | 后端获取   | ⭐⭐⭐⭐⭐  | Web应用、移动应用 | ✅ 推荐     |
| **混合**       | `code id_token` `code token` `code id_token token` | 前后端分离 | ⭐⭐⭐⭐   | 单页应用SPA       | ✅ 推荐     |
| **隐式**       | `id_token` `id_token token` `token`                | 前端直接   | ⭐      | 历史遗留          | ❌ 已废弃   |
| **密码**       | `password`                                         | 直接返回   | ⭐⭐     | 内部信任应用      | ⚠️ 谨慎使用 |
| **客户端凭证** | `client_credentials`                               | 服务端     | ⭐⭐⭐⭐   | 服务间调用        | ✅ 推荐     |
| **设备授权**   | `device_code`                                      | 设备轮询   | ⭐⭐⭐⭐   | IoT/电视/CLI      | ✅ 推荐     |
| **刷新令牌**   | `refresh_token`                                    | 服务端     | ⭐⭐⭐⭐   | 所有模式的扩展    | ✅ 必需     |

------

## 五、现代最佳实践推荐

### 1. **Web 应用**





### 2. **单页应用 (SPA)**





### 3. **移动应用**





### 4. **服务间通信**





### 5. **命令行工具**





## 总结

理解不同授权模式的关键是：

1. **谁在使用**（用户 vs 服务）
2. **在哪里运行**（浏览器 vs 原生应用 vs 服务端）
3. **安全要求**（能否暴露令牌在前端）