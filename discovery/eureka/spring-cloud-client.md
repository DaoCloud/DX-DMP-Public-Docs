### Using Service Discovery in Spring Boot & Spring Cloud

### Using RestTemplate

```java
@Component
class RestTemplateExample implements CommandLineRunner {

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public void run(String... strings) throws Exception {
        ResponseEntity<List<Bookmark>> exchange =
                this.restTemplate.exchange(
                        // The bookmark-service here will be converted to the real IP:PORT when it runs
                        "http://bookmark-service/{userId}/bookmarks",
                        HttpMethod.GET,
                        null,
                        new ParameterizedTypeReference<List<Bookmark>>() {
                        },
                        (Object) "mstine");

        exchange.getBody().forEach(System.out::println);
    }

}
```

### Using FeignClient

`FeignClient` is an **declarative** `HTTP Client` provided by `Spring`. Refer to [Appendix 1](# Appendix) for details

```java
// The bookmark-service here will be converted to the real IP:PORT when it runs
@FeignClient("bookmark-service")
interface BookmarkClient {

    @RequestMapping(method = RequestMethod.GET, value = "/{userId}/bookmarks")
    List<Bookmark> getBookmarks(@PathVariable("userId") String userId);
}
```

### Using DiscoveryClient

You can also use the `Eureka Client` provided by `Spring` to get the service address and other information directly from the registered services.

```java
@Component
class DiscoveryClientExample implements CommandLineRunner {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Override
    public void run(String... strings) throws Exception {
        discoveryClient.getInstances("bookmark-service").forEach((ServiceInstance s) -> {
            // The ServiceInstance here is the real instance object
            // the port number of the service instance
            int port = s.getPort();
            // The address of the service instance (Hostname/IP) is determined by the registered address
            String host = s.getHost();
            System.out.println(ToStringBuilder.reflectionToString(s));
        });
    }
}
```

## Appendix

1. [Declarative REST Client: Feign](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html)
