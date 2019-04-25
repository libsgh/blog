---
title: spring boot 集成spring security + 验证码
categories: spring boot
date: 2018-4-23 14:21:42
tags: [java,spring boot]
comments: true
---

### pom.xml
- spring boot 版本：2.0.0.RELEASE
- 验证码：hutool工具生成，可以选择其他方式
- freemarker模板

<!-- more -->

``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions> 
        <exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
	</exclusions> 
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>com.xiaoleilu</groupId>
    <artifactId>hutool-core</artifactId>
    <version>3.3.2</version>
</dependency>
```
### application.yml

>主要是freemarker配置

```
spring:
  freemarker:
    cache: 'false'
    charset: UTF-8
    check-template-location: 'true'
    content-type: text/html
    expose-request-attributes: 'true'
    expose-session-attributes: 'true'
    request-context-attribute: request
    suffix: .html
    template-loader-path: classpath:views
```

### Security配置类
> 因为要用验证码验证，这里就不能用默认的<code>formLogin()</code>了，需要添加登录认证过滤器<code>CaptchaAuthenticationFilter</code>集成自
<code>UsernamePasswordAuthenticationFilter</code>

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Autowired
	private CustomDetailService userDetailsService;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests()
				.antMatchers("/dwr/**","/user/getCaptcha","/tool/**","/toLogin","/logout","/**/favicon.ico").permitAll()
				.antMatchers("/**").hasRole("ADMIN")
				.anyRequest().fullyAuthenticated()
				/*.and()
				.formLogin()
				.loginPage("/toLogin")
				.failureUrl("/toLogin?error")
				.usernameParameter("userName")
				.passwordParameter("password")
				.permitAll()*/
			.and()
				.logout()
				.logoutUrl("/logout")
				.deleteCookies("remember-me")
				.logoutSuccessUrl("/toLogin")
				.permitAll()
			.and()
			.rememberMe();
		http.headers().cacheControl();
		http.headers().httpStrictTransportSecurity();
		http.headers().xssProtection();
		http.headers().contentTypeOptions();
		CaptchaAuthenticationFilter captchaAuthenticationFilter = new CaptchaAuthenticationFilter();
		captchaAuthenticationFilter.setAuthenticationManager(authenticationManager());
		http.addFilterAfter(captchaAuthenticationFilter, BasicAuthenticationFilter.class)
		.exceptionHandling()
        .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/toLogin"));
	}
	
	@Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
		    .passwordEncoder(new BCryptPasswordEncoder());
    }
	@Override
    public void configure(WebSecurity web) throws Exception {
        //解决静态资源被拦截的问题
        web.ignoring().antMatchers("/res/**");
    }
	
}
```
- 在生成验证码的时候可以将验证码放入到session中
- 从session中取出验证码信息进行验证
- <code>LoginAuthenticationFailureHandler</code>,在验证码错误的时候能够抛出异常，重定向，定位错误信息
- <code>CaptchaException</code>定义了一个验证码异常，需要继承<code>AuthenticationException</code>

```java
public class CaptchaAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
	
	public static final String SECURITY_FORM_CAPTCHA_KEY = "captcha";
	
    public static final String SESSION_GENERATED_CAPTCHA_KEY = "circleCaptcha";  
    
    public CaptchaAuthenticationFilter() {
    	this.setUsernameParameter("username");
    	this.setPasswordParameter("password");
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/toLogin", "POST"));
        this.setAuthenticationFailureHandler(new LoginAuthenticationFailureHandler());
    }
    
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,  
            HttpServletResponse response) throws AuthenticationException {  
    	CircleCaptcha circleCaptcha = this.obtainGeneratedCaptcha(request);  
        String inputCode = this.obtainCaptcha(request);
        if(!circleCaptcha.verify(inputCode)){  
            throw new CaptchaException("captcha not matched!");
        }  
        String username = obtainUsername(request);
        String password = obtainPassword(request);

        UsernamePasswordAuthenticationToken authRequest
                = new UsernamePasswordAuthenticationToken(username, password);

        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }  
      
    protected String obtainCaptcha(HttpServletRequest request){  
        return request.getParameter(SECURITY_FORM_CAPTCHA_KEY);  
    }  
      
    protected CircleCaptcha obtainGeneratedCaptcha (HttpServletRequest request){  
        return (CircleCaptcha)request.getSession().getAttribute(SESSION_GENERATED_CAPTCHA_KEY);  
    }  
}
```

```java
public class LoginAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {
    private static final String PASS_ERROR_URL = "/toLogin?error";
    private static final String CAPTCHA_ERROR_URL = "/toLogin?captcha";

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        if (exception instanceof CaptchaException) {
            getRedirectStrategy().sendRedirect(request, response, CAPTCHA_ERROR_URL);
        } else {
            getRedirectStrategy().sendRedirect(request, response, PASS_ERROR_URL);
        }
    }
}

```

```java
public class CaptchaException extends AuthenticationException {

	private static final long serialVersionUID = 1L;

	public CaptchaException(String msg) {
		super(msg);
	}

}
```

### 验证码

```java
@RequestMapping(value = "/getCaptcha", method = RequestMethod.GET)
public void captcha(HttpServletRequest request, HttpServletResponse response) throws IOException{
	CircleCaptcha captcha = CaptchaUtil.createCircleCaptcha(200, 100, 4, 20);
	HttpSession session = request.getSession();
	session.setAttribute("circleCaptcha", captcha);
	response.setContentType("image/png");
	OutputStream stream = response.getOutputStream();
	captcha.write(stream);
	stream.flush();
    stream.close();
}
```
### UserDetailsService授权：从数据库中获取登录用户信息，权限，角色

```java
@Service
public class CustomDetailService implements UserDetailsService {
	
	@Autowired
	private SysUserDao userDao;
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		LoginUser user = userDao.findByName(username);
		if (user == null) {
			throw new UsernameNotFoundException(username);
		}
		return user;
	}

}
```
### 定义一个UserDetails类

```java
public class LoginUser implements UserDetails {

	private static final long serialVersionUID = 4542856156463291156L;

	private Integer id;

	private String username;

	private String password;
	
	private String headImg;
	
	private String role;
	
	private Date createTime;
	
	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getRole() {
		return role;
	}

	public void setRole(String role) {
		this.role = role;
	}

	public String getHeadImg() {
		return headImg;
	}

	public void setHeadImg(String headImg) {
		this.headImg = headImg;
	}

	public Date getCreateTime() {
		return createTime;
	}

	public void setCreateTime(Date createTime) {
		this.createTime = createTime;
	}

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		List<GrantedAuthority> auths = new ArrayList<>();
        String role = this.getRole();
        auths.add(new SimpleGrantedAuthority(role));
	    return auths;
	}

	@Override
	public String getPassword() {
		return password;
	}

	@Override
	public String getUsername() {
		return username;
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}
}
```
### 登录跳转

```java
@RequestMapping(value = "/toLogin", method = {RequestMethod.GET, RequestMethod.POST})
public ModelAndView getLoginPage(@RequestParam Optional<String> error, @RequestParam Optional<String> captcha, Map<String,Object> param) {
	if(error.isPresent()) {
		param.put("status", ErrorCode.loginFailed.getCode());
		param.put("message","用户名或密码错误");
	} else {
		param.put("status", ErrorCode.success.getCode());
		param.put("message","登陆成功");
	}
	if(captcha.isPresent()) {
		param.put("status", ErrorCode.loginFailed.getCode());
		param.put("message","请输入正确的验证码");
	}
    return new ModelAndView("user/login");
}
```

### 登录的form表单

>注意<code>csrf</code>如果没有被禁用需要有<code>${_csrf.token}</code>这个参数，<code>remember-me</code>可以实现记住登录的功能需要在<code>HttpSecurity</code>处开启

```
<form id="form" action="${base}/toLogin" method="post">
  <div class="form-group has-feedback">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
    <input type="email" class="form-control" name="username" placeholder="请输入用户名或邮箱">
    <span class="glyphicon glyphicon-user form-control-feedback"></span>
  </div>
  <div class="form-group has-feedback">
    <input type="password" class="form-control" name="password" placeholder="请输入密码">
    <span class="glyphicon glyphicon-lock form-control-feedback"></span>
  </div>
 <div class="form-group">
  	<input type="text" class="form-control" name="captcha" style="width: 120px" placeholder="请输入验证码">
 </div>
 <div class="form-group">
    <img id="captcha" alt="" src="${base}/user/getCaptcha" height="50px" onclick="reImg()">
 </div>
  <div class="row">
    <div class="col-xs-12">
      <div class="checkbox icheck">
        <label>
             <input type="checkbox" name="remember-me" value="true" >
	          <label for="ispersis">自动登陆</label>
	     </label>
      </div>
    </div>
    <!-- /.col -->
  </div>
</form>
```
### 退出表单

> 只支持post请求

```
<form id="logoutForm" action="${base}/logout" method="post">
 	<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
	<a href="#" class="btn bg-navy margin"><span class="glyphicon glyphicon-off"></span> 退出</a>
</form>
```

**页面要显示登录用户信息，需要定义request请求的全局变量，可以用拦截器，但有一种更简单的实现**

```java
@ControllerAdvice
public class CustomControllerAdvice {
	@ModelAttribute("user")
	    public LoginUser getCurrentUser(Authentication authentication) {
	        return (authentication == null) ? null : (LoginUser) authentication.getPrincipal();
	}
	@ModelAttribute("base")
		public String getContextPath(HttpServletRequest request) {
			return request.getContextPath();
	}
}
```
