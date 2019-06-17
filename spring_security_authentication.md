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
    
### 核心组件
SecurityContextHolder
SecurityContextHolder是spring security最基本的组件。用于存储安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限等这些都被保存在SecurityContextHolder中。SecurityContextHolder默认是使用ThreadLocal实现的，这样就保证了本线程内所有的方法都可以获得SecurityContext对象。

可以通此方法过来获取当前操作用户信息：

SecurityContextHolder.getContext().getAuthentication().getPrincipal();
默认返回的对象是UserDetails实例，其中包含了username，password和权限等信息，当然，我们也可以通过实现这个接口自定义我们自己的UserDetails实例，给我们自己的应用使用，以符合需要的业务逻辑。比如下面只对token进行操作就可以吧token作为属性放入UserDetails实现类中。

### Authentication
Authentication是Spring Security方式的认证主体。

<1> Authentication是spring security包中的接口，直接继承自Principal类，而Principal是位于java.security包中的。可以见得，Authentication在spring security中是最高级别的身份/认证的抽象。
<2> 由这个顶级接口，我们可以得到用户拥有的权限信息列表，密码，用户细节信息，用户身份信息，认证信息。
authentication.getPrincipal()返回了一个Object，我们将Principal强转成了Spring Security中最常用的UserDetails，这在Spring Security中非常常见，接口返回Object，使用instanceof判断类型，强转成对应的具体实现类。接口详细解读如下：

getAuthorities()，权限信息列表，默认是GrantedAuthority接口的一些实现类，通常是代表权限信息的一系列字符串。
getCredentials()，密码信息，用户输入的密码字符串，在认证过后通常会被移除，用于保障安全。
getDetails()，细节信息，web应用中的实现接口通常为 WebAuthenticationDetails，它记录了访问者的ip地址和sessionId的值。
getPrincipal()，最重要的身份信息，大部分情况下返回的是UserDetails接口的实现类，也是框架中的常用接口之一。
AuthenticationManager
AuthenticationManager（接口）是认证相关的核心接口，也是发起认证的出发点，因为在实际需求中身份认证的方式有多种，一般不使用AuthenticationManager，而是使用AuthenticationManager的实现类ProviderManager ,ProviderManager内部会维护一个List<AuthenticationProvider>列表，存放多种认证方式，实际上这是委托者模式的应用（Delegate）。也就是说，核心的认证入口始终只有一个：AuthenticationManager，不同的认证方式对应不同的AuthenticationProvider。

总结：

SecurityContextHolder：存放身份信息的容器

Authentication：用户信息的抽象

AuthenticationManager：身份认证器

### 认证流程
1、通过过滤器过滤到用户请求的接口，获取身份信息（假如有多个认证方式会配置provider的顺序）

2、一般将身份信息封装到封装成Authentication下的实现类UsernamePasswordAuthenticationToken中

3、通过AuthenticationManager 身份管理器（通过配置找到对应的provider）负责验证这个UsernamePasswordAuthenticationToken

4、认证成功后（认证逻辑一般在service中），AuthenticationManager身份管理器返回一个被填充满了信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除）Authentication实例。

5、SecurityContextHolder安全上下文容器将第2步填充了信息的UsernamePasswordAuthenticationToken，通过SecurityContextHolder.getContext().setAuthentication(…)方法，设置到其中来建立安全上下文（security context)。
