Dynamic Compiler
===

## Description

文字列からクラスをコンパイルする機能を提供します.

## Usage

### pom.xml 記述

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <!-- Maven Repository on GitHub  start -->
    <git.branchName>mvn-repo</git.branchName>
    <git.repositoryOwner>vermeerlab</git.repositoryOwner>
    <git.repositoryName>maven</git.repositoryName>
    <!-- Maven Repository on GitHub  end-->
</properties>

<repositories>
    <!-- Maven Repository on GitHub  start -->
    <repository>
        <id>org.vermeerlab</id>
        <url>https://raw.github.com/${git.repositoryOwner}/${git.repositoryName}/${git.branchName}/</url>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
    <!-- Maven Repository  on GitHub  end-->
</repositories>

<dependencies>
    <dependency>
        <groupId>org.vermeerlab</groupId>
        <artifactId>vermeerlab-dynamic-compiler</artifactId>
        <version>0.1.0</version>
    </dependency>
</dependencies>

```
### Sample

テストコードの抜粋

#### インターフェースの実装を文字列で指定

```java
@Test
public void 文字列からクラスをコンパイル() throws NoSuchMethodException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
    String classString = "public class Test implements java.util.function.Supplier { public Object get(){return \"testclass\";}}";
    Class<?> clazz = new ClassCompiler().compile("Test", classString);
    Object value = ((Supplier) clazz.getDeclaredConstructor().newInstance()).get();
    Assert.assertThat(value, is("testclass"));
}
```


## Code
[BitBucket](https://bitbucket.org/vermeerlab/dynamic-compiler)

## Version

* 0.1.0
初回リリース


## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)
