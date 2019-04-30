[toc]

> 　　有时候我们在一些独立的的线程中会用到受 `Spring` 管理的类，比如一条专门用来发短信、增加积分的线程。这时候不应该在重新 new 一个新对象进行使用，一方面已经有被 Spring 管理了，再 new 的话比较浪费。另一方面如果单纯的 new 一个对象，就不能使用这个类里面注入的 Spring 管理的 Bean 进行操作。

# 使用ApplicationContextAware接口

　　`ApplicationContextAware`是 Spring 官方提供的一个用来解决如下类似问题的接口：

* 但在某些特殊的情况下，`Bean` 需要实现某个功能，但该功能必须借助于 Spring 容器才能实现，此时就必须让该Bean先获取 `Spring 容器`，然后借助于Spring容器实现该功能。为了让 Bean 获取它所在的 Spring 容器，可以让该Bean实现 `ApplicationContextAware` 接口。

　　如下代码，实现了 `ApplicationContextAware` 接口，并提供了几种获取 Spring容器中 Bean 的功能：

```
@Component
public class SpringContextHandler implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        SpringContextHandler.applicationContext = context;
    }

    /**
     * 获取applicationContext对象
     */
    public static ApplicationContext getApplicationContext() {
        assertContextInjected();
        return applicationContext;
    }

    /**
     * 根据bean的id来查找对象
     *
     * @param name bean名称
     * @return bean
     */
    public static <T> T getBeanByName(String name) {
        assertContextInjected();
        return (T) applicationContext.getBean(name);
    }

    /**
     * 根据bean的class来查找对象
     *
     * @param c class对象
     * @return bean
     */
    public static <T> T getBeanByClass(Class c) {
        assertContextInjected();
        return (T) applicationContext.getBean(c);
    }

    /**
     * 根据bean的class来查找所有的对象(包括子类)
     *
     * @param c class对象
     * @return bean Map
     */
    public static Map getBeansByClass(Class c) {
        assertContextInjected();
        return applicationContext.getBeansOfType(c);
    }

    /**
     * 检查ApplicationContext不为空.
     */
    private static void assertContextInjected() {
        Validate.validState(applicationContext != null,
                "applicaitonContext属性未注入, 请把SpringContextHandler交由Spring管理");
    }

}
```

　　工具类完成，需要在非Spring管理的类中使用Spring容器管理的Bean时即可使用此工具进行获取。
