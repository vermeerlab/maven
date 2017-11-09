Pluggable Annotation Processing API Core
===

## Description

`Pluggable Annotation Processing API`を実行する基底となるプロジェクトです.

ラウンド毎、および ラウンド毎の処理結果をまとめて最後に実行する ２種類のコマンドを制御できます.

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
        <artifactId>annotation-processor-core</artifactId>
        <version>0.4.0</version>
    </dependency>
</dependencies>

<build>
    <plugins>

        <!-- for annotation processor start-->
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
                <source>${maven.compiler.source}</source>
                <target>${maven.compiler.target}</target>
                <!-- Disable annotation processing for ourselves. -->
                <compilerArgument>-proc:none</compilerArgument>
                <compilerArgs>
                    <arg>-Xlint</arg>
                </compilerArgs>
                <showDeprecation>true</showDeprecation>
            </configuration>
        </plugin>
        <!-- annotation processor end -->
    </plugins>
</build>

```
---
### Command Class の実装

`org.vermeerlab.apt.command.ProcessorCommandInterface`、または`org.vermeerlab.apt.command.PostProcessorCommandInterface`を実装したクラスをコンパイル時に検索して実行します.

２つのインターフェースの違い

* ラウンド毎に処理（`ProcessorCommandInterface`）
* ラウンド毎の処理に加えて、最終ラウンドに処理（`PostProcessorCommandInterface`）

---
### 定義用Xmlの設定

#### 定義用Xmlが存在しない場合（デフォルト）

* `ProcessorCommandInterface` または `PostProcessorCommandInterface` 実装したクラスを処理対象とします
* コンソールに実行コマンドは表示しません


#### 定義用Xmlが存在しない場合

##### 定義用Xmlファイルの指定

pom.xml に `Annotation Processor`にて参照する 定義用Xmlのパスをシステムプロパティ（`processorcommand.classpath`）に設定します.

```xml
<properties>
    <processorcommand.classpath>/processor-command.xml</processorcommand.classpath>
</properties>

<build>
    <plugins>
        <!-- for annotation processor start-->
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.0</version>
            <configuration>
                <source>${maven.compiler.source}</source>
                <target>${maven.compiler.target}</target>
                <!-- Disable annotation processing for ourselves. -->
                <compilerArgument>-proc:none</compilerArgument>
                <compilerArgs>
                    <arg>-Xlint</arg>
                </compilerArgs>
                <showDeprecation>true</showDeprecation>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.bsc.maven</groupId>
            <artifactId>maven-processor-plugin</artifactId>
            <version>2.2.4</version>
            <executions>
                <execution>
                    <id>process</id>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <phase>generate-sources</phase>
                    <configuration>
                        <compilerArguments>-encoding ${project.build.sourceEncoding}</compilerArguments>
                        <systemProperties>
                            <processorcommand.classpath>${processorcommand.classpath}</processorcommand.classpath>
                        </systemProperties>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <!-- annotation processor end -->
    </plugins>
</build>
```

以下の対応が可能です.

* 実際に実行しているコマンドクラスを確認をコンソールに出力
* 適用したくないコマンドクラスの除外

詳細は xmlコメントを参照してください.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
Annotation Processor で実行するコマンドを登録する設定ファイルです.
コマンドクラスは
org.vermeerlab.apt.command.ProcessorCommandInterface
を実装してください.

ラウンドで まとめて行うコマンドは
org.vermeerlab.apt.command.PostProcessorCommandInterface
を実装してください.
-->
<root>
    <!--
    実行コマンドリストを標準出力制御を設定してください.
    出力する場合は true.
    デフォルトは false（出力しない）
    -->
    <displayCommandList>true</displayCommandList>

    <!--
    実行対象外とするコマンドクラスの完全修飾名を設定してください.
    -->
    <excludeCommands>
        <excludeCommand>
        </excludeCommand>
    </excludeCommands>
</root>
```

## 独自拡張について

本パッケージは 実行コマンドの実行制御と コマンド実行時のパラメータを外部ファイルにて指定できる基底プロジェクトです.

要件にあわせて拡張することを前提としています.

 [JavaPoet](https://github.com/square/javapoet)を使用して、Javaコードを生成する拡張をしたプロジェクトです.

```
https://bitbucket.org/vermeerlab/apt-javapoet
```







## Code
[BitBucket](https://bitbucket.org/vermeerlab/apt-core)

## Version
* 0.4.0
実装クラスの検索機能の実装（xmlファイルでの指定をしなくても実行コマンドとなる）

* 0.3.0
実行コマンドクラスをprocessor-command.xmlから読み込むのではなくクラスパス配下（jar含む）から検索する。
processor-command.xmlにより資産読み込みの指定をできるようにする。

* 0.2.0（リポジトリなし）
機能分割（base,xmlパッケージを別プロジェクトに分割）
bug fixed

* 0.1.0
初回リリース（リポジトリなし）
ブログ掲載 http://vermeer.hatenablog.jp/entry/2017/08/18/230711


## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)
