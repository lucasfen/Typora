## 自动装配

1. @SpringBootApplication注解

   大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

   - `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
   - `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
   - `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。

2. @EnableAutoConfiguration注解

3. `AutoConfigurationImportSelector`类

   - 判断自动装配开关是否打开
   - 获取`EnableAutoConfiguration`注解中的 `exclude` 和 `excludeName`
   - 获取需要自动装配的所有配置类，读取`META-INF/spring.factories`

4. @ConditionalOnXXX

   筛选满足条件的类进行加载

**总结**：

Spring Boot 通过`@EnableAutoConfiguration`开启自动装配，通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配，自动配置类其实就是通过`@Conditional`按需加载的配置类，想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖