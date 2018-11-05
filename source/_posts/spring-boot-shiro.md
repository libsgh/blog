---
title: spring boot集成shiro、cas
categories: spring boot
date: 2018-3-23 18:51:31
tags: [java,spring boot]
comments: true
---

>**说明**：
1. 用户-角色-权限，对应关系是一对多
2. 需要能够拦截url请求，控制访问权限
3. 可以控制页面按钮、菜单的显示与否
本例采用cas（4.x）单点登录，shiro权限过滤，redis保存登录session

<!-- more -->

### pom.xml中添加添加依赖

```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-cas</artifactId>
    <version>1.2.6</version>
</dependency>
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>2.4.2.1-RELEASE</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.2.6</version>
</dependency>
<!-- redis api -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

### java bean配置shiro

```java
@Configuration
public class ShiroCasConfig {
	
	@Bean
	public FilterRegistrationBean shiroFilterRegistration() {
		FilterRegistrationBean filterRegistration = new FilterRegistrationBean();
		filterRegistration.setFilter(new DelegatingFilterProxy("shiroFilter"));
		filterRegistration.addInitParameter("targetFilterLifecycle", "true");
		filterRegistration.setEnabled(true);
		filterRegistration.addUrlPatterns("/*");
		return filterRegistration;
	}
	
	@Bean(name = "lifecycleBeanPostProcessor")
    public LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
	
	@Bean(name = "shiroFilter")
	public ShiroFilterFactoryBean shiroFilter(ShiroConfigProperties shiroConfigProperties){
		ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
		bean.setSecurityManager(getDefaultWebSecurityManager(shiroConfigProperties));
		bean.setLoginUrl(shiroConfigProperties.getLoginUrl());
		bean.setSuccessUrl("/");
		bean.setUnauthorizedUrl("/403");
		
		LogoutFilter logoutFilter = new LogoutFilter();
		logoutFilter.setRedirectUrl(shiroConfigProperties.getLogoutUrl());
		
		Map<String, Filter> filters = new LinkedHashMap<>();
		filters.put("casFilter", getCasFilter());
		filters.put("logout", logoutFilter);
		filters.put("casLogout", casLogoutFilter(shiroConfigProperties));
		filters.put("guttv_auth", new ShiroAuthFilter());
		filters.put("user", new AjaxUserFilter());
		bean.setFilters(filters);
		Map<String, String> chains = new LinkedHashMap<>();
		chains.put("/authentication*", "casLogout,casFilter");
		chains.put("/resources/**", "anon");
		chains.put("/403", "anon");
		chains.put("/help", "anon");
		chains.put("/about", "anon");
		chains.put("/readme", "anon");
		chains.put("/parseMD", "anon");
		chains.put("/msg/**", "anon");
		chains.put("/charts/**", "anon");
		chains.put("/logout", "logout");
		chains.put("/health", "anon");
		chains.put("/info", "anon");
		chains.put(shiroConfigProperties.getManagementContextPath() + "/**", "anon");//放过所有端点请求
		chains.put("/**", "casLogout,user,guttv_auth");
		bean.setFilterChainDefinitionMap(chains);
		return bean;
	}
	
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(ShiroConfigProperties shiroConfigProperties) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(casRealm());
        //securityManager.setCacheManager(new MemoryConstrainedCacheManager());
        //采用redis缓存
        securityManager.setCacheManager(getCacheManager(shiroConfigProperties));
        securityManager.setSessionManager(getSessionManager(shiroConfigProperties));
        securityManager.setSubjectFactory(new CasSubjectFactory());
        return securityManager;
    }
    
	/**
	 * @Description: cas登出过滤器
	 * @return
	 */
	@Bean(name = "casLogout")
	public CasLogoutFilter casLogoutFilter(ShiroConfigProperties shiroConfigProperties) {
		CasLogoutFilter casFilter= new CasLogoutFilter();
		casFilter.setSessionManager(getSessionManager(shiroConfigProperties));
		return casFilter;
	}
	
	/**
     * CAS Filter
     */
    @Bean(name = "casFilter")
    public CasFilter getCasFilter() {
        CasFilter casFilter = new CasFilter();
        casFilter.setName("casFilter");
        casFilter.setEnabled(true);
        casFilter.setFailureUrl("/403");
        return casFilter;
    }
	
	/**
	 * @Description: session管理器
	 * @return
	 */
	@Bean
	public DefaultWebSessionManager getSessionManager(ShiroConfigProperties shiroConfigProperties) {
		DefaultWebSessionManager dws = new DefaultWebSessionManager();
		dws.setGlobalSessionTimeout(shiroConfigProperties.getSessiontimeout());//设置全局超时时间
		dws.setDeleteInvalidSessions(true);
		dws.setSessionValidationSchedulerEnabled(true);
		dws.setSessionValidationScheduler(sessionValidationScheduler(shiroConfigProperties.getInterval()));//设置会话调度器
		dws.setSessionDAO(sessionDAO(shiroConfigProperties));
		dws.setSessionIdCookieEnabled(true);
		return dws;
	}
	
	/**
	 * @Description: sessionDao
	 * @return
	 */
	@Bean
	public RedisSessionDAO sessionDAO(ShiroConfigProperties shiroConfigProperties) {
		RedisSessionDAO rd = new RedisSessionDAO();
		rd.setRedisManager(getRedisManager(shiroConfigProperties));
		return rd;
	}
	
	@Bean
	public MethodInvokingFactoryBean methodInvokingFactoryBean(ShiroConfigProperties shiroConfigProperties){
		MethodInvokingFactoryBean bean = new MethodInvokingFactoryBean();
		bean.setStaticMethod("org.apache.shiro.SecurityUtils.setSecurityManager");
		bean.setArguments(new Object[]{getDefaultWebSecurityManager(shiroConfigProperties)});
		return bean;
	}
	
	/**
	 * @Description: redis管理器
	 * @return
	 */
	@Bean
	public RedisManager getRedisManager(ShiroConfigProperties shiroConfigProperties) {
		RedisManager rm = new RedisManager();
		rm.setHost(shiroConfigProperties.getHost());
		rm.setPassword(shiroConfigProperties.getPassword());
		rm.setExpire(shiroConfigProperties.getExpire());
		rm.setPort(shiroConfigProperties.getPort());
		return rm;
	}
	
	  /**
	 * @Description: redis缓存管理器
	 * @return
	 */
	@Bean
	public RedisCacheManager getCacheManager(ShiroConfigProperties shiroConfigProperties) {
		RedisCacheManager rcm = new RedisCacheManager();
		rcm.setRedisManager(getRedisManager(shiroConfigProperties));
		return rcm;
	}
    
   /**
	 * @Description: 会话验证调度器
	 * @return
	 */
	@Bean(name = "sessionValidationScheduler")
	public ExecutorServiceSessionValidationScheduler sessionValidationScheduler(long interval) {
		ExecutorServiceSessionValidationScheduler esss = new ExecutorServiceSessionValidationScheduler();
		esss.setInterval(interval);
		//esss.setSessionManager(getSessionManager());
		return esss;
	}

    @Bean
	public MyCasRealm casRealm(){
		return new MyCasRealm();
	}
    
}
```

**注入一些属性配置**

>cas服务端地址、client端地址、redis连接参数 

```java
@Component
public class ShiroConfigProperties {
	
	@Value("${cas.shiro.casServer}")
	private String casServer;
	
	@Value("${cas.shiro.casClient}")
	private String casClient;
	
	@Value("${cas.shiro.casServer}/login?service=${cas.shiro.casClient}/authentication")
	private String loginUrl;
	
	@Value("${cas.shiro.casServer}/logout?service=${cas.shiro.casClient}")
	private String logoutUrl;
	
	@Value("${redis.host}") 
	private String host;
	
	@Value("${redis.port}") 
	private int port;
	
	@Value("${redis.expire}") 
	private int expire;
	
	@Value("${redis.password}") 
	private String password;
	
	@Value("${session.timeout}") 
	private long sessiontimeout;
	
	@Value("${interval.time}") 
	private long interval;
	
	@Value("${management.context-path}") 
	private String managementContextPath;
	
	public String getLogoutUrl() {
		return logoutUrl;
	}

	public String getCasServer() {
		return casServer;
	}

	public String getCasClient() {
		return casClient;
	}

	public String getLoginUrl() {
		return loginUrl;
	}

	public String getHost() {
		return host;
	}

	public int getPort() {
		return port;
	}

	public int getExpire() {
		return expire;
	}

	public String getPassword() {
		return password;
	}

	public long getSessiontimeout() {
		return sessiontimeout;
	}

	public long getInterval() {
		return interval;
	}

	public String getManagementContextPath() {
		return managementContextPath;
	}
	
}
```

**UserFilter**
>忘记为什么要加这个过滤器了，如果去掉会有问题

```java
public class AjaxUserFilter extends UserFilter {
	
	 protected boolean onAccessDenied(ServletRequest request,
        ServletResponse response) throws Exception {
		 
        HttpServletRequest req = WebUtils.toHttp(request);
        String xmlHttpRequest = req.getHeader("X-Requested-With");
        if ( xmlHttpRequest != null )
            if ( xmlHttpRequest.equalsIgnoreCase("XMLHttpRequest") )  {
                HttpServletResponse res = WebUtils.toHttp(response);
                res.sendError(HttpServletResponse.SC_UNAUTHORIZED);
                return Boolean.FALSE;
        }
        return super.onAccessDenied(request, response);
    }
}
```

**url权限过滤**
>这里判断了 如果是ajax请求返回status和msg方便前端弹出提示,如果非ajax直接重定向到403

```java
/**
 * 自定义权限url过滤
 * 
 * @author libs 2016-9-26
 */
public class ShiroAuthFilter extends AuthorizationFilter {
	
	private AntPathMatcher urlMatcher = new AntPathMatcher();
	private Gson gson = new Gson();
	
	@Override
	protected boolean isAccessAllowed(ServletRequest servletRequest, ServletResponse servletResponse, Object obj)
			throws Exception {
		String requestURI = WebUtils.getPathWithinApplication(WebUtils.toHttp(servletRequest));
		Subject subject = this.getSubject(servletRequest, servletResponse);
		User user = (User) subject.getPrincipal();
		// 超级管理员无阻
		if (subject.hasRole(UserAuthority.admin.getValue()))
			return Boolean.TRUE;
		// 通过subject判断用户有没有些url权限
		return this.isPermitted(user, Enums.Project.OMS.getValue() + Constants.AUTH_SEPARATOR + requestURI);
	}
	
	/**
     * Handles the response when access has been denied.  It behaves as follows:
     * <ul>
     * <li>If the {@code Subject} is unknown<sup><a href="#known">[1]</a></sup>:
     * <ol><li>The incoming request will be saved and they will be redirected to the login page for authentication
     * (via the {@link #saveRequestAndRedirectToLogin(javax.servlet.ServletRequest, javax.servlet.ServletResponse)}
     * method).</li>
     * <li>Once successfully authenticated, they will be redirected back to the originally attempted page.</li></ol>
     * </li>
     * <li>If the Subject is known:</li>
     * <ol>
     * <li>The HTTP {@link HttpServletResponse#SC_UNAUTHORIZED} header will be set (401 Unauthorized)</li>
     * <li>If the {@link #getUnauthorizedUrl() unauthorizedUrl} has been configured, a redirect will be issued to that
     * URL.  Otherwise the 401 response is rendered normally</li>
     * </ul>
     * <code><a name="known">[1]</a></code>: A {@code Subject} is 'known' when
     * <code>subject.{@link org.apache.shiro.subject.Subject#getPrincipal() getPrincipal()}</code> is not {@code null},
     * which implicitly means that the subject is either currently authenticated or they have been remembered via
     * 'remember me' services.
     * 20170104 modify by libs 
     * 修改：判断是否是ajax请求，ajax请求输出json
     * @param request  the incoming <code>ServletRequest</code>
     * @param response the outgoing <code>ServletResponse</code>
     * @return {@code false} always for this implementation.
     * @throws IOException if there is any servlet error.
     */
	protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws IOException {
		HttpServletRequest httpRequest = (HttpServletRequest) request;  
        HttpServletResponse httpResponse = (HttpServletResponse) response;  
        Map<String, Object> map = Maps.newHashMap();
        
        Subject subject = getSubject(request, response);
        // If the subject isn't identified, redirect to login URL
        if (subject.getPrincipal() == null) {
            if (HttpUtil.isAjax(httpRequest)) {
            	map.put("status", 911);
            	map.put("msg", "您尚未登录或登录时间过长,请重新登录！");
            	HttpUtil.outJson(httpResponse, gson.toJson(map));  
            } else {  
                saveRequestAndRedirectToLogin(request, response);  
            }  
        } else {
        	 if (HttpUtil.isAjax(httpRequest)) {
        		 map.put("status", HttpServletResponse.SC_FORBIDDEN);
        		 map.put("msg", "未授权的访问，请重新登录或者联系管理员！");
             	 HttpUtil.outJson(httpResponse,  gson.toJson(map));  
             } else {  
	            // If subject is known but not authorized, redirect to the unauthorized URL if there is one
	            // If no unauthorized URL is specified, just return an unauthorized HTTP status code
	            String unauthorizedUrl = getUnauthorizedUrl();
	            //SHIRO-142 - ensure that redirect _or_ error code occurs - both cannot happen due to response commit:
	            if (StringUtils.hasText(unauthorizedUrl)) {
	                WebUtils.issueRedirect(request, response, unauthorizedUrl);
	            } else {
	                WebUtils.toHttp(response).sendError(HttpServletResponse.SC_UNAUTHORIZED);
	            }
            }  
        }
        return false;
    }
	
	/**
	 * 自定义url权限认证
	 * @param user
	 * @param url 
	 * @return
	 */
	private boolean isPermitted(User user, String url) {
		List<String> list = user.getAuthUrls();
		if(list != null && list.size() != 0){
			for (String pattern : list) {
				if(urlMatcher.match(pattern, url)){
					//通过校验
					return Boolean.TRUE;
				}
			}
		}
		return Boolean.FALSE;
	}

}
```

** MyCasRealm 自定义授权**

>这里获取用户所有的权限url封装到user中，用户权限信息会存到redis中

```java
public class MyCasRealm extends CasRealm{
	
	
	private static Logger log = LoggerFactory.getLogger(MyCasRealm.class);
	
	@Autowired
	private UserServiceProxy userServiceProxy;
	
	@Autowired
	private UserRoleServiceProxy userRoleServiceProxy;
	
	private Gson gson = new Gson();
	@Value("${cas.shiro.casServer}")
	public String casServer;
	@Value("${cas.shiro.casClient.oms_console}")
	public String casClient;
	
	@PostConstruct
	public void initProperty(){
		setDefaultRoles("adminrole");
		// cas服务端地址
        setCasServerUrlPrefix(casServer);
        // 客户端回调地址
        setCasService(casClient+"/authentication");
    }
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)
			throws AuthenticationException {
		CasToken casToken = (CasToken)token;
		if (token == null) {
			return null;
		}
		String ticket = (String)casToken.getCredentials();
		if (!org.apache.shiro.util.StringUtils.hasText(ticket)) {
			return null;
		}
		TicketValidator ticketValidator = ensureTicketValidator();
		try{
			Assertion casAssertion = ticketValidator.validate(ticket, getCasService());

			AttributePrincipal casPrincipal = casAssertion.getPrincipal();
			String userId = casPrincipal.getName();
			log.debug("Validate ticket : {} in CAS server : {} to retrieve user : {}", new Object[] { ticket, 
			getCasServerUrlPrefix(), userId });

			Map attributes = casPrincipal.getAttributes();

			casToken.setUserId(userId);
			String rememberMeAttributeName = getRememberMeAttributeName();
			String rememberMeStringValue = (String)attributes.get(rememberMeAttributeName);
			boolean isRemembered = (rememberMeStringValue != null) && (Boolean.parseBoolean(rememberMeStringValue));
			if (isRemembered) {
				casToken.setRememberMe(true);
			}
			//subejct中添加用户的权限，方便获取
			User user = this.obtainGrantedUserAuthUrls(userId);
			List principals = CollectionUtils.asList(new Object[] { user, attributes });
			PrincipalCollection principalCollection = new SimplePrincipalCollection(principals, getName());
			return new SimpleAuthenticationInfo(principalCollection, ticket);
		} catch (TicketValidationException e) {
			throw new CasAuthenticationException("Unable to validate ticket [" + ticket + "]", e);
		}
	}
	/**
	 * 自定义授权
	 */
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals){
		User user =  (User) principals.getPrimaryPrincipal();
		List<String> permissions = user.getAuthUrls();
		SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
		simpleAuthorizationInfo.addStringPermissions(permissions);
		simpleAuthorizationInfo.addRole(user.getAuthority());
		return simpleAuthorizationInfo;
	}

    /**
     * 获取用户的urllist
     * @param username
     * @return
     */
	private User obtainGrantedUserAuthUrls(String username) {
		User query = new User();
		query.setLoginname(username);
		String userStr= userServiceProxy.getByLoginName(query);
		User user = gson.fromJson(userStr, User.class);
		UserRole ur = new UserRole();
		ur.setUserId(user.getId());
		List<Auth> list = userRoleServiceProxy.getUserAllAuths(ur);
		List<String> urList = Lists.newArrayList();
		for (Auth auth : list) {
			if (auth != null && StringUtils.isNotBlank(auth.getAuthUrl())) {
				String[] urls = auth.getAuthUrl().split(";");
				for (String url : urls) {
					urList.add(auth.getSystem()+Constants.AUTH_SEPARATOR+url);
				}
			}
		}
		user.setAuthUrls(urList);
		return user;
}
```

**CasLogoutFilter、SingleSignOutHandler单点登出**

>在一个系统中退出，其他系统也会退出

```java
public class CasLogoutFilter extends AdviceFilter{
    private static final Logger log = LoggerFactory.getLogger(CasLogoutFilter.class);
    private static final SingleSignOutHandler HANDLER = new SingleSignOutHandler();

    private SessionManager sessionManager;

    public void setSessionManager(SessionManager sessionManager) {
        this.sessionManager = sessionManager;
    }
    /**
     * 如果请求中包含了ticket参数，记录ticket和sessionID的映射
     * 如果请求中包含logoutRequest参数，标记session为无效
     * 如果session不为空，且被标记为无效，则登出
     * 
     * @param request  the incoming ServletRequest
     * @param response the outgoing ServletResponse
     * @return 是logoutRequest请求返回false，否则返回true
     * @throws Exception if there is any error.
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest req = (HttpServletRequest)request;
        if (HANDLER.isTokenRequest((HttpServletRequest)req)) {
            //通过浏览器发送的请求，链接中含有token参数，记录token和sessionID
            HANDLER.recordSession(req);
            return true;
        } else if (HANDLER.isLogoutRequest(req)) {
            //cas服务器发送的请求，链接中含有logoutRequest参数，在之前记录的session中设置logoutRequest参数为true
            //因为Subject是和线程是绑定的，所以无法获取登录的Subject直接logout
            HANDLER.invalidateSession(req,sessionManager);
            // Do not continue up filter chain
            return false;
        } else {
            log.trace("Ignoring URI " + req.getRequestURI());
        }
        Subject subject = SecurityUtils.getSubject();
        Session session = subject.getSession(false);
        if (session!=null&&session.getAttribute(HANDLER.getLogoutParameterName())!=null) {
            try {
                subject.logout();
            } catch (SessionException ise) {
                log.debug("Encountered session exception during logout.  This can generally safely be ignored.", ise);
            }
        }
        return true;
    }
}
```

```java
/**
 * Performs CAS single sign-out operations in an API-agnostic fashion.
 *
 */
public final class SingleSignOutHandler {

    /** Logger instance */
    private final Log log = LogFactory.getLog(getClass());

    /** The name of the artifact parameter.  This is used to capture the session identifier. */
    private String artifactParameterName = "ticket";

    /** Parameter name that stores logout request */
    private String logoutParameterName = "logoutRequest";

    private HashMapBackedSessionMappingStorage storage = new HashMapBackedSessionMappingStorage();

    protected SingleSignOutHandler(){
        init();
    }
    /**
     * @param name Name of the authentication token parameter.
     */
    public void setArtifactParameterName(final String name) {
        this.artifactParameterName = name;
    }

    /**
     * @param name Name of parameter containing CAS logout request message.
     */
    public void setLogoutParameterName(final String name) {
        this.logoutParameterName = name;
    }
    protected String getLogoutParameterName() {
        return this.logoutParameterName;
    }

    /**
     * Initializes the component for use.
     */
    public void init() {
        CommonUtils.assertNotNull(this.artifactParameterName, "artifactParameterName cannot be null.");
        CommonUtils.assertNotNull(this.logoutParameterName, "logoutParameterName cannot be null.");
    }

    /**
     * Determines whether the given request contains an authentication token.
     *
     * @param request HTTP reqest.
     *
     * @return True if request contains authentication token, false otherwise.
     */
    public boolean isTokenRequest(final HttpServletRequest request) {
        return CommonUtils.isNotBlank(CommonUtils.safeGetParameter(request, this.artifactParameterName));
    }

    /**
     * Determines whether the given request is a CAS logout request.
     *
     * @param request HTTP request.
     *
     * @return True if request is logout request, false otherwise.
     */
    public boolean isLogoutRequest(final HttpServletRequest request) {
        return "POST".equals(request.getMethod()) && !isMultipartRequest(request) &&
            CommonUtils.isNotBlank(CommonUtils.safeGetParameter(request, this.logoutParameterName));
    }

    /**
     * 记录请求中的token和sessionID的映射对
     * 
     * @param request HTTP request containing an authentication token.
     */
    public void recordSession(final HttpServletRequest request) {
        Session session = SecurityUtils.getSubject().getSession();

        final String token = CommonUtils.safeGetParameter(request, this.artifactParameterName);
        if (log.isDebugEnabled()) {
            log.debug("Recording session for token " + token);
        }

        storage.addSessionById(token, session);
    }

    /**
     * 从logoutRequest参数中解析出token，根据token获取到sessionID，再根据sessionID获取到session，设置logoutRequest参数为true
     * 从而标记此session已经失效。
     *
     * @param request HTTP request containing a CAS logout message.
     */
    public void invalidateSession(final HttpServletRequest request, final SessionManager sessionManager) {
        final String logoutMessage = CommonUtils.safeGetParameter(request, this.logoutParameterName);
        if (log.isTraceEnabled()) {
            log.trace ("Logout request:\n" + logoutMessage);
        }

        final String token = XmlUtils.getTextForElement(logoutMessage, "SessionIndex");
        if (CommonUtils.isNotBlank(token)) {
            Serializable sessionId = storage.getSessionIDByMappingId(token);
            if (sessionId!=null) {
                try {
                    Session session = sessionManager.getSession(new DefaultSessionKey(sessionId));
                    if(session != null) {
                        //设置会话的logoutParameterName 属性表示无效了，这里直接使用了request的参数名
                        session.setAttribute(logoutParameterName, true);
                        if (log.isDebugEnabled()) {
                            log.debug ("Invalidating session [" + sessionId + "] for token [" + token + "]");
                        }
                    }
                } catch (Exception e) {

                }
            }
        }
    }

    private boolean isMultipartRequest(final HttpServletRequest request) {
        return request.getContentType() != null && request.getContentType().toLowerCase().startsWith("multipart");
    }
}
```

**存储ticket到sessionID的映射**

```java
/**
 * 存储ticket到sessionID的映射
 */
public final class HashMapBackedSessionMappingStorage {
	
    /**
     * Maps the ID from the CAS server to the Session ID.
     */
    private final Map<String,Serializable> MANAGED_SESSIONS_ID = new HashMap<String,Serializable>();

	public synchronized void addSessionById(String mappingId, Session session) {
        MANAGED_SESSIONS_ID.put(mappingId, session.getId());

	}                               

	public synchronized Serializable getSessionIDByMappingId(String mappingId) {
        return MANAGED_SESSIONS_ID.get(mappingId);
	}
}
```

**重写jsp权限标签，定义权限过滤逻辑**

>1. 需要自定义shiro.tld,修改对应的部分<%@taglib prefix="shiro" uri="/WEB-INF/views/shiro.tld"%>
2. e：<shiro:hasPermission name="/combo/update">xxx</shiro:hasPermission>" />
3. freemarker模板可以也可以自定义PermissionTag、PrincipalTag、SecureTag

```java
/**
 * 自定义hasPermissionTag标签
 * @author libs
 * 2016-11-3
 */
public class HasPermissionTag extends PermissionTag {

	private static final long serialVersionUID = 1L;

	@Override
	protected boolean showTagBody(String p) {
		return (getSubject() != null) && (getSubject().hasRole(UserAuthority.admin.getValue())||
       		 (getSubject().isPermitted(Enums.Project.OMS.getValue()+Constants.AUTH_SEPARATOR+p.trim())));
	}

}
```

```java
/**
 * 自定义hasAnyPermissionTag标签
 * @author libs
 * 2016-11-2 15:59
 */
public class HasAnyPermissionTag extends PermissionTag {

	private static final long serialVersionUID = 1L;

	@Override
	protected boolean showTagBody(String permissions) {
		 boolean hasAnyPermissions = false;
		 String realUrl = "";
         Subject subject = getSubject();
         if (subject != null) {
             for (String p : permissions.split(",")) {
                 if ((getSubject() != null) && (getSubject().hasRole(UserAuthority.admin.getValue())||
                		 (getSubject().isPermitted(Enums.Project.OMS.getValue()+Constants.AUTH_SEPARATOR+p.trim())))) {
                         hasAnyPermissions = true;
                         realUrl = p;
                         break;
                 }
             }
         }
         pageContext.setAttribute("realUrl", realUrl);
         return hasAnyPermissions;
	}

}
```

### application.properties/yml中添加配置

>management.context-path设置actuator端点context-path，方便在url权限控制中放行

```
redis.host=127.0.0.1
#redis port
redis.port=6379
#redis password
redis.password=1234
#redis expire (seconds)
redis.expire=86400
#session timeout (seconds)
session.timeout=86400000
#cas expiretime(millisecond)
interval.time=3600000
cas.shiro.casServer=http://localhost:9993/cas
cas.shiro.casClient=http://localhost:${server.port}/xxx
management.context-path = /manager
```

**End~**