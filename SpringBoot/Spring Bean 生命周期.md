

Spring Bean 的生命周期：实例化、属性赋值、初始化、销毁

- [ ] 实例化 `BeanFactoryPostProcessor `实现类，执行`postProcessBeanFactory `方法
- [ ] 实例化`BeanPostProcessor `实现类
- [ ] 实例化`InstantiationAwareBeanPostProcessorAdapter `，执行 `postProcessBeforeInstantiation `方法
- [ ] 执行bean的构造器
- [ ] 执行`InstantiationAwareBeanPostProcessor `的 `postProcessPropertyValues `方法
- [ ] 为`Bean`注入属性
- [ ] 调用 `BeanNameAware `的 `setBeanName `方法
- [ ] 调用 `BeanFactoryAware `的 `setBeanFactory `方法
- [ ] 执行 `BeanPostProcessor `的 `postProcessBeforeInitialization `方法
- [ ] 调用实体类中标注 `@PostConstruct` 的 init 方法
- [ ] 执行 `InitializingBean `的 `afterPropertiesSet `方法
- [ ] 调用`<bean>`的 `init-method` 指定的方法方法
- [ ] 执行 `BeanPostProcessor `的 `postProcessAfterInitialization `方法
- [ ] 执行 `InstantiationAwareBeanPostProcessor `的 `postProcessAfterInstantiation `方法
- [ ] 容器初始化成功，业务逻辑调用，销毁容器
- [ ] 调用实体类中标注 `@PreDestroy `的 destroy 方法
- [ ] 执行 `DisposableBean `的 `destroy `方法
- [ ] 调用`<bean>`的` destroy-method` 指定的方法方法



![](https://images0.cnblogs.com/i/580631/201405/181453414212066.png)

![](https://images0.cnblogs.com/i/580631/201405/181454040628981.png)