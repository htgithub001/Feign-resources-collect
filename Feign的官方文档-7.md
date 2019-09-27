# 7. Declarative REST Client: Feign

原文链接: https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html


Feign is a declarative web service client. It makes writing web service clients easier. 
<b>To use Feign create an interface and annotate it</b>. 
It has pluggable annotation support including Feign annotations and JAX-RS annotations. 
Feign also supports pluggable encoders and decoders. 
Spring Cloud adds support for Spring MVC annotations and for using the same HttpMessageConverters used by default in Spring Web. 
<b>Spring Cloud integrates Ribbon and Eureka to provide a load balanced http client when using Feign</b>.

## 7.1 How to Include Feign

To include Feign in your project use the starter with group org.springframework.cloud and artifact id spring-cloud-starter-openfeign. 
See the Spring Cloud Project page for details on setting up your build system with the current Spring Cloud Release Train.

Example spring boot app
```
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

StoreClient.java. 

```
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

In the @FeignClient annotation the String value ("stores" above) is an <b>arbitrary client name, 
which is used to create a Ribbon load balancer (see below for details of Ribbon support). </b>
You can also specify a URL using the url attribute (absolute value or just a hostname). 
The name of the bean in the application context is the fully qualified name of the interface. 
To specify your own alias value you can use the qualifier value of the @FeignClient annotation.

## 7.3 Creating Feign Clients Manually

In some cases it might be necessary to customize your Feign Clients in a way that is not possible using the methods above. 
In this case you can create Clients using the Feign Builder API. Below is an example which creates two Feign Clients 
with the same interface but configures each one with a separate request interceptor.

```
@Import(FeignClientsConfiguration.class)
class FooController {

	private FooClient fooClient;

	private FooClient adminClient;

    	@Autowired
	public FooController(
			Decoder decoder, Encoder encoder, Client client, Contract contract) {
		this.fooClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
                .contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
				.target(FooClient.class, "http://PROD-SVC");
		this.adminClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
				.target(FooClient.class, "http://PROD-SVC");
    }
}
```


In the above example FeignClientsConfiguration.class is the default configuration provided by Spring Cloud Netflix.

PROD-SVC is the name of the service the Clients will be making requests to.

The Feign Contract object defines what annotations and values are valid on interfaces. The autowired Contract bean provides supports for SpringMVC annotations, instead of the default Feign native annotations.

## 7.4 Feign Hystrix Support

If Hystrix is on the classpath and feign.hystrix.enabled=true, Feign will wrap all methods with a circuit breaker. 
Returning a com.netflix.hystrix.HystrixCommand is also available. 
This lets you use reactive patterns (with a call to .toObservable() or .observe() or asynchronous use (with a call to .queue()).

To disable Hystrix support on a per-client basis create a vanilla Feign.Builder with the "prototype" scope, e.g.:
```
@Configuration
public class FooConfiguration {
    	@Bean
	@Scope("prototype")
	public Feign.Builder feignBuilder() {
		return Feign.builder();
	}
}
```

注意:Prior to the Spring Cloud Dalston release, if Hystrix was on the classpath Feign would have wrapped all methods in a circuit breaker by default. This default behavior was changed in Spring Cloud Dalston in favor for an opt-in approach.


