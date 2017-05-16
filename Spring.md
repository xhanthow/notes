# 第一章 Spring之旅

容器是Spring框架的核心

（1）bean工厂`org.springframework.beans.factory.BeanFactory`是最简单的容器，提供基本的DI支持。

（2）应用上下文`org.springframework.context.ApplicationContext `基于BeanFactory构建，并提供应用框架级别的服务。

![bean生命周期](C:\Users\Anthow\Desktop\IMG\spring\bean生命周期.png)





# 第二章 装配bean

@ComponentScan ->`basePackages `或者`basePackageClasses `

> JavaConfig与其他的Java代码又有所区别， 在概念上， 它与应用程序中的业务逻辑和领域代码是不同的。 尽管它与其他的组件一样都使用相同的语言进行表述， 但JavaConfig是配置代码。 这意味着它不应该包含任何业务逻辑， JavaConfig也不应该侵入到业务逻辑代码之中。 尽管不是必须的， 但通常会将JavaConfig放到单独的包中， 使它与其他的应用程序逻辑分离开来， 这样对于它的意图就不会产生困惑了。 



`@Import `->class, `@ImportResource `->xml



> 不管使用JavaConfig还是使用XML进行装配， 我通常都会创建一个根配置（ rootconfiguration） ， 也就是这里展现的这样， 这个配置会将两个或更多的装配类和/或XML文件组合起来。 我也会在根配置中启用组件扫描（ 通过<context:component-scan>或@ComponentScan)。

## 小结##

Spring中装配bean的三种主要方式： 自动化配置、 基于Java的显式配置以及基于XML的显式配置。 







# 第三章 高级装配

>Spring在确定哪个profile处于激活状态时， 需要依赖两个独立的属性： spring.profiles.active和spring.profiles.default。 如果设置了spring.profiles.active属性的话， 那么它的值就会用来确定哪个profile是激活的。 但如果没有设置spring.profiles.active属性的话， 那Spring将会查找spring.profiles.default的值。 如果spring.profiles.active和spring.profiles.default均没有设置的话， 那就没有激活的profile， 因此只会创建那些没有定义在profile中的bean。 



有多种方式来设置这两个属性：

* 作为DispatcherServlet的初始化参数；
* 作为Web应用的上下文参数；
* 作为JNDI条目；
* 作为环境变量；
* 作为JVM的系统属性；
* 在集成测试类上， 使用@ActiveProfiles注解设置。 

> Spring提供了@ActiveProfiles注解， 我们可以使用它来指定运行测试时要激活哪个profile。  



### 条件化的bean

> 设置给@Conditional的类可以是任意实现了Condition接口的类型。 可以看出来， 这个接口实现起来很简单直接， 只需提供matches()方法的实现即可。 如果matches()方法返回true， 那么就会创建带有@Conditional注解的bean。 如果matches()方法返回false， 将不会创建这些bean。 



### bean的作用域

> 使用ConfigurableBeanFactory类的SCOPE_PROTOTYPE常量设置了原型作用域。 你当然也可以使用@Scope("prototype")， 但是使用SCOPE_PROTOTYPE常量更加安全并且不易出错。 



>  购物车bean来说， 会话作用域是最为合适的， 因为它与给定的用户关联性最大。 @Scope同时还有一个proxyMode属性， 它被设置成了ScopedProxyMode.INTERFACES。 这个属性解决了将会话或请求作用域的bean注入到单例bean中所遇到的问题 

```
假设我们要将ShoppingCart bean注入到单例StoreService bean的Setter方法中.因为StoreService是一个单例的bean， 会在Spring应用上下文加载的时候创建。 当它创建的时候， Spring会试图将ShoppingCart bean注入到setShoppingCart()方法中。 但是ShoppingCart bean是会话作用域的， 此时并不存在。 直到某个用户进入系统， 创建了会话之后， 才会出现ShoppingCart实例。另外， 系统中将会有多个ShoppingCart实例： 每个用户一个。 我们并不想让Spring注入某个固定的ShoppingCart实例到StoreService中。 我们希望的是当StoreService处理购物车功能时， 它所使用的ShoppingCart实例恰好是当前会话所对应的那一个。Spring并不会将实际的ShoppingCart bean注入到StoreService中， Spring会注入一个到ShoppingCart bean的代理， 如图所示。 这个代理会暴露与ShoppingCart相同的方法， 所以StoreService会认为它就是一个购物车。 但是， 当StoreService调用ShoppingCart的方法时， 代理会对其进行懒解析并将调用委托给会话作用域内真正的ShoppingCart bean。
```

![作用域代理](C:\Users\Anthow\Desktop\IMG\spring\作用域代理.png)



### 运行时值注入

SpEL







# 第四章 面向切面的Spring

> 因为Spring基于动态代理， 所以Spring只支持方法连接点。 这与一些其他的AOP框架是不同的， 例如AspectJ和JBoss， 除了方法切点， 它们还提供了字段和构造器接入点。 



> 切点用于准确定位应该在什么地方应用切面的通知。 通知和切点是切面的最基本元素。  



* arg() 限制连接点匹配参数为指定类型的执行方法
* @args() 限制连接点匹配参数由指定注解标注的执行方法
* execution() 用于匹配是连接点的执行方法
* this() 限制连接点匹配AOP代理的bean引用为指定类型的类
* target 限制连接点匹配目标对象为指定类型的类
* @target() 限制连接点匹配特定的执行对象， 这些对象对应的类要具有指定类型的注解
* within() 限制连接点匹配指定的类型
* @within() 限制连接点匹配指定注解所标注的类型（ 当使用Spring AOP时， 方法定义在由指定的注解所标注的类里）
* @annotation 限定匹配带有指定注解的连接点 





@DeclareParents注解由三部分组成：

* value属性指定了哪种类型的bean要引入该接口。 


* defaultImpl属性指定了为引入功能提供实现的类。


* @DeclareParents注解所标注的静态属性指明了要引入的接口。 





# 第五章 构建Spring Web应用程序

![springMVC请求过程](C:\Users\Anthow\Desktop\IMG\spring\springMVC请求过程.png)



在请求离开浏览器时 ， 会带有用户所请求内容的信息， 至少会包含请求的URL。 但是还可能带有其他的信息， 例如用户提交的表单信息。请求旅程的第一站是Spring的DispatcherServlet。 与大多数基于Java的Web框架一样，Spring MVC所有的请求都会通过一个前端控制器（ front controller） Servlet。 前端控制器是常用的Web应用程序模式， 在这里一个单实例的Servlet将请求委托给应用程序的其他组件来执行实际的处理。 在Spring MVC中， DispatcherServlet就是前端控制器。DispatcherServlet的任务是将请求发送给Spring MVC控制器（ controller） 。 控制器是一个用于处理请求的Spring组件。 在典型的应用程序中可能会有多个控制器， DispatcherServlet需要知道应该将请求发送给哪个控制器。 所以DispatcherServlet以会查询一个或多个处理器映射（ handler mapping） 来确定请求的下一站在哪里。 处理器映射会根据请求所携带的URL信息来进行决策。一旦选择了合适的控制器， DispatcherServlet会将请求发送给选中的控制器 。 到了控制器， 请求会卸下其负载（ 用户提交的信息） 并耐心等待控制器处理这些信息。 （ 实际上，设计良好的控制器本身只处理很少甚至不处理工作， 而是将业务逻辑委托给一个或多个服务对象进行处理。 ）控制器在完成逻辑处理后， 通常会产生一些信息， 这些信息需要返回给用户并在浏览器上显示。 这些信息被称为模型（ model） 。 不过仅仅给用户返回原始的信息是不够的——这些信息需要以用户友好的方式进行格式化， 一般会是HTML。 所以， 信息需要发送给一个视图（ view） ， 通常会是JSP。控制器所做的最后一件事就是将模型数据打包， 并且标示出用于渲染输出的视图名。 它接下来会将请求连同模型和视图名发送回DispatcherServlet 。这样， 控制器就不会与特定的视图相耦合， 传递给DispatcherServlet的视图名并不直接表示某个特定的JSP。 实际上， 它甚至并不能确定视图就是JSP。 相反， 它仅仅传递了一个逻辑名称， 这个名字将会用来查找产生结果的真正视图。 DispatcherServlet将会使用视图解析器（ view resolver） 来将逻辑视图名匹配为一个特定的视图实现， 它可能是也可能不是JSP。既然DispatcherServlet已经知道由哪个视图渲染结果， 那请求的任务基本上也就完成了。 它的最后一站是视图的实现（ 可能是JSP） ， 在这里它交付模型数据。 请求的任务就完成了。 视图将使用模型数据渲染输出， 这个输出会通过响应对象传递给客户端（ 不会像听上去那样硬编码） 





> 扩展AbstractAnnotationConfigDispatcherServletInitializer的任意类都会自动地配置DispatcherServlet和Spring应用上下文， Spring的应用上下文会位于应用程序的Servlet上下文之中。 

>**直接转发方式（Forward)**，客户端和浏览器只发出一次请求，Servlet、HTML、JSP或其它信息资源，由第二个信息资源响应该请求，在请求对象request中，保存的对象对于每个信息资源是共享的。
>
>**间接转发方式（Redirect）**实际是两次HTTP请求，服务器端在响应第一次请求的时候，让浏览器再向另外一个URL发出请求，从而达到转发的目的。
>
>举个通俗的例子：
>
>直接转发就相当于：“A找B借钱，B说没有，B去找C借，借到借不到都会把消息传递给A”
>
>**　　间接转发就相当于："A找B借钱，B说没有，让A去找C借"。**







# 第六章 渲染Web视图

>使用名为InternalResourceViewResolver的视图解析器。 在它的配置中， 为了得到视图的名字， 会使用“WEB-INF/views/”前缀和“.jsp”后缀， 从而确定来渲染模型的JSP文件的物理位置。  

> 如果想让InternalResourceViewResolver将视图解析为JstlView， 而不是InternalResourceView的话， 那么我们只需设置它的viewClass属性即可 

```java
	// 配置jsp试图解析器
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/views/");
		resolver.setSuffix(".jsp");
		resolver.setViewClass(JstlView.class);
		return resolver;
	}
```



为了要在Spring中使用Thymeleaf， 我们需要配置三个启用Thymeleaf与Spring集成的bean：

* ThymeleafViewResolver： 将逻辑视图名称解析为Thymeleaf模板视图；
* SpringTemplateEngine： 处理模板并渲染结果；
* TemplateResolver： 加载Thymeleaf模板。 






# 第七章 SpringMVC的高级技术

```java

	@Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
		registration.setMultipartConfig(new MultipartConfigElement("xxx"));
	}
```



> 如果我们想往Web容器中注册其他组件的话， 只需创建一个新的初始化器就可以了。最简单的方式就是实现Spring的WebApplicationInitializer接口。 

```java
public class MyServletInitializer implements WebApplicationInitializer{
	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		servletContext.addFilter("xx", xx.class);
      	// 添加过滤器的映射路径
		servletContext.addServlet("xx", xx.class);
        // 添加servlet的映射路径
	}
}

```



> 在典型的Spring MVC应用中， 我们会需要DispatcherServlet和ContextLoaderListener。 AbstractAnnotationConfigDispatcherServletInitializer会自动注册它们 



> ContextLoaderListener和DispatcherServlet各自都会加载一个Spring应用上下文。 上下文参数contextConfigLocation指定了一个XML文件的地址， 这个文件定义了根应用上下文， 它会被ContextLoaderListener加载。 



> WebApplicationInitializer      AnnotationConfigDispatcherServletInitializer  
>
> [关系介绍](http://joshlong.com/jl/blogPost/simplified_web_configuration_with_spring.html)
>
> ![继承关系](C:\Users\Anthow\Desktop\IMG\spring\继承关系.png)

```java
// this class will be scanned by Spring on application startup and bootstrapped
public class AppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        WebApplicationContext context = getContext();
        servletContext.addListener(new ContextLoaderListener(context));
        ServletRegistration.Dynamic dispatcher = servletContext.addServlet("DispatcherServlet", new DispatcherServlet(context));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/*");
    }

    private AnnotationConfigWebApplicationContext getContext() {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.setConfigLocation("eu.kielczewski.example.config");
        return context;
    }

}
```



```java
@Configuration
@ComponentScan(basePackages = "eu.kielczewski.example")
public class AppConfig {
}
```

```java
/**
 WebMvcConfig class enables Spring MVC with @EnableWebMvc annotation. It extends WebMvcConfigurerAdapter, which provides empty methods 	  that can be overridden to customize default configuration of Spring MVC 
*/
@EnableWebMvc
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
}
```





异常处理:

> @ResponseStatus  @ExceptionHandler 





控制器通知。 控制器通知（controlleradvice） 是任意带有@ControllerAdvice注解的类， 这个类会包含一个或多个如下类型的方法：

* @ExceptionHandler注解标注的方法；
* @InitBinder注解标注的方法；
* @ModelAttribute注解标注的方法。 

> 在带有@ControllerAdvice注解的类中， 以上所述的这些方法会运用到整个应用程序所有控制器中带有@RequestMapping注解的方法上。 



> @ControllerAdvice最为实用的一个场景就是将所有的@ExceptionHandler方法收集到一个类中， 这样所有控制器的异常就能在一个地方进行一致的处理。 

![ControllerAdvice](C:\Users\Anthow\Desktop\IMG\spring\ControllerAdvice.png)



> RedirectAttributes提供了一组addFlashAttribute()方法来添加flash属性。 

![RedirectAttributes](C:\Users\Anthow\Desktop\IMG\spring\RedirectAttributes.png)





# 第九章 保护web应用

> DelegatingFilterProxy是一个特殊的Servlet Filter， 它本身所做的工作并不多。 只是将工作委托给一个javax.servlet.Filter实现类， 这个实现类作为一个<bean>注册在Spring应用的上下文中。 AbstractSecurityWebApplicationInitializer实现了WebApplicationInitializer， 因此Spring会发现它， 并用它在Web容器中注册DelegatingFilterProxy。  尽管我们可以重载它的appendFilters()或insertFilters()方法来注册自己选择的Filter， 但是要注册DelegatingFilterProxy的话， 我们并不需要重载任何方法。 

> 不管我们通过web.xml还是通过AbstractSecurityWebApplicationInitializer的子类来配置DelegatingFilterProxy， 它都会拦截发往应用中的请求， 并将请求委托给ID为springSecurityFilterChain bean 。



### 使用基于内存的用户存储 

> 因为我们的安全配置类扩展了WebSecurityConfigurerAdapter， 因此配置用户存储的最简单方式就是重载configure()方法， 并以AuthenticationManagerBuilder作为传入参数。 AuthenticationManagerBuilder有多个方法可以用来配置Spring Security对认证的支持。 通过inMemoryAuthentication()方法， 我们可以启用、 配置并任意填充基于内存的用户存储。

```java
/**
 * Add this annotation to an {@code @Configuration} class to have the Spring Security
 * configuration integrate with Spring MVC.
 *
 * @deprecated Use EnableWebSecurity instead which will automatically add the Spring MVC
 * related Security items.
 * @author Rob Winch
 * @since 3.2
 */
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
      // 需要注意的是， roles()方法是authorities()方法的简写形式。 roles()方法所给定
	  //的值都会添加一个“ROLE_”前缀
		auth.inMemoryAuthentication()
				.withUser("user").password("123").roles("USER").and()
				.withUser("admin").password("321").roles("USER", "ADMIN").and()
          		.withUser("test").password("abc").authorities("ROLE_USER");
	}
}

```



### 配置自定义的用户服务 

![自定义权限验证](C:\Users\Anthow\Desktop\IMG\spring\自定义权限验证.png)



![自定义验证的使用](C:\Users\Anthow\Desktop\IMG\spring\自定义验证的使用.png)







> 对每个请求进行细粒度安全性控制的关键在于重载configure(HttpSecurity)方法。 

```java
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests()
				.antMatchers("url").authenticated()
				.antMatchers(HttpMethod.POST, "/url").authenticated()
				// SpEL表达式
				.antMatchers("url").access("hasRole('USER') AND hasIpAddress('xxx')")
				.anyRequest().permitAll()
				.and()
				.requiresChannel()
				// 需要https
				.antMatchers("url").requiresSecure();
	}
```





# 第十章 通过spring和JDBC征服数据库

>  Spring将数据访问过程中固定的和可变的部分明确划分为两个不同的类： 模板（ template） 和回调（ callback） 。 模板管理过程中固定的部分， 而回调处理自定义的数据访问代码。 

![数据层模板模式](C:\Users\Anthow\Desktop\IMG\spring\数据层模板模式.png)

> 如图所示， Spring的模板类处理数据访问的固定部分——事务控制、 管理资源以及处理异常。 同时， 应用程序相关的数据访问——语句、 绑定参数以及整理结果集——在回调的实现中处理。 事实证明， 这是一个优雅的架构， 因为你只需关心自己的数据访问逻辑即可。 