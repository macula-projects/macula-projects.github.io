---
type: docs
title: "项目集成"
linkTitle: "项目集成"
weight: 3
description: "本文介绍项目开始时的登录验证集成问题"
---

### 登录集成

项目开始时，首先要解决登录的验证问题，通过实现CustomUserLoginRepository接口来解决这个问题：

```java
@Component
public class CustomMyUserLoginRepository implements CustomUserLoginRepository {
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public UserPrincipal loadUserByUsername(String username) throws UsernameNotFoundException {
      // 参考下面代码完成
      JpaUIMUser user = uimUserRepository.findByUserName(username);
      if (user != null) {
          UserPrincipalImpl principal = new UserPrincipalImpl(user.getUserName(), user.getPassword());
          // 根据需要，将额外的用户信息放到Attributes中
          // UserInfo userInfo = xxxx.getUserInfo(username);
          // principal.addAttribute("userInfo", userInfo);
          return principal;
      }
      // throw new UsernameNotFoundException("AbstractUserDetailsAuthenticationProvider.badCredentials");
          }

          @Override
          public void postValidateUserPrincipal(UserPrincipal principal) throws AuthenticationException {
              // TODO 可以校验凭据的合法性，非法则抛出异常
          }
}
```

### 登录日志集成

框架默认会把登录日志记录到框架的表中，但是高并发的生产环境建议实现下面的接口，可以把登录日志记录到您要存放的地方。实现类不受macula.properties的是否记录日志配置的影响

```java
public interface CustomUserSessionHistoryPersistance {

    /**
     * 保存用户登录、修改密码的历史
     * @param userSessionHistory
     */
    void saveUserSessionHistory(UserSessionHistory userSessionHistory);
}
```

### 访问日志集成

框架默认会把访问日志记录到框架的表中，但是高并发的生产环境建议实现下面的接口，可以把登录日志记录到您要存放的地方。实现类不受macula.properties的是否记录日志配置的影响

```
public interface CustomAccessLogPersistance {

    void saveAccessLog(AccessLog accessLog);

}
```



