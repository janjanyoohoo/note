# WebService接口

- 基于CXF创建WeebService服务

```java
package org.platform.sms.api.config;

import org.apache.cxf.Bus;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.platform.sms.api.service.SmsService;
import org.platform.sms.api.service.impl.SmsServiceImpl;
import org.apache.cxf.bus.spring.SpringBus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.platform.sms.api.service.impl.WsReceiveServiceImpl;
import org.platform.sms.api.service.impl.WsReceiveUserServieImpl;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

import javax.xml.ws.Endpoint;

/**
 * webservice注入配置
 */
@Configuration
public class WsConfig {
	//声明cxf的servlet,配置映射的路径
    @Bean(name = "cxfServlet")
    public ServletRegistrationBean dispatcherServlet() {
        return new ServletRegistrationBean(new CXFServlet(), "/ws/*");
    }
	//声明SpringBus的bean,服务发布用
    @Bean(name = Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {
        return new SpringBus();
    }
    //声明服务提供类的bean
    @Bean
    public WsReceiveServiceImpl wsReceiveService(){
        return new WsReceiveServiceImpl();
    }
    //声明服务提供类的bean
    @Bean
    public WsReceiveUserServieImpl wsReceiveServiceUser(){
        return new WsReceiveUserServieImpl();
    }


    /**
     * 发布服务
     */
    @Bean
    public Endpoint endpointReceive() {
        EndpointImpl endpoint = new EndpointImpl(springBus(), wsReceiveService());
        endpoint.publish("/sms");
        return endpoint;
    }

    /**
     * 发布服务
     */
    @Bean
    public Endpoint endpointReceiveUser() {
        EndpointImpl endpoint = new EndpointImpl(springBus(), wsReceiveServiceUser());
        endpoint.publish("/sms/user");
        return endpoint;
    }
}

```



