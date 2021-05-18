# Introduction to Service Registration and Discovery

This chapter focuses on how to implement the registration and discovery of services.

## Versions

- Platform built-in Spring Cloud Netflix Eureka Server: `2.0.1.RELEASE`
- Client Spring Cloud Starter Netflix Eureka Client: `>= 1.4.1.RELEASE`

## Introduction

Generally speaking, service registration and discovery include two parts, one is the Server and the other is the Client. Server is a public service that provides service registration and discovery functions for the Client, maintains information about the Client registered to itself, and at the same time provides an interface for the Client to obtain information about other services in the registry, so that the dynamically changing Client can invoke between services. The Client registers its own service information to the Server in a certain way and maintains its own information consistency within the normal scope to facilitate other services to discover itself, and at the same time, it can get information of other services it depends on through the Server to complete service invocation.

Spring Cloud Netflix Eureka is a basic component provided by Spring Cloud for service discovery and registration, and is one of the prerequisites for building Spring Cloud microservices architecture. This allows developers to focus more on the business logic and accelerate the implementation of the service architecture and project development.

In Netfilx, Eureka is a RESTful-style service registration and discovery base service component. The Eureka Client regularly registers itself with the Eureka Server and discovers other services from the Server, and a load balancer is built into the Eureka Client for basic load balancing.

## Appendix

1. [Eureka Official WIKI](https://github.com/Netflix/eureka/wiki)
