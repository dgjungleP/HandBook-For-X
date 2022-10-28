> 安全模块，一般都是默认打开的，建议旨在网关应用中打开，集群内部不建议开启。

## 核心组件

### Authentication 
> 认证/身份验证凭证

#### 内置实现
1. UsernamePasswordAuthenticationToken 基于用户名密码的认证场景
2. RememberMeAuthenticationToken 基于`记住我`的认证场景
3. AnonymousAuthenticationToken 记忆匿名访问的用户

说明：
1. 自己也可以通过`Authentication`接口来自己实现

### AuthenticationProvider
> 认证提供者

#### 内置实现
1. AbstractUserDetailsAuthenticationProvider 基于用户名密码的认证场景
2. RememberMeAuthenticationProvider 基于`记住我`的认证场景
3. AnonymousAuthenticationProvider 记忆匿名访问的用户

说明：
1. 自己也可以通过`AuthenticationProvider`接口来自己实现

### AuthenticationManager
> 认证管理者

### 内部实现类
1. ProviderManager

说明：
1. 整个认证流程的入口



## 一般的制作流程

1. 定制一个凭证/令牌类
2. 定制一个认证提供者和凭证/令牌类进行配套，并完成对自制凭证/令牌实例的验证
3. 定制一个`过滤器类`,从请求中获取用户信息组装成定制凭证/令牌，交付给认证管理者
4. 定制一个HTTP安全认证配置类（实现`AbstractHttpConfigurer`）
5. 定义一个Spring Security的安全配置类（实现`WebSecurityConfigurerAdapter`），对Web容器的HTTP安全认证机制进行配置


## 基于数据源的认证制作流程

### 核心类

1. UserNamePasswordAuthenticationToken
2. AbstractUserDetailsAuthenticationProvider
3. UserDetailsService
4. UserDetails
5. PasswordEncoder