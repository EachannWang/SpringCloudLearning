# SpringCloudLearning
SpringCloudLearning.Finchley
项目为参考博客实操 https://blog.csdn.net/zhengzizhi/article/details/81113085
consul 服务发现集群 docker部署 https://blog.csdn.net/fenglailea/article/details/79098246

1.eureka-server（服务注册中心）
    创建服务注册中心（使用idea2018.1版创建项目之后/选择new model/选择Spring Initializr/选择Cloud Discovery/选择erreka server快速创建，以下类似）
    需要在EurekaServerApplication类上加@EnableEurekaServer注解
    通过eureka.client.registerWithEureka=false和eureka.client.fetchRegistry=false来表明自己的身份是一个eureka server
    eureka是一个高可用组件，每一个实例注册之后需想注册中心发送心跳
2.service-hi（服务提供者）
    向服务注册中心注册服务
    需要在ServiceHiApplication类上加@EnableEurekaClient注解
    指明服务注册中心地址eureka.client.serviceUrl.defaultZone: http://localhost:8761/eureka/
3.service-ribbon（服务消费者）
    在工程的启动类中,通过@EnableDiscoveryClient向服务中心注册
    并且向程序的ioc注入一个bean:restTemplate;并通过@LoadBalanced注解表明这个restRemplate开启负载均衡的功能
    public String hiService(String name) {
            return restTemplate.getForObject("http://service-hi/hi?name="+name,String.class);
    }
4.service-feign（服务消费者）
    a.Feign 采用的是基于接口的注解
    b.Feign 整合了ribbon，具有负载均衡的能力
    c.整合了Hystrix，具有熔断的能力
    在程序的启动类ServiceFeignApplication ，加上@EnableFeignClients注解开启Feign的功能
    定义一个feign接口，通过@FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口，代码如下
    @FeignClient(value = "service-hi")
    public interface SchedualServiceHi {
        @RequestMapping(value = "/hi",method = RequestMethod.GET)
        String sayHiFromClientOne(@RequestParam(value = "name") String name);
    }
5.Hystrix（断路器）
    a.在3的基础上在程序的启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix
        改造HelloService类，在hiService方法上加上@HystrixCommand注解。
        该注解对该方法创建了熔断器的功能，并指定了 fallbackMethod熔断方法
    b.在4的基础上  
        @FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
        public interface SchedualServiceHi {
            @RequestMapping(value = "/hi",method = RequestMethod.GET)
            String sayHiFromClientOne(@RequestParam(value = "name") String name);
        }

6.service-zuul（路由网关）
    Zuul的主要功能是路由转发和过滤器。
    路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能
7.分布式配置中心
    config-server
    config-client
    SpringCloudConfig（git上静态文件配置）
    erreka-config-server（高可用配置注册中心）
8.消息总线
    在config-client基础上修改
    