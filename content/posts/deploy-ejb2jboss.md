---
categories:
    - java
date: 2019-10-23T20:28:20Z
description: deploy ejb with restful service to jboss
keywords: ejb, restful, jboss
lastmod: 2023-08-18T13:13:20Z
tags:
    - ejb
    - restful
    - jboss
    - gradle
    - maven
title: deploy ejb with restful service to jboss
---



> Wildfly: 18.0.1.Final
>
> JDK: 11.0.2
>
> Gradle: 5.6.2
>
> Maven: 3.6.2
>
> We'll deploy an ear package with two war packages(One of them uses the rest service) to jboss.

[Source code]( https://github.com/pplmx/DeployEjb2JBoss )

<!-- more -->

# Here is the project structure

![1571834311583](assets/1571834311583.png)

> The base is module ejb
>
> Web and app modules both depend on ejb
>
> ear includes web and app

# root build.gradle

```groovy
group 'individual.cc'
version '1.0-SNAPSHOT'

allprojects {
    repositories {
        jcenter()
        mavenCentral()
    }
}

subprojects {
    group 'individual.cc'
    version '1.0-SNAPSHOT'
}
```

# ejb module: build.gradle

```groovy
plugins {
    id 'java'
}

sourceCompatibility = 11

dependencies {
    compileOnly 'javax:javaee-api:8.0.1'
}
```

# ejb module: session bean

```java
package individual.cc.jar.bean.session;

import javax.ejb.Stateless;

@Stateless
public class SimpleStatelessEjb {
    public String hello() {
        return "hello world, EJB";
    }
}
```

# web module: build.gradle

```groovy
plugins {
    id 'war'
}

sourceCompatibility = 11

dependencies {
    providedCompile project(':ejb')

    compileOnly 'javax:javaee-api:8.0.1'
}
```

# web module: web servlet

```java
package individual.cc.web.servlet;

import individual.cc.jar.bean.session.SimpleStatelessEjb;

import javax.ejb.EJB;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet({"/", "/ejbServlet"})
public class EjbServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @EJB
    SimpleStatelessEjb statelessBean;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        PrintWriter writer = resp.getWriter();
        String msg = statelessBean.hello();
        writer.println(msg);
    }
}

```

# app module: build.gradle

```groovy
plugins {
    id 'war'
}

sourceCompatibility = 11

ext {
    jerseyVersion = '2.29.1'
}

dependencies {
    providedCompile project(':ejb')
    compileOnly 'javax:javaee-api:8.0.1'

    implementation "org.glassfish.jersey.containers:jersey-container-servlet:${jerseyVersion}"
}

```

# app module: controller and rest configuration

```java
package individual.cc.app.servlet;

import individual.cc.jar.bean.session.SimpleStatelessEjb;

import javax.ejb.EJB;
import javax.ejb.Stateless;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;


@Stateless
@Path("hello")
public class HelloController {

    @EJB
    SimpleStatelessEjb statelessBean;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public String hello() {
        return statelessBean.hello();
    }
}

```

```java
package individual.cc.app.configuration;

import individual.cc.app.servlet.HelloController;
import org.glassfish.jersey.server.ResourceConfig;

import javax.ws.rs.ApplicationPath;

/**
 * 'services', '/services', or '/services/*'
 * is all the same. Jersey will change it to be '/services/*'
 * <==>
 * <servlet-mapping>
 * <servlet-name>RestApplication</servlet-name>
 * <url-pattern>/services/*</url-pattern>
 * </servlet-mapping>
 * <p>
 * Here with the @ApplicationPath, it's just like if we configured the servlet mapping in the web.xml
 */
@ApplicationPath("services")
public class RestApplication extends ResourceConfig {

    public RestApplication() {
//        packages("individual.cc.app.servlet");
        register(HelloController.class);
    }
}

```

# ear module: build.gradle

```groovy
plugins {
    id 'ear'
}

dependencies {
    // The following dependencies will be the ear modules and
    // will be placed in the ear root
    deploy project(':ejb')
    deploy project(path: ':web', configuration: 'archives')
    deploy project(path: ':app', configuration: 'archives')
}

ear {
    appDirName 'src/main/app'  // use application metadata found in this folder
    // put dependent libraries into APP-INF/lib inside the generated EAR
    libDirName 'APP-INF/lib'
    deploymentDescriptor {  // custom entries for application.xml:
        initializeInOrder = true
    }
}
```

# build & deploy

1. clean and build ejb module
2. clean and build app/web module
3. clean and build ear module

Copy `ear module/build/libs/ear-1.0-SNAPSHOT.ear` to `JBOSS HOME/standalone/deployments`

Run `JBOSS HOME/bin/standalone.bat` as `administrator`

http://127.0.0.1:8080/web-1.0-SNAPSHOT/

http://127.0.0.1:8080/app-1.0-SNAPSHOT/services/hello

https://127.0.0.1:8443/web-1.0-SNAPSHOT/

https://127.0.0.1:8443/app-1.0-SNAPSHOT/services/hello

All of the above will output **hello world, EJB**

---

---

# if maven, replace 5 build.gradle

## root pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>individual.cc</groupId>
    <artifactId>j2ee</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>ear</module>
        <module>ejb</module>
        <module>web</module>
        <module>app</module>
    </modules>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!--custom package-->
            <dependency>
                <groupId>individual.cc</groupId>
                <artifactId>ejb</artifactId>
                <version>1.0-SNAPSHOT</version>
                <type>ejb</type>
            </dependency>
            <dependency>
                <groupId>individual.cc</groupId>
                <artifactId>web</artifactId>
                <version>1.0-SNAPSHOT</version>
                <type>war</type>
            </dependency>
            <dependency>
                <groupId>individual.cc</groupId>
                <artifactId>app</artifactId>
                <version>1.0-SNAPSHOT</version>
                <type>war</type>
            </dependency>
            <dependency>
                <groupId>individual.cc</groupId>
                <artifactId>ear</artifactId>
                <version>1.0-SNAPSHOT</version>
                <type>ear</type>
            </dependency>

            <!--external package-->
            <dependency>
                <groupId>javax</groupId>
                <artifactId>javaee-api</artifactId>
                <version>8.0.1</version>
            </dependency>

            <dependency>
                <groupId>org.glassfish.jersey.containers</groupId>
                <artifactId>jersey-container-servlet</artifactId>
                <version>2.29.1</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <version>3.2.2</version>
                    <configuration>
                        <failOnMissingWebXml>false</failOnMissingWebXml>
                    </configuration>
                </plugin>
                <plugin>
                    <artifactId>maven-ear-plugin</artifactId>
                    <version>3.0.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-ejb-plugin</artifactId>
                    <version>3.0.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

## ejb module: pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>individual.cc</groupId>
        <artifactId>j2ee</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>ejb</artifactId>
    <packaging>ejb</packaging>

    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-ejb-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## web module: pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>individual.cc</groupId>
        <artifactId>j2ee</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>web</artifactId>
    <packaging>war</packaging>

    <dependencies>
        <!--custom package-->
        <dependency>
            <groupId>individual.cc</groupId>
            <artifactId>ejb</artifactId>
            <type>ejb</type>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```

## app module: pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>individual.cc</groupId>
        <artifactId>j2ee</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>app</artifactId>
    <packaging>war</packaging>

    <dependencies>
        <!--custom package-->
        <dependency>
            <groupId>individual.cc</groupId>
            <artifactId>ejb</artifactId>
            <type>ejb</type>
        </dependency>

        <dependency>
            <groupId>org.glassfish.jersey.containers</groupId>
            <artifactId>jersey-container-servlet</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```

## ear module: pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>individual.cc</groupId>
        <artifactId>j2ee</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>ear</artifactId>
    <packaging>ear</packaging>

    <dependencies>
        <dependency>
            <groupId>individual.cc</groupId>
            <artifactId>web</artifactId>
            <type>war</type>
        </dependency>
        <dependency>
            <groupId>individual.cc</groupId>
            <artifactId>app</artifactId>
            <type>war</type>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-ear-plugin</artifactId>
                <configuration>
                    <!--<initializeInOrder>true</initializeInOrder>-->
                    <modules>
                        <webModule>
                            <groupId>individual.cc</groupId>
                            <artifactId>web</artifactId>
                            <!--MUST reset the name of a package what is in ear package-->
                            <bundleFileName>web-in-ear.war</bundleFileName>
                            <!--set custom context root-->
                            <contextRoot>/web</contextRoot>
                        </webModule>
                        <webModule>
                            <groupId>individual.cc</groupId>
                            <artifactId>app</artifactId>
                            <!--MUST reset the name of a package what is in ear package-->
                            <bundleFileName>app-in-ear.war</bundleFileName>
                            <!--set custom context root-->
                            <contextRoot>/app</contextRoot>
                        </webModule>
                    </modules>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

## build & deploy

`clean and install under root pom.xml `

Then deploy it to jboss, you can access the following urls:

http://localhost:8080/web/

http://localhost:8080/app/services/hello

https://localhost:8443/web/

https://localhost:8443/app/services/hello
