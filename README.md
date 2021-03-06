
[![CircleCI](https://circleci.com/gh/Playtika/feign-reactive/tree/master.svg?style=shield&circle-token=d229579db6903be702f2416a357d1a01fb5c5fc0)](https://circleci.com/gh/Playtika/feign-reactive/tree/master)
[![codecov](https://codecov.io/gh/Playtika/feign-reactive/branch/master/graph/badge.svg)](https://codecov.io/gh/Playtika/feign-reactive)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/9bf7bea79a2b4048bf3ed8fa247092d1)](https://www.codacy.com/app/alekseibevzenko/testcontainers-spring-boot?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Playtika/testcontainers-spring-boot&amp;utm_campaign=Badge_Grade)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.playtika.reactivefeign/feign-reactor/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.playtika.reactivefeign/feign-reactor)


# feign-reactive

[ ![Download](https://api.bintray.com/packages/kptfh/feign-reactive/client/images/download.svg) ](https://bintray.com/kptfh/feign-reactive/client/_latestVersion)

Use Feign with Spring WebFlux

## Overview

Implementation of Feign on Spring WebClient. Brings you the best of two worlds together : 
concise syntax of Feign to write client side API on fast, asynchronous and
non-blocking HTTP client of Spring WebClient.

## Modules
  
  **_feign-reactor-core_** : base classes and interfaces that should allow to implement alternative reactor Feign
  
  **_feign-reactor-webclient_** : Spring WebClient based implementation of reactor Feign 
  
  **_feign-reactor-cloud_** : Spring Cloud implementation of reactor Feign (Ribbon/Hystrix)
  
  **_feign-reactor-rx2_** : Rx2 compatible implementation of reactor Feign (depends on feign-reactor-webclient)

## Usage

Write Feign API as usual, but every method of interface
 - may accept `org.reactivestreams.Publisher` as body
 - must return `reactor.core.publisher.Mono` or `reactor.core.publisher.Flux`.

```java
@Headers({ "Accept: application/json" })
public interface IcecreamServiceApi {

  @RequestLine("GET /icecream/flavors")
  Flux<Flavor> getAvailableFlavors();

  @RequestLine("GET /icecream/mixins")
  Flux<Mixin> getAvailableMixins();

  @RequestLine("POST /icecream/orders")
  @Headers("Content-Type: application/json")
  Mono<Bill> makeOrder(IceCreamOrder order);

  @RequestLine("GET /icecream/orders/{orderId}")
  Mono<IceCreamOrder> findOrder(@Param("orderId") int orderId);

  @RequestLine("POST /icecream/bills/pay")
  @Headers("Content-Type: application/json")
  Mono<Void> payBill(Publisher<Bill> bill);
}
```
Build the client :

```java

/* Create instance of your API */
IcecreamServiceApi client = ReactiveFeign
    .builder()
    .target(IcecreamServiceApi.class, "http://www.icecreame.com")

/* Execute nonblocking requests */
Flux<Flavor> flavors = icecreamApi.getAvailableFlavors();
Flux<Mixin> mixins = icecreamApi.getAvailableMixins();
```

or cloud aware client :

```java
 IcecreamServiceApi client = CloudReactiveFeign.<IcecreamServiceApi>builder()
    .setFallback(new TestInterface() {
        @Override
        public Mono<String> get() {
            return Mono.just("fallback");
        }
    })
    .setLoadBalancerCommand(
         LoadBalancerCommand.builder()
                 .withLoadBalancer(AbstractLoadBalancer.class.cast(getNamedLoadBalancer(serviceName)))
                 .withRetryHandler(new DefaultLoadBalancerRetryHandler(1, 1, true))
                 .build()
    )
    .target(IcecreamServiceApi.class, "http://" + serviceName);

/* Execute nonblocking requests */
Flux<Flavor> flavors = icecreamApi.getAvailableFlavors();
Flux<Mixin> mixins = icecreamApi.getAvailableMixins();
```

## Rx2 Usage 

Write Feign API as usual, but every method of interface
 - may accept `Flowable`, `Observable`, `Single` or `Maybe` as body
 - must return `Flowable`, `Observable`, `Single` or `Maybe`.

```java
@Headers({"Accept: application/json"})
public interface IcecreamServiceApi {

  @RequestLine("GET /icecream/flavors")
  Flowable<Flavor> getAvailableFlavors();

  @RequestLine("GET /icecream/mixins")
  Observable<Mixin> getAvailableMixins();

  @RequestLine("POST /icecream/orders")
  @Headers("Content-Type: application/json")
  Single<Bill> makeOrder(IceCreamOrder order);

  @RequestLine("GET /icecream/orders/{orderId}")
  Maybe<IceCreamOrder> findOrder(@Param("orderId") int orderId);

  @RequestLine("POST /icecream/bills/pay")
  @Headers("Content-Type: application/json")
  Single<Long> payBill(Bill bill);
```
Build the client :

```java

/* Create instance of your API */
IcecreamServiceApi client = Rx2ReactiveFeign
    .builder()
    .target(IcecreamServiceApi.class, "http://www.icecreame.com")

/* Execute nonblocking requests */
Flowable<Flavor> flavors = icecreamApi.getAvailableFlavors();
Observable<Mixin> mixins = icecreamApi.getAvailableMixins();
```

## Maven

```xml
<repositories>
    <repository>
        <id>bintray-kptfh-feign-reactive</id>
        <name>bintray</name>
        <url>https://dl.bintray.com/kptfh/feign-reactive</url>
    </repository>
</repositories>
...
<dependencies>
    ...
    
    <dependency>
        <groupId>io.github.reactivefeign</groupId>
        <artifactId>feign-reactor-cloud</artifactId>
        <version>1.0.0</version>
    </dependency>
    
    or if you don't need cloud specific functionality
    
    <dependency>
        <groupId>io.github.reactivefeign</groupId>
        <artifactId>feign-reactor-webclient</artifactId>
        <version>1.0.0</version>
    </dependency>
    
    or if you tend to use Rx2 interfaces
    
    <dependency>
            <groupId>io.github.reactivefeign</groupId>
            <artifactId>feign-reactor-rx2</artifactId>
            <version>1.0.0</version>
        </dependency>
    ...
</dependencies>
```

## License

Library distributed under Apache License Version 2.0.
