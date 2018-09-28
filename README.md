## SpringCloud + Docker 自动构建微服务容器 Demo


- 使用spring cloud 搭建demo,包含一个注册中心、一个配置中心、一个服务提供者、一个服务调用者
- 使用com.spotify:docker-maven-plugin 插件，package 时自动构建DockerImage,并推送到指定的Docker仓库
- 使用docker-compose 编排容器，一键启动所有容器；使用wait-for脚本，支持按顺序启动容器（启动完成后启动下一个容器），解决容器启动依赖问题

### 模块介绍 

- eureka-server 微服务注册中心
- config-server 微服务配置中心
- hello-service hello服务提供者 （启动依赖配置中心）
- hello-client hello服务调用者 
- docker  存放docker-compose.yml配置文件

### 容器启动顺序

1. eureka-server 
2. config-server 和 hello-service （互不影响）
3. hello-client

### 使用方式

1.本地必须具有maven环境（用于打包）和docker环境（用于本地构建镜像），并且拥有一个可使用（上传镜像）的docker仓库，本地docker环境需配置好docker仓库地址。
 - 本地安装Docker及Docker仓库安装可参考 https://blog.csdn.net/u013197629/article/details/82879096

 
2.更改父项目pom里面 的docker仓库相关<properties> 信息为本地环境信息。
```
    <properties>
        <docker.registryServerId>230-5000</docker.registryServerId>
        <docker.registryHost>10.20.26.230:5000</docker.registryHost>
        <docker.registryUrl>http://${docker.registryHost}/v2</docker.registryUrl>
    </properties>
```
 - 如果使用的是私有仓库，必须配置鉴权信息。远程docker仓库账号密码配置方式：在本地maven的settings配置文件里配置对应serverId的鉴权信息
 ```
 <server>
     <id>230-5000</id>
     <username>admin</username>
     <password>admin</password>
     <configuration>
         <email>wlin@xxx.com</email>
     </configuration>
 </server>
 ```
3.在父pom里运行mvn package命令，构建所有镜像（将同时推送到配置的仓库）

- 正常运行结果日志（单个模块) 参考如下：
```
[INFO] Scanning for projects...
Downloading: http://10.20.26.250:8081/nexus/content/repositories/releases/me/wlin/cloud/spring-cloud-docker/1.0-SNAPSHOT/maven-metadata.xml
[WARNING] 
[WARNING] Some problems were encountered while building the effective model for me.wlin.cloud:feign-hystrix-config-demo:jar:0.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.springframework.boot:spring-boot-starter-actuator:jar -> duplicate declaration of version (?) @ line 74, column 15
[WARNING] 'version' contains an expression but should be a constant. @ me.wlin.cloud:spring-cloud-docker:${version}, G:\work\workspace\spring-cloud-docker-demo\pom.xml, line 17, column 14
[WARNING] 'version' contains an expression but should be a constant. @ me.wlin.cloud:spring-cloud-docker:${version}, G:\work\soft\maven\apache-maven-3.2.1\repository\me\wlin\cloud\spring-cloud-docker\1.0-SNAPSHOT\spring-cloud-docker-1.0-SNAPSHOT.pom, line 17, column 14
[WARNING] 
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING] 
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING] 
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building hello-client 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ feign-hystrix-config-demo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ feign-hystrix-config-demo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 5 source files to G:\work\workspace\spring-cloud-docker-demo\hello-client\target\classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ feign-hystrix-config-demo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory G:\work\workspace\spring-cloud-docker-demo\hello-client\src\test\resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ feign-hystrix-config-demo ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.18.1:test (default-test) @ feign-hystrix-config-demo ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ feign-hystrix-config-demo ---
[INFO] Building jar: G:\work\workspace\spring-cloud-docker-demo\hello-client\target\app.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.5.9.RELEASE:repackage (default) @ feign-hystrix-config-demo ---
[INFO] 
[INFO] --- docker-maven-plugin:0.4.13:build (build-image) @ feign-hystrix-config-demo ---
[INFO] Copying G:\work\workspace\spring-cloud-docker-demo\hello-client\target\app.jar -> G:\work\workspace\spring-cloud-docker-demo\hello-client\target\docker\app.jar
[INFO] Building image wlin/hello-client
Step 1/3 : FROM frolvlad/alpine-oraclejdk8:slim
 ---> ceae9ac75359
Step 2/3 : ADD /app.jar //
 ---> edf4953e5aa5
Removing intermediate container cd87e5225192
Step 3/3 : ENTRYPOINT java -jar /app.jar
 ---> Running in 4ba314b602eb
 ---> e76393ba82a4
Removing intermediate container 4ba314b602eb
Successfully built e76393ba82a4
[INFO] Built wlin/hello-client
[INFO] 
[INFO] --- docker-maven-plugin:0.4.13:tag (tag-image) @ feign-hystrix-config-demo ---
[INFO] Creating tag 10.20.26.230:5000/wlin/hello-client:0.0.1-SNAPSHOT from wlin/hello-client
[INFO] 
[INFO] --- docker-maven-plugin:0.4.13:push (push-image) @ feign-hystrix-config-demo ---
[INFO] Pushing 10.20.26.230:5000/wlin/hello-client:0.0.1-SNAPSHOT
The push refers to a repository [10.20.26.230:5000/wlin/hello-client]
7972eb0f9919: Preparing 
cc74a499898c: Preparing 
602dfdb9055b: Preparing 
cd7100a72410: Preparing 
cd7100a72410: Layer already exists 
602dfdb9055b: Layer already exists 
cc74a499898c: Layer already exists 
7972eb0f9919: Pushing [>                                                  ]  426.5kB/41.38MB
7972eb0f9919: Pushing [=>                                                 ]  852.5kB/41.38MB
7972eb0f9919: Pushing [===>                                               ]  2.982MB/41.38MB
7972eb0f9919: Pushing [=====>                                             ]   4.26MB/41.38MB
7972eb0f9919: Pushing [======>                                            ]  5.538MB/41.38MB
7972eb0f9919: Pushing [========>                                          ]  7.242MB/41.38MB
7972eb0f9919: Pushing [==========>                                        ]   8.52MB/41.38MB
7972eb0f9919: Pushing [============>                                      ]  10.22MB/41.38MB
7972eb0f9919: Pushing [=============>                                     ]   11.5MB/41.38MB
7972eb0f9919: Pushing [===============>                                   ]  13.21MB/41.38MB
7972eb0f9919: Pushing [=================>                                 ]  14.48MB/41.38MB
7972eb0f9919: Pushing [===================>                               ]  16.19MB/41.38MB
7972eb0f9919: Pushing [=====================>                             ]  17.47MB/41.38MB
7972eb0f9919: Pushing [=======================>                           ]  19.17MB/41.38MB
7972eb0f9919: Pushing [========================>                          ]  20.45MB/41.38MB
7972eb0f9919: Pushing [==========================>                        ]  22.15MB/41.38MB
7972eb0f9919: Pushing [============================>                      ]  23.43MB/41.38MB
7972eb0f9919: Pushing [==============================>                    ]  25.13MB/41.38MB
7972eb0f9919: Pushing [===============================>                   ]  26.41MB/41.38MB
7972eb0f9919: Pushing [=================================>                 ]  28.12MB/41.38MB
7972eb0f9919: Pushing [====================================>              ]  29.82MB/41.38MB
7972eb0f9919: Pushing [=====================================>             ]   31.1MB/41.38MB
7972eb0f9919: Pushing [=======================================>           ]   32.8MB/41.38MB
7972eb0f9919: Pushing [=========================================>         ]  34.51MB/41.38MB
7972eb0f9919: Pushing [===========================================>       ]  35.78MB/41.38MB
7972eb0f9919: Pushing [============================================>      ]  37.06MB/41.38MB
7972eb0f9919: Pushing [==============================================>    ]  38.77MB/41.38MB
7972eb0f9919: Pushing [================================================>  ]  40.47MB/41.38MB
7972eb0f9919: Pushing [==================================================>]  41.38MB
7972eb0f9919: Pushed 
0.0.1-SNAPSHOT: digest: sha256:0b00f05c1d51ee9b60f2e78f83d10791ce31d8d862be80c372f83c44a6bed52d size: 1163
null: null 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13.272 s
[INFO] Finished at: 2018-09-27T16:32:50+08:00
[INFO] Final Memory: 59M/605M
[INFO] ------------------------------------------------------------------------
```


4.更改docker模块内的docker-compose.yml文件内的images地址为你的构建时推送的docker仓库地址（包含tag），然后将docker模块（即项目内的整个docker目录，包含yml和wait-for脚本）复制到任意含有docker和docker-compose的环境（需配置了构建时指定的仓库地址，如果是私有仓库，还需注意权限问题），如果本地环境符合，可以直接在项目docker路径内运行。
 - docker-compose 安装官方文档：https://docs.docker.com/compose/install/#prerequisites

5.在复制的docker-compose.yml所在目录，运行docker-compose up （可以看到所有启动日志）或 docker-compose up -d 命令启动所有容器。
```
[root@server ~]# docker-compose up -d 
Starting temp_eurekaServer_1 ... done
Starting temp_hello-service_1 ... done
Starting temp_config-server_1 ... done
Starting temp_hello-client_1  ... done

[root@server ~]# docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
0885ce551787        10.20.26.230:5000/wlin/hello-client:0.0.1-SNAPSHOT    "/bin/sh /wait-for..."   7 weeks ago         Up 15 seconds       0.0.0.0:8260->8260/tcp   temp_hello-client_1
b9c132efbfaf        10.20.26.230:5000/wlin/hello-service:0.0.1-SNAPSHOT   "/bin/sh /wait-for..."   7 weeks ago         Up 16 seconds       0.0.0.0:8270->8270/tcp   temp_hello-service_1
96ee586afd18        10.20.26.230:5000/wlin/config-server:0.0.1-SNAPSHOT   "/bin/sh /wait-for..."   7 weeks ago         Up 16 seconds       0.0.0.0:8210->8210/tcp   temp_config-server_1
ee1e8011a9ed        10.20.26.230:5000/wlin/eureka-server:0.0.1-SNAPSHOT   "java -Djava.secur..."   7 weeks ago         Up 17 seconds       0.0.0.0:8280->8280/tcp   temp_eurekaServer_1
3b29b82a169a        registry:2.5                                          "/entrypoint.sh /e..."   8 weeks ago         Up About an hour    0.0.0.0:5000->5000/tcp   repo

```
6.使用本地浏览器或者http工具访问client地址： 127.0.0.1:8260/client/hello?name=tom
```
[root@server ~]# curl 127.0.0.1:8260/client/hello?name=tom
hi,tom.I'm hello-service from the port 8270
```