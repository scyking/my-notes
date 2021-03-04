# springboot整合shiro

标签（空格分隔）： shiro spring-boot

---

## 说明

仅仅是初步整合，细节配置需结合业务做修改。

[官网参考地址](http://shiro.apache.org/spring-boot.html#configuration-properties)

## 添加依赖

```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>${apache-shiro.version}</version>
</dependency>
```
    
## 添加必要的bean

- 域管理对象

    ```
    @Bean
    public Realm realm() {
        // 自定义Realm实例
        return new UserRealm();
    }
    ```
    自定义Realm实例：
    ```
    public class UserRealm extends AuthorizingRealm {

        /**
         * 授权：用于验证权限
         *
         * @param principalCollection
         * @return
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            // 获取用户信息
            // principalCollection.getPrimaryPrincipal();
            // 当前用户所有角色
            Set<String> roles = new HashSet<>();
            // 当前用户所有权限
            Set<String> permissions = new HashSet<>();
    
            SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
            simpleAuthorizationInfo.setStringPermissions(permissions);
            simpleAuthorizationInfo.setRoles(roles);
            return simpleAuthorizationInfo;
        }
    
        /**
         * 认证：用于登录
         *
         * @param authenticationToken
         * @return
         * @throws AuthenticationException
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            // 添加用户登录信息
            // authenticationToken
            SimpleAuthenticationInfo info = new SimpleAuthenticationInfo("myPrincipal", "myCredentials", getName());
            return info;
        }
    }
    ```
    
- ShiroFilterChainDefinition

    ```
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        // all paths are managed via annotations
        chainDefinition.addPathDefinition("/**", "anon");
        // or allow basic authentication, but NOT require it.
        // chainDefinition.addPathDefinition("/**", "authcBasic[permissive]");
        return chainDefinition;
    }
    ```
    
- 缓存管理

    ```
    @Bean
    protected CacheManager cacheManager() {
        return new MemoryConstrainedCacheManager();
    }
    ```
    
## 使用

- `@RequiresGuest`              只有游客可以访问
- `@RequiresAuthentication`     需要登录才能访问
- `@RequiresUser`               已登录的用户或“记住我”的用户能访问
- `@RequiresRoles`              已登录的用户需具有指定的角色才能访问
- `@RequiresPermissions`        已登录的用户需具有指定的权限才能访问




