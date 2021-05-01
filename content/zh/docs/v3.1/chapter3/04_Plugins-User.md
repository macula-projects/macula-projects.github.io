---
type: docs
title: "用户相关"
linkTitle: "用户相关"
weight: 4
description: "本文介绍了登录用户相关信息"
---

## 用户凭据UserPrincipal

实际上，只需要通过用户名，即可通过UserContextFactory构建出用户上下文信息，对于已登录的用户，可以通过SecurityUtils.getUserDetails\(\)获取用户信息。通过用户上下文可方便的得到一些用户相关信息。

1. 用户凭据信息

   用户接口是提供登录用户（或指定用户）信息的主要方式，也可构建出用户上下文信息。

   ```java
   public interface UserPrincipal extends UserDetails, Principal {

   }

   public abstract class AbstractUserPrincipal implements UserPrincipal {

   }

   public class UserPrincipalImpl extends AbstractUserPrincipal {
      //~~~~~~~~~~~~~~~Principal~~~~~~~~~~~~~~~~~~~//
       /**
        * 获取当前用户名
        */
       @Override
       public String getName() {
           return getUsername();
       }

       @Override
       public int hashCode() {
           return getUsername().hashCode();
       }

       @Override
       public String toString() {
           return getName();
       }

       @Override
       public boolean equals(Object rhs) {
           if (rhs instanceof UserPrincipalImpl) {
               return getUsername().equals(((UserPrincipalImpl) rhs).getUsername());
           }
           return false;
       }

       //~~~~~~~~~~~~~~~~~~~ for UserDetails ~~~~~~~~~~~~~~~~~~~~~~~~~//
       /**
        * 用户的角色集合
        */
       @Override
       @JsonIgnore
       public Collection<? extends GrantedAuthority> getAuthorities() {
           if (authorities == null || authorities.isEmpty()) {
               Collection<Role> roles = this.getRoles();
               Collection<String> roleCodes = new ArrayList<String>();
               for (Role role : roles) {
                   if (!role.isOpposite()) {
                       roleCodes.add(role.getCode());
                   } else {
                       roleCodes.add(InnerSecurityUtils.createOppositeRoleCode(role.getCode()));
                   }
               }
               List<GrantedAuthority> tmpAuthorities = AuthorityUtils.createAuthorityList(roleCodes
                       .toArray(new String[roleCodes.size()]));
               authorities = Collections.unmodifiableCollection(tmpAuthorities);
           }
           return authorities;
       }

       /**
        * 当前用户名
        */
       @Override
       public String getUsername() {
           return this.username;
       }

       /**
        * 当前用户密码，不会被JSON序列化
        */
       @Override
       @JsonIgnore
       public String getPassword() {
           return password;
       }

       /**
        * 账号是否未过期
        */
       @Override
       public boolean isAccountNonExpired() {
           Boolean r = (Boolean) this.getAttributeValue(ACCOUNT_NON_EXPIRED_ATTR);
           return r != null ? r.booleanValue() : true;
       }

       /**
        * 账号是否没有被锁定
        */
       @Override
       public boolean isAccountNonLocked() {
           Boolean r = (Boolean) this.getAttributeValue(ACCOUNT_NON_LOCKED_ATTR);
           return r != null ? r.booleanValue() : true;
       }

       /**
        * 凭据是否未过期
        */
       @Override
       public boolean isCredentialsNonExpired() {
           Boolean r = (Boolean) this.getAttributeValue(CREDENTIALS_NON_EXPIRED_ATTR);
           return r != null ? r.booleanValue() : true;
       }

       /**
        * 是否有效用户
        */
       @Override
       public boolean isEnabled() {
           Boolean r = (Boolean) this.getAttributeValue(ENABLED_ATTR);
           return r != null ? r.booleanValue() : true;
       }

       // ~~~~~~~~~~~~~~~~~~ extend attribute~~~~~~~~~~~~~~~~~~~~~~~~~//
       /**
        * 用户姓名
        */
       @Override
       public String getNickname() {
           String nickname = (String) getAttributeValue(NICKNAME_ATTR);
           return nickname != null ? nickname : getUsername();
       }

       /**
        * 用户所在地区
        */
       @Override
       public Locale getLocale() {
           String locale = (String) getAttributeValue(LOCALE_ATTR);
           if (locale != null) {
               return new Locale(locale);
           }
           return null;
       }

       /**
        * 是否是一个需要锁屏的用户
        */
       @Override
       public boolean isIllegalRequest() {
           String illegalRequest = (String) getAttributeValue(ILLEGAL_REQUEST_ATTR);
           if (illegalRequest != null) {
               return Boolean.valueOf(illegalRequest);
           }
           return false;
       }

       /**
        * 用户登录的验证类型（password、dyna_code、service_pass等）
        */
       @Override
       public String getLoginAuthType() {
           String authType = (String) getAttributeValue(AUTH_TYPE);
           if (StringUtils.isEmpty(authType)) {
               authType = MaculaConstants.AUTH_TYPE_UNKNOWN;
           }
           return authType;
       }

       /**
        * 登录终端类型（PC、MOBILE、KIOSK）
        */
       @Override
       public String getLoginTerminalType() {
           String terminalType = (String) getAttributeValue(TERMINAL_TYPE);
           if (StringUtils.isEmpty(terminalType)) {
               terminalType = MaculaConstants.TERMINAL_PC;
           }
           return terminalType;
       }

       // ~~~~~~~~~~~~~~~~~~~~~~~~~~ cas attributes ~~~~~~~~~~~~~~~~~~~~~~~~~~~ //

       @Override
       public Object getAttributeValue(String attribute) {
           return this.attributes.get(attribute);
       }

       @Override
       public void addAttributes(Map<String, Object> attributes) {
           if (attributes != null) {
               this.attributes.putAll(attributes);
           }
       }

       @Override
       public void addAttribute(String attribute, Object value) {
           this.attributes.put(attribute, value);
       }
   }
   ```

2. 用户信息的获取

   * 通过SecurityUtils获取

     对于已经登录的用户，可通过SecurityUtils来获取

     ```java
     UserPrincipal principal = SecurityUtils.getUserDetails();

     public final class SecurityUtils {

         private SecurityUtils() {
             // Noops!
         }

         private static final AuthenticationTrustResolver authenticationTrustResolver = new AuthenticationTrustResolverImpl();

         public static final List<String> IGNORE_USERS = Arrays.asList("_cas_stateful_", "_cas_stateless_", MaculaConstants.ANONYMOUS_USER);

         private static final Map<String, UserPrincipal> STATIC_CACHE_USERS = new ConcurrentHashMap<String, UserPrincipal>();

         /**
          * 获取登录用户信息.
          * 
          * @return 登录用户上下文信息
          */
         public static UserPrincipal getUserDetails() {
             Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
             if (authentication != null) {
                 Object principal = authentication.getPrincipal();
                 if (principal instanceof UserPrincipal) {
                     return (UserPrincipal) principal;
                 }

                 HttpServletRequest request = ApplicationContext.getRequest();
                 Principal requestPrincipal = null;
                 if (request != null) {
                     requestPrincipal = request.getUserPrincipal();
                     if (requestPrincipal instanceof UserPrincipal) {
                         return (UserPrincipal) principal;
                     }
                 }

                 String username = null;

                 if (requestPrincipal != null) {
                     username = requestPrincipal.getName();
                 } else if (principal instanceof String) {
                     username = (String) principal;
                 }

                 if (username != null) {
                     UserPrincipal userPrincipal = STATIC_CACHE_USERS.get(username);
                     if (userPrincipal == null) {
                         userPrincipal = new UserPrincipalImpl((String) principal, null);
                         STATIC_CACHE_USERS.put(username, userPrincipal);
                     }
                     return userPrincipal;
                 }
             }
             // 如果连上下文都没有，则应该是后台运行用户
             return getBackgroundUserDetails();
         }

         /**
          * 判断当前用户是否可以访问URL或者method
          * 
          * @param url
          * @param method
          * @return boolean
          */
         public static boolean hasAccess(String url, String method) {
             return UserContextStaticServiceHolder.getUserPrincipalService().hasAccess(getUserDetails(), url, method);
         }

         public static void clearCachedStaticUserPrincipals() {
             STATIC_CACHE_USERS.clear();
         }

         /**
          * 获取一个后端模拟用户
          */
         public static UserPrincipal getBackgroundUserDetails() {
             return new UserPrincipalImpl(MaculaConstants.BACKGROUND_USER, null);
         }

         /**
          * 获取当前用户上下文，可以用来解析表达式或者规则
          * 
          * @return UserContext
          */
         public static UserContext getUserContext() {
             UserPrincipal userDetails = getUserDetails();
             return userDetails.createUserContext();
         }

         /**
          * 判断当前用户是否是匿名用户
          * 
          * @return boolean
          */
         public static boolean isAnonymous() {
             Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
             return authentication == null || authenticationTrustResolver.isAnonymous(authentication);
         }

         /**
          * 判断是否是匿名用户凭据
          * 
          * @param authentication 带判断的凭据
          * @return boolean
          */
         public static boolean isAnonymous(Authentication authentication) {
             return authentication == null || authenticationTrustResolver.isAnonymous(authentication);
         }

         /**
          * 判断是否是匿名用户
          * @param userPrincipal
          * @return boolean
          */
         public static boolean isAnonymous(UserPrincipal userPrincipal) {
             return MaculaConstants.ANONYMOUS_USER.equals(userPrincipal.getUsername());
         }

         /**
          * 是否是后台用户
          * @param userPrincipal
          * @return boolean
          */
         public static boolean isBackgroudUser(UserPrincipal userPrincipal) {
             return MaculaConstants.BACKGROUND_USER.equals(userPrincipal.getUsername());
         }

         /**
          * 是否是合法用户，系统中有些是保留的用户名，不能使用
          * @param userName
          * @return
          */
         public static boolean isValidUser(String userName) {
             return userName != null && !IGNORE_USERS.contains(userName);
         }
     }
     ```

   * 通过UserContext获取

     如果仅有用户的用户名信息，也可通过先构建UserContext，然后通过UserContext反向构建UserPrincipal的方式构建用户信息。

     ```java
     String userName = "Wilson";
     UserContext userContext = userContextFactory.createContext(userName);
     UserPrincipal userPrincipal = userContext.getUser();
     ```

   * 通过HttpServletRequest获取

     对于已经登录的用户，可以通过HttpServletRequest来直接获取用户上下文，如：

     ```java
     UserPrincipal principal = (UserPrincipal) request.getUserPrincipal();
     ```



## 用户上下文UserContext

1. 用户上下文接口

   ```java
   public interface UserContext {

       UserPrincipal getUser();

       String getUsername();

       Object resolve(String property);

       Object resolve(String property, UserContext userContext);

       boolean isResolved(String property);

       EvaluationContext createEvaluationContext();

       void fireUserChangedEvent();

       PolicyResult vote(String code, Object target);

       PolicyResult vote(String code);

       void destory();

     }
   ```

2. 通过UserPrincipal直接获取

   如果已经有了UserPrincipal信息，可通过UserPrincipal信息直接获取。

   ```java
   UserContext userContext = userPrincipal.createUserContext();
   ```

3. 通过UserContextFactory构建

   如果仅有用户的用户名信息，可通过UserContextFactory构建UserContext。

   ```java
   String userName = "Wilson";

   UserContext userContext = userContextFactory.createContext(userName);
   ```


_**重要**_

_对于登录用户的UserPrincipal来说，其信息是与用户登录Session相关的，在Session失效后，其UserPrincipal将自动失效。_

