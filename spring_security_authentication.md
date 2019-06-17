### AuthenticationManager

该对象提供了认证方法的入口，接收一个Authentiaton对象作为参数;

    public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
    }
  
### ProviderManager

它是 AuthenticationManager 的一个实现类，提供了基本的认证逻辑和方法；它包含了一个 List<AuthenticationProvider> 对象，
通过 AuthenticationProvider 接口来扩展出不同的认证提供者
(当Spring Security默认提供的实现类不能满足需求的时候可以扩展AuthenticationProvider 覆盖supports(Class<?> authentication) 方法)；

### 验证逻辑
AuthenticationManager 接收 Authentication 对象作为参数，并通过 authenticate(Authentication) 方法对其进行验证:

    authenticationManager.authenticate()
    
AuthenticationProvider实现类用来支撑对 Authentication 对象的验证动作；
UsernamePasswordAuthenticationToken实现了 Authentication主要是将用户输入的用户名和密码进行封装，并供给 AuthenticationManager 进行验证；
验证完成以后将返回一个认证成功的 Authentication 对象:

    authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(loginVisitor.getEmail(), loginVisitor.getPassword(), new ArrayList<>())
            )
      
### Authentication

Authentication对象中的主要方法

    public interface Authentication extends Principal, Serializable {
	//#1.权限结合，可使用AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_ADMIN")返回字符串权限集合
	Collection<? extends GrantedAuthority> getAuthorities();
	//#2.用户名密码认证时可以理解为密码
	Object getCredentials();
	//#3.认证时包含的一些信息。
	Object getDetails();
	//#4.用户名密码认证时查询到的userDetails对象
	Object getPrincipal();
	#5.是否被认证，认证为true	
	boolean isAuthenticated();
	#6.设置是否能被认证
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
    }
    
    
