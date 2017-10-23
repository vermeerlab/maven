XML操作
===

XMLを操作するクラス群です

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
        <artifactId>vermeerlab-xml</artifactId>
        <version>0.3.0</version>
    </dependency>
</dependencies>
```

## Code
[BitBuckt](https://bitbucket.org/vermeerlab/xml)


## Version

* 0.2.1

CDATAでは空白除去しないように修正

テストケースの追記・バグ改修・JavaDoc見直し

* 0.2.0

ファイルパスからxmlを読み込む機能の追加

* 0.1.0

初期リリース（リポジトリなし）

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)
