```sh
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v1.5.3.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatEmbeddedServletContainerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```
Is this screen familiar? Then you are a Spring-Boot user.

Next, we will change its banner.

How?

Add a `banner.txt` file to your classpath.

And run your application.
```sh

    PPPPPPPPP      ZZZZZZZZZZ       GGGGGGGGG
    PP      PP            ZZ       GG
    PPPPPPPPP           ZZ         GG    GGGGGG
    PP                ZZ           GG       GG
    PP              ZZ             GG       GG
    PP             ZZZZZZZZZZ       GGGGGGGGG


2017-05-04 12:36:36.338  INFO 2099 --- [           main] c.l.a.AdminApplication  : Starting AdminApplication on localhost with PID 2099 (/apps/myapp.jar started by pwebb)
2017-05-04 12:36:36.341 DEBUG 2099 --- [           main] c.l.a.AdminApplication  : Running with Spring Boot v1.5.1.RELEASE, Spring v4.3.6.RELEASE
2017-05-04 12:36:36.341  INFO 2099 --- [           main] c.l.a.AdminApplication  : The following profiles are active: local
```
_It's so cooooool_

_Try it !_

> References

* https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-banner
