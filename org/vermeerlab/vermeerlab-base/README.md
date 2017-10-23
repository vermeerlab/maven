基底クラス群
===

アプリケーション共通の基底となるクラス群です。

## Usage

### pom.xml

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
        <artifactId>vermeerlab-base</artifactId>
        <version>0.3.0</version>
    </dependency>
</dependencies>
```

## Code
[BitBuckt](https://bitbucket.org/vermeerlab/base/overview)


## Version
* 0.3.0  
ClassLoader関連は [FastClasspathScanner](https://github.com/lukehutch/fast-classpath-scanner)を使用するように改修


* 0.2.0  
型検証機能の追加  
ClassLoader機能の追加

* 0.1.0  
初期リリース（リポジトリなし）

## Licence
Licensed under the [Apache License, Version 2.0][Apache]  
[Apache]:http://www.apache.org/licenses/LICENSE-2.0

## Author
[_vermeer_](https://twitter.com/_vermeer_)
