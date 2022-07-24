# 背景

```
计算两个文本的编辑距离，因为Java只适用英文字符，故中文编辑距离需要启用python服务rpc调用
```

python版本，3.9

python的bash文件位于~/.bash_profile



![](https://tva1.sinaimg.cn/large/e6c9d24ely1h34k77lg0pj212a0modhg.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h34k9uqnnuj21040biabu.jpg)

压缩比更大。



python3 -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. qu.proto



![](https://tva1.sinaimg.cn/large/e6c9d24ely1h34u8rv0wij21dk0u0tcc.jpg)



在子模块放入依赖即可

```xml
<build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <protoSourceRoot>${project.basedir}/src/main/java/com/wzr/yi/proto</protoSourceRoot>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.16.1:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>

    </build>
```

