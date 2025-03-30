SpringBott-3.4.4

------



# Filter

- springboot纯后端项目中，可以自己定义一些Filter功能

## FilterRegistrationBean

- 使用该spring提供的该接口来注册Filter

### 1. 过滤器

```java
package com.erick.princess.filter;

import jakarta.servlet.*;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

@Slf4j
public class LoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        /*1. 请求前处理*/
        log.info("日志Filter开始");

        /*继续执行下一个过滤器*/
        chain.doFilter(request, response);

        /*2. 请求后处理*/
        log.info("日志filter结束");
    }
}
```

```java
package com.erick.princess.filter;


import jakarta.servlet.*;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

@Slf4j
public class AuthFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        /*1. 过滤器开始*/
        log.info("权限Filter开始");

        /*继续下一个*/
        chain.doFilter(request, response);

        log.info("权限Filter结束");
    }
}
```

```java
package com.erick.princess.filter;

import jakarta.servlet.*;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

@Slf4j
public class CheckFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        /*1. 过滤器开始*/
        log.info("Check Filter开始");

        /*继续下一个*/
        chain.doFilter(request, response);

        log.info("Check Filter结束");
    }
}
```

### 2. 顺序配置

- 将过滤器交给spring去管理，并声明执行顺序

![image-20250330115938274](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250330115938274.png)

```java
package com.erick.princess;

import com.erick.princess.filter.AuthFilter;
import com.erick.princess.filter.CheckFilter;
import com.erick.princess.filter.LoggingFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;

@Configuration
public class FilterConfig {
    /*优先级越低，最先执行*/
    /*可使用@Conditional，实现动态的注册*/
    @Bean
    public FilterRegistrationBean<LoggingFilter> loginFilterRegistration() {
        FilterRegistrationBean<LoggingFilter> register = new FilterRegistrationBean<>();
        register.setFilter(new LoggingFilter());
        register.addUrlPatterns("/*");
        register.setOrder(Ordered.HIGHEST_PRECEDENCE);   /*优先级1*/
        return register;
    }

    @Bean
    public FilterRegistrationBean<AuthFilter> authFilterRegistration() {
        FilterRegistrationBean<AuthFilter> register = new FilterRegistrationBean<>();
        register.setFilter(new AuthFilter());
        register.addUrlPatterns("/*");/*匹配所有路径*/
        register.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);   /*优先级2*/
        return register;
    }

    @Bean
    public FilterRegistrationBean<CheckFilter> checkFilterRegistration() {
        FilterRegistrationBean<CheckFilter> register = new FilterRegistrationBean<>();
        register.setFilter(new CheckFilter());
        register.addUrlPatterns("/*");
        register.setOrder(Ordered.HIGHEST_PRECEDENCE + 2);   /*优先级3*/
        return register;
    }
}
```

### 3. 路径匹配规则

- url指的是对应的controller的路径

```bash
# 精确匹配    /api/login    
- 只保护 url是 /api/login的资源


# 目录匹配       
     # /api/*
     - 保护 所有以/api/开始的子路径， 比如/api/login       api/erick/test

# 默认匹配       /*
- 匹配所有路径
```

- 一个filter，可以指定多个匹配路径

```bash
# 精确匹配 > 目录匹配 > 默认匹配

- "/*", "/api/*"                     第二个永远不会生效
- "/api/login", "/api/*", "/*"       正确配置
```

# 入门Demo

- SpringSecurity: 为系统提供声明式安全访问控制功能，减少了为系统安全而编写大量重复代码的工作

```bash
# Authentication
- 验证某个用户，是否为系统中的合法主体
- 用户认证一般要求用户提供用户名和密码，系统通过校验用户名和密码来完成认证

# Authorization
- 验证某个用户，是否有权限执行某个操作
- 不同用户所具有的权限是不同的，比如只读权限和读写权限
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 接口

```java
package com.erick.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/hello")
    public String sayHello() {
        return "test";
    }
}
```

## 访问

- 项目启动后，spring-security已经生效，默认拦截全部请求，如果用户没有登录，则跳转到内置登录页面
- 默认用户名：user。默认密码：项目启动后由spring-security自动生成，每次都不一样

```bash
# url:  访问后会自动跳转到
- http://localhost:8080/test/hello   --->   http://localhost:8080/login
    
# 填写用户名/密码    
- 验证成功：跳转到 http://localhost:8080/test/hello
- 验证失败：继续当前登录页面
```

![image-20250327132839410](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250327132839410.png)

# 默认Authentication

- 账户密码默认是由spring-security定义生成的(默认用户名密码或者自定义用户名密码)，存储在内存中
- 本质就是一系列的过滤器链
- 实际是需要从数据库中查询，因此要自定义认证逻辑
- http://localhost:8080/user/test

## AbstractAuthenticationProcessingFilter

- 实现了Filter接口

![image-20250330131709363](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250330131709363.png)

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
    // 1. 进入验证
		doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
	}

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);
			return;
		}
    // 2. 默认保护所有的url，页面输入目标api后，经过上面的requiresAuthentication()
    // 页面跳转到form表单，用户输入用户名和密码，再次进入验证
		try {
       // 3. 交给子类UsernamePasswordAuthenticationFilter去验证
			Authentication authenticationResult = attemptAuthentication(request, response);
			if (authenticationResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				return;
			}
			this.sessionStrategy.onAuthentication(authenticationResult, request, response);
			// Authentication success
			if (this.continueChainBeforeSuccessfulAuthentication) {
				chain.doFilter(request, response);
			} 
       // 4.1 认证成功
			successfulAuthentication(request, response, chain, authenticationResult);
		}
		catch (InternalAuthenticationServiceException failed) {
			this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
			 // 4.2 认证失败
      unsuccessfulAuthentication(request, response, failed);
		}
		catch (AuthenticationException ex) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, ex);
		}
	}
```

```java
// 认证成功	
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
			Authentication authResult) throws IOException, ServletException {
		SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
		context.setAuthentication(authResult);
		this.securityContextHolderStrategy.setContext(context);
		this.securityContextRepository.saveContext(context, request, response);
		if (this.logger.isDebugEnabled()) {
			this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
		}
		this.rememberMeServices.loginSuccess(request, response, authResult);
		if (this.eventPublisher != null) {
			this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
		}
		this.successHandler.onAuthenticationSuccess(request, response, authResult);
	}
```

```java
// 认证失败	
protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException failed) throws IOException, ServletException {
		this.securityContextHolderStrategy.clearContext();
		this.logger.trace("Failed to process authentication request", failed);
		this.logger.trace("Cleared SecurityContextHolder");
		this.logger.trace("Handling authentication failure");
		this.rememberMeServices.loginFail(request, response);
		this.failureHandler.onAuthenticationFailure(request, response, failed);
	}
```

## UsernamePasswordAuthenticationFilter

- spring提供的实现类，org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
- 默认会使用该过滤链来获取用户前端页面的请求的用户名和密码

```java
	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
    // 1. 只允许post提交
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
    // 2. 从request中解析出 username    request.getParameter()
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : "";
     // 3. 从request中解析出 password    request.getParameter()
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
    // 4. UsernamePasswordAuthenticationToken 去封装了当前用户名和密码
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
    // 5. 用AuthenticationManager去验证,触发验证，后面就会交给UserDetailsService的实现类去具体验证
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

## UserDetailsService接口

- spring-security提供的接口

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

![image-20250329113510447](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250329113510447.png)

### InMemoryUserDetailsManager

-  默认策略，实现上面的接口
-  具体的配置在org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration中

```java
// 如果没有其他的UserDetailsService的实现类，则会使用该类
@ConditionalOnMissingBean(value = { AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class,
		AuthenticationManagerResolver.class }, type = "org.springframework.security.oauth2.jwt.JwtDecoder")
public class UserDetailsServiceAutoConfiguration {
  
  // 将该bean注入进去，成为spring的默认的用户名和密码验证类
	@Bean
	public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
			ObjectProvider<PasswordEncoder> passwordEncoder) {
		SecurityProperties.User user = properties.getUser();
		List<String> roles = user.getRoles();
		return new InMemoryUserDetailsManager(User.withUsername(user.getName())
			.password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
			.roles(StringUtils.toStringArray(roles))
			.build());
	}
	}
}

```

```bash
- 1. 浏览器发起请求，经过拦截器
- 2. 没有认证，执行InMemoryUserDetailsManager中的loadUserByUsername方法
- 3. 将输入的用户名和内存中的进行比较，用户名通过后，再次比较输入的密码和内存中的密码
```

### UserDetails接口

- 上面loadUserByUsername的返回值
- User：由spring-security提供

![image-20250329114523807](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250329114523807.png)

```java
package org.springframework.security.core.userdetails;

import java.io.Serializable;
import java.util.Collection;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;

public interface UserDetails extends Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();   // 用户所有权限

	String getPassword();      // 获取密码
	String getUsername();      // 获取用户名

   // 用户是否过期
	default boolean isAccountNonExpired() { 
		return true;
	}

   // 是否被锁
	default boolean isAccountNonLocked() {
		return true;
	}

   // 密码凭证是否过期
	default boolean isCredentialsNonExpired() {
		return true;
	}

  // 是否开启
	default boolean isEnabled() {
		return true;
	}
}
```

## PasswordEncoder

- spring-security提供的密码编译的工具接口类
- spring-security提供了该类的多种实现：MD5, SHA, BCrypt，用户在注册时候可以直接使用该密码编译器

![image-20250329115517399](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250329115517399.png)

```java
package org.springframework.security.crypto.password;

public interface PasswordEncoder {
   // 对密码加密，在注册用户中，将用户提交的密码加密后再存储到数据库中
   // 相同字符串，每次加密后的数据是不同的，不可逆，肯定不会被暴力枚举
    String encode(CharSequence rawPassword);
   // 用户登录后，新提交的密码，经过相同加密规则加密后，再与数据库保存的加密密码进行匹配
    boolean matches(CharSequence rawPassword, String encodedPassword);

   // 用户判断是否需要对密码再次进行加密，以便使密码更加安全
    default boolean upgradeEncoding(String encodedPassword) {
       return false;
    }
}
```

### BCryptPasswordEncoder

- Spring-security比较推荐的一种密码加密方式

```java
package com.citi;

import org.junit.jupiter.api.Test;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordEncrypt {
    BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();

    @Test
    public void test() {
        /*match的时候并不是通过equals*/
        String encryptPassword = bCryptPasswordEncoder.encode("123456");
        // true
        System.out.println(bCryptPasswordEncoder.matches("123456", encryptPassword));
    }
}
```

# 默认Authorization

## AuthorizationFilter

-  入口的Filter

```java
	@Override
	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)
			throws ServletException, IOException {

		HttpServletRequest request = (HttpServletRequest) servletRequest;
		HttpServletResponse response = (HttpServletResponse) servletResponse;

		if (this.observeOncePerRequest && isApplied(request)) {
			chain.doFilter(request, response);
			return;
		}

		if (skipDispatch(request)) {
			chain.doFilter(request, response);
			return;
		}

		String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
		request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);
		try {
      // 1. getAuthentication(): 获取系统上下文中的用户信息
      // 2. authorize()：验证当前用户
      // 3. 如果配置了跳过某个api，则this.authorizationManager.authorize就会放行当前接口
			AuthorizationResult result = this.authorizationManager.authorize(this::getAuthentication, request);
			this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, result);
			if (result != null && !result.isGranted()) {
				throw new AuthorizationDeniedException("Access Denied", result);
			}
			chain.doFilter(request, response);
		}
		finally {
			request.removeAttribute(alreadyFilteredAttributeName);
		}
	}
```

```java
	private Authentication getAuthentication() {
    // 从securityContextHolderStrategy.getContext()中拿到Authentication信息
		Authentication authentication = this.securityContextHolderStrategy.getContext().getAuthentication();
		if (authentication == null) {
			throw new AuthenticationCredentialsNotFoundException(
					"An Authentication object was not found in the SecurityContext");
		}
		return authentication;
	}
```

## AuthorizationManager

- 验证当前用户的身份信息

# JWT+SpringSecurity

- 默认拦截所有请求，所有请求进来都会进行查看当前身份信息的校验

## 工具类

```java
package com.erick.util;

import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class JWTUtils {

    /*为了方便，暂时用Base64代替*/
    public static String encrypt(String data) {
        Base64.Encoder encoder = Base64.getEncoder();
        return encoder.encodeToString(data.getBytes(StandardCharsets.UTF_8));
    }

    public static String decrypt(String data) {
        Base64.Decoder decoder = Base64.getDecoder();
        return new String(decoder.decode(data));
    }
}
```

## 登录Auth

![image-20250330181518074](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250330181518074.png)

### SecurityBaseConfig

```java
package com.erick.security.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@Slf4j
public class SecurityBaseConfig {
    /*使用SpringSecurity提供的密码处理类
     * 也可以自定义其他加密方式，只需要自己去实现PasswordEncoder接口*/
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /*提供一个AuthenticationManager给容器，后面要用该验证器来验证用户名和密码*/
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### ErickLoginFilter

- 指定路径的才会走该过滤器

```java
package com.erick.security.filter;

import com.erick.entity.NikeUser;
import com.erick.util.JWTUtils;
import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.servlet.FilterChain;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import java.io.IOException;

@Slf4j
public class ErickLoginFilter extends UsernamePasswordAuthenticationFilter {

    /*1. 只有在路径是/user/login，并且方式是post时，才使用*/
    public ErickLoginFilter(AuthenticationManager authenticationManager) {
        this.setPostOnly(true);
        this.setAuthenticationManager(authenticationManager);
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/user/login", "POST"));
    }

    /*验证的逻辑*/
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) {
        try {
            /*1. 解析json请求体*/
            NikeUser user = new ObjectMapper().readValue(request.getInputStream(), NikeUser.class);

            /*2. 创建认证令牌*/
            UsernamePasswordAuthenticationToken authRequest =
                    UsernamePasswordAuthenticationToken.unauthenticated(user.getNikeUserName(), user.getNikeUserPassword());

            /*3. 触发认证流程, 会自动交给UserDetailService去验证*/
            return getAuthenticationManager().authenticate(authRequest);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }


    /*下面两个回调： AbstractAuthenticationProcessingFilter的方法*/
    /*验证成功的回调*/
    @Override
    public void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
                                         Authentication authResult) throws IOException {

        log.info("Successful authentication");
        // 1. 生成对应的token
        User user = (User) authResult.getPrincipal();
        String username = user.getUsername();
        String access_token = JWTUtils.encrypt(username);
        // 2. 响应结果
        response.setStatus(HttpServletResponse.SC_OK);
        response.getWriter().println("ACCESS_TOKEN: " + access_token);
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
    }

    /*验证失败的回调*/
    @Override
    public void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                                           AuthenticationException failed) {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    }
}
```

### SecurityConfig

```java
package com.erick.security.config;

import com.erick.security.filter.ErickLoginFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http, AuthenticationManager authenticationManager) throws Exception {
        /*纯后端项目，禁用系统自带的表单*/
        http.formLogin(AbstractHttpConfigurer::disable);
        /*禁用csrf*/
        http.csrf(AbstractHttpConfigurer::disable);

        http.authorizeHttpRequests(auth ->
                /*放行/user/login接口，不再受AuthorizationFilter的查看*/
                auth.requestMatchers("/user/login").permitAll()
                        .requestMatchers("/user/register").permitAll() /*放行该接口，但是不会走ErickLoginFilter的过滤器*/
        );

        /*添加该配置过滤器*/
        http.addFilterAt(new ErickLoginFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

### NikeUserDetailService

```java
package com.erick.security;

import com.erick.entity.NikeUser;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

/*用来和数据库的用户名密码进行比对*/
@Service
@Slf4j
public class NikeUserDetailService implements UserDetailsService {

    /**
     * @param username 用户通过请求传入的用户名
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("NikeUserDetailService--->loadUserByUsername {}", username);
        // 去数据库查询
        NikeUser userInDB = getUserByUsername(username);
        if (userInDB == null) {
            throw new UsernameNotFoundException("当前用户不存在");
        }

        /*权限不能为null*/
        List<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        /*返回一个UserDetails的实现类：User对象*/
        return new User(username, userInDB.getNikeUserPassword(), grantedAuthorities);
    }

    private NikeUser getUserByUsername(String username) {
        // 1. 从数据库根据username查找该用户名
        NikeUser user = new NikeUser();
        user.setNikeUserName("lucy");
        /*经过BCypt加密后的密码*/
        user.setNikeUserPassword(new BCryptPasswordEncoder().encode("123456"));
        return user;
    }
}
```

## 授权Auth

- 经过上面的认证后，后面请求中就会携带当前ACCESS_TOKEN作为header

```java
package com.erick.security.filter;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

/*一次验证的接口*/
@Component
@Slf4j
public class JWTAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String authorization = request.getHeader("Authorization");
        /*如果token不合法，则验证失败，继续走下面的逻辑*/
        if (authorization == null || !authorization.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        // 验证token是否成功，就可以访问接口
        String accessToken = authorization.substring(7);
        log.info("当前携带的token{}", accessToken);
    }
}

```

```java
package com.erick.security.config;

import com.erick.security.filter.ErickLoginFilter;
import com.erick.security.filter.JWTAuthFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Autowired
    private JWTAuthFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http, AuthenticationManager authenticationManager) throws Exception {
        /*纯后端项目，禁用系统自带的表单*/
        http.formLogin(AbstractHttpConfigurer::disable);
        /*禁用csrf*/
        http.csrf(AbstractHttpConfigurer::disable);

        http.authorizeHttpRequests(auth ->
                /*放行/user/login接口，不再受AuthorizationFilter的查看*/
                auth.requestMatchers("/user/login").permitAll()
                        .requestMatchers("/user/register").permitAll() /*放行该接口，但是不会走ErickLoginFilter的过滤器*/
        );

        /*添加该配置过滤器*/
        http.addFilterAt(new ErickLoginFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);
      
        // JWT的前置处理器
        http.addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```

