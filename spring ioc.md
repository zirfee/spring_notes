

### ioc过程大概分为以下几步

1. 解析bean配置

2. 解析后的bean类信息封装成beanDefinition对象放到到map集合里

3. 容器启动时,从map里找到对应的beanDefinition利用反射new出对象

   > 如果对象是单例,就把单例对象放到另一个map集合,便于复用,同时防止GC回收
   
   > 多例则直接抛出对象,失去引用时就会被回收
   
4. 通过settter/getter方法注入bean对象


5. 之后如果还需要注入该bean,如果bean是单例,直接返回bean对象;如果是多例,new一个新对象返回

#### 一些细节
- 解析bean时可以是xml或注解
- 单例对象scope是session则放到session里
- 对于beanFactory,对象默认懒加载,只有需要一个bean时,框架才从map里找到对应的beanDefinition利用反射new出对象

  对于applicationContext,容器启动时会马上实例化所有非懒加载的bean
