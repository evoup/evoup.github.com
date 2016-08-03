---
layout: post
title: "maven打包时候加入依赖"
date: 2014-03-15 14:52
comments: true
categories: [java,maven]
---

maven打包默认不加入依赖的jar包，需要改动pom.xml，加入以下内容：
<!-- more -->

{% codeblock lang:xml pom.xml %}
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                        <configuration>
                            <excludes>
                                <exclude>/conf/**</exclude>
                                <exclude>/fonts/**</exclude>
                                <exclude>/images/**</exclude>
                                <exclude>/sounds/**</exclude>
                            </excludes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>     
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>1.2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.company.mod.cls</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <testFailureIgnore>true</testFailureIgnore>
                </configuration>
            </plugin>
       </plugins>
    </build>
{% endcodeblock %}

