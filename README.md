

# SpringRestProtobuf
This is a Prototype for a SpringBoot-REST implementation. It offers a Restful service using Google Protocol Buffers (protobuf) and JSON.

My project is a feasibility study and is **NOT PRODUCTION READY**.

# Prerequisites
- [`Maven 3.3.9`](http://maven.apache.org/)
- [`Protoc 2.6.1`](https://github.com/google/protobuf/releases)
- [`Oracle JDK 1.8.0_72`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

# Quickstart
If you are only interested in my source code or want to compare my version to yours, follow these steps:

1. Clone Project from GitHub `git clone https://github.com/sam6-de/SpringRestProtobuf.git`
2. Build Project using Maven `cd ./SpringRestProtobuf/ && mvn clean install`
3. Start embedded Jetty `cd ./SpringRestProtobuf/target/ && java -jar SpringRestWebapp-1.0-SNAPSHOT.jar`

Now the service is up and running using port `8080`. Simply connect via browser, for example by using [http://localhost:8080/rest/hello/world](http://localhost:8080/rest/hello/world)

Jersey automatically generates a `WADL` file which describes all valid operations: [http://localhost:8080/rest/application.wadl](http://localhost:8080/rest/application.wadl).
This is simplified WADL with user and core resources only. To get full WADL with extended resources use the query parameter detail: [http://localhost:8080/rest/application.wadl?detail=true](http://localhost:8080/rest/application.wadl?detail=true)

# Build from scratch
## Preparation

If you want to set up your Maven project as multi module project, you should first create a root `pom.xml`-file.

    mvn archetype:generate \
        -DarchetypeGroupId=org.codehaus.mojo.archetypes \
        -DarchetypeArtifactId=pom-root \
        -DarchetypeVersion=RELEASE \
        -DgroupId=de.sam6.demo \
        -DartifactId=SpringRestProtobuf \
        -DinteractiveMode=false
In order to create a new/first submodule in the project, change the directory and use the Maven webapp archetype, that will create (most of) necessary folders and files:

    cd RestfulProtobuf/
    mvn archetype:generate \
        -DarchetypeArtifactId=maven-archetype-webapp \
        -DgroupId=de.sam6.demo \
        -DartifactId=SpringRestWebapp \
        -DinteractiveMode=false
Because the `java` and `test` source folders are not created automatically, we have to do this manually:

    mkdir SpringRestWebapp/src/main/java
    mkdir -p SpringRestWebapp/src/test/java

Now you can import the whole multi-module-project into your favourite IDE.

### Parent POM
To use the spring-boot-framework in all submodules you have to add these lines to your master POM file formerly created using the maven archetype `pom-root`

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.2.RELEASE</version>
    </parent>

Because we want everything to be compiled in Java 8, we have to tell maven to do so:

    <properties>
        <java.version>1.8</java.version>
    </properties>
    
### Module POM

Now you can use the following dependencies in your modules POM that was created in the second step:

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

The [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) provides some features we need in later steps. So we have to add some lines to the build configuration of our module POM:

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    
    
## Implementation

As first step we implement the SpringBootApplication that will be our starting point later. Therefore create the new class `MyConfiguration.java` in package `de.sam6.demo.springrestprotobuf`:

    package de.sam6.demo.springrestprotobuf;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class MyConfiguration {
    
        public static void main(String[] args) {
            SpringApplication.run(MyConfiguration.class, args);
        }
    }
    
Now we can create two new classes that will do a more or less useless thing: say hello ;-)

The first class could be something like this:

    package de.sam6.demo.springrestprotobuf.model;
    
    public class HelloModel {

        private final long id;
        private final String content;

        public HelloModel(long id, String content) {
            this.id = id;
            this.content = content;
        }

        public long getId() {
            return id;
        }
        
        public String getContent() {
            return content;
        }
    }

And the resource controller could be:

    package de.sam6.demo.springrestprotobuf.controller;

    import java.util.concurrent.atomic.AtomicLong;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.ResponseBody;

    @Controller
    @RequestMapping("/say-hello")
    public class HelloWorldController {

        private static final String template = "Hello, %s!";
        private final AtomicLong counter = new AtomicLong();

        @RequestMapping(method=RequestMethod.GET)
        public @ResponseBody HelloModel sayHello(@RequestParam(value="name", required=false, defaultValue="world") String name) {
            return new HelloModel(counter.incrementAndGet(), String.format(template, name));
        }
    }
    
