---

layout: post
title:  How to check version of your spring application on production
categories: [spring , java]


How to check version of your spring application on production
Today's software deployment is a repetitive process some times you may deploy your software a hundred times in a day especially when your system is on microservices architect you have many services and continuously deployment.
so one of the important things is you must know which version of your service is on production now in this article we learn how to use spring actuator and maven git plugin to find out which version of our application is running on production now

---

So we need actuator in our dependencies:
```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
Now you can run your application and go to `localhost:8080/actuator` you see some routes in a JSON. The git related information goes in the `localhost:8080/actuator/info`, by default this endpoint is not exposed but you can expose this route to expose it you need to add this to your properties:
```properties
management.endpoints.web.exposure.include=info
```
Alternatively, you can expose all endpoints by using * but now we need only information endpoint. in the information endpoint we don't. have anything, for now so we need to add the maven git plugin
```xml
<plugin>
<groupId>pl.project13.maven</groupId>
<artifactId>git-commit-id-plugin</artifactId>
</plugin>
```
Now build application again and check the info endpoint
```json
{
  "git": {
    "branch": "master",
    "commit": {
      "id": "98951f1",
      "time": "2021-07-15T12:10:58Z"
    }
  }
}
```
If you want somthing like a message in this endpoint you can add `info.message=hello` world in your application properties any thing you write after info. will be key and you can add any value you want the response of info endpoint
```json
{
  "message": "hello world",
  "git": {
    "branch": "master",
    "commit": {
      "id": "98951f1",
      "time": "2021-07-15T12:10:58Z"
    }
  }
}
```
I hope you enjoy it ;)
And Leave comment that really makes me happy❤️
You can also read this article on Medium blog here