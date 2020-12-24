## SpringBoot 多数据源

> 基于Spring `AbstractRoutingDataSource` + `AOP`

1. 配置DataSource

   - 修改配置文件

   ```
   spring:
    datasource:
      dba:
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://127.0.0.1:3306/dba?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 1234
      dbb:
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://127.0.0.1:3306/dbb?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: 1234
   ```

   - 实现 `AbstractRoutingDataSource`

   ```
   public class DynamicDataSource extends AbstractRoutingDataSource {

     private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();

     public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
        super.setDefaultTargetDataSource(defaultTargetDataSource);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
     }

     @Override
     protected Object determineCurrentLookupKey() {
         return CONTEXT_HOLDER.get();
     }

     public static void setDataSource(String dataSource) {
         CONTEXT_HOLDER.set(dataSource);
     }

     public static void clearDataSource() {
         CONTEXT_HOLDER.remove();
     }
   }
   ```

   - 配置多数据源

   ```
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.dba")
    public DataSource dba() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.dbb")
    public DataSource dbb() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public DataSource dataSource(@Autowired DataSource dba, @Autowired DataSource dbb) throws Exception {
        Map<Object, Object> targetDataSources = new HashMap<>(5);
        targetDataSources.put("dba", dba);
        targetDataSources.put("dbb", dbb);
        return new DynamicDataSource(dba, targetDataSources);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(@Autowired DataSource dataSource) throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapping/*.xml"));
        return sqlSessionFactoryBean.getObject();
    }
   ```

   - 使用AOP配置动态数据源

   ```
   @Aspect
   @Component
   public class DataSourceAspect {

       @Around("execution(* com.test.service.*.insert*(..))")
       public Object around(ProceedingJoinPoint point) throws Throwable {
           DynamicDataSource.setDataSource("dba");
           try {
               return point.proceed();
           } finally {
               DynamicDataSource.clearDataSource();
           }
       }

       @Around("execution(* com.test.service.*.select*(..))")
       public Object aroundBbb(ProceedingJoinPoint point) throws Throwable {
           DynamicDataSource.setDataSource("dbb");
           try {
               return point.proceed();
           } finally {
               DynamicDataSource.clearDataSource();
           }
      }
   }
   ```

   - 移除SpringBoot自动配置数据源

   ```
   @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
   ```