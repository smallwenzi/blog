title: resttemplate 问题
author: wen
tags:
  - resttemplate
  - spring cloud
categories:
  - spring cloud
date: 2019-05-29 16:38:00
---
最近遇到一个问题：
mq消费 同步调用其他服务，突然不消费mq数据，且mq数据一直堆积

原因：
如果什么都不设置，RestTemplate默认使用的是SimpleClientHttpRequestFactory，其内部使用的是jdk的java.net.HttpURLConnection创建底层连接，默认是没有连接池的，connectTimeout和readTimeout都是 -1，即没有超时时间

解决方案：
http链接池配置
```
 
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
 
@Component
@ConfigurationProperties(prefix = "http-pool")
@Data
public class HttpPoolProperties {
 
    private Integer maxTotal;
    private Integer defaultMaxPerRoute;
    private Integer connectTimeout;
    private Integer connectionRequestTimeout;
    private Integer socketTimeout;
    private Integer validateAfterInactivity;
 
}
```
配置文件
```
http-pool.maxTotal=200
http-pool.defaultMaxPerRoute=100
http-pool.connectTimeout=5000
http-pool.connectionRequestTimeout=1000
http-pool.socketTimeout=600000
http-pool.validateAfterInactivity=2000
```
RestTemplateConfig
```
@Configuration
public class RestTemplateConfig {
 
    @Autowired
    private HttpPoolProperties httpPoolProperties;
 
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate(httpRequestFactory());
    }
 
    @Bean
    public ClientHttpRequestFactory httpRequestFactory() {
        return new HttpComponentsClientHttpRequestFactory(httpClient());
    }
 
    @Bean
    public HttpClient httpClient() {
        Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory>create()
                .register("http", PlainConnectionSocketFactory.getSocketFactory())
                .register("https", SSLConnectionSocketFactory.getSocketFactory())
                .build();
        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager(registry);
        connectionManager.setMaxTotal(httpPoolProperties.getMaxTotal());
        connectionManager.setDefaultMaxPerRoute(httpPoolProperties.getDefaultMaxPerRoute());
        connectionManager.setValidateAfterInactivity(httpPoolProperties.getValidateAfterInactivity());
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(httpPoolProperties.getSocketTimeout()) //服务器返回数据(response)的时间，超过抛出read timeout
                .setConnectTimeout(httpPoolProperties.getConnectTimeout()) //连接上服务器(握手成功)的时间，超出抛出connect timeout
                .setConnectionRequestTimeout(httpPoolProperties.getConnectionRequestTimeout())//从连接池中获取连接的超时时间，超时间未拿到可用连接，会抛出org.apache.http.conn.ConnectionPoolTimeoutException: Timeout waiting for connection from pool
                .build();
        return HttpClientBuilder.create()
                .setDefaultRequestConfig(requestConfig)
                .setConnectionManager(connectionManager)
                .build();
    }
}
```
引用
```
  @Autowired
    private RestTemplate restTemplate;
```