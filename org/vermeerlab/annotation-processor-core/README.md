Pluggable Annotation Processing API Core
===

## Description

`Pluggable Annotation Processing API`を実行する基底となるプロジェクトです.

ラウンド毎、および ラウンド毎の処理結果をまとめて最後に実行する ２種類のコマンドの作成ができます.

`Annotation Processor`を実行するコマンドはコンパイル都度、クラスパスから検索します（Jarファイルにアーカイブされているクラスも対象にします）.


## Usage

### 流れ

1. `Annotation Processor`が実行できる`pom.xml`の設定
2. Command Class を実装
3. コンパイル時に`Annotation Processor`により実行される（※）

※ プロジェクトのクラスパスリソース直下の`processor-command.xml`により、実行時の制御ができます（任意）.

---

### pom.xml 記述

利用ライブラリとして、また`Annotation Processor`を実行する設定をするために`pom.xml`を編集します.

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
        <version>0.3.0</version> <!-- target version -->
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
### proseccor-command.xml の設定（任意）

#### ファイルが存在しない場合（デフォルト）
  
* コンソールに実行コマンドは表示しません  
* 全ての`.class`ファイル および、`jar`ファイルを検索対象とします

#### ファイルが存在する場合

`processor-command.xml`の設定により、以下の対応が可能です.

* 実際に実行しているコマンドクラスを確認をコンソールに出力
* 適用したくないコマンドクラスの除外
* クラスパス配下のファイルだけを対象にして、`jar`を対象外
* 検索対象の`jar`ファイルを指定

詳細は xmlコメントを参照してください.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
Annotation Processor で実行するコマンドを登録する設定ファイルです.
コマンドクラスは
org.vermeerlab.apt.command.ProcessorCommandInterface
を実装してください.

全ラウンドで まとめて行うコマンドは
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

    <!--
    コマンドクラスの検索対象としてクラスパス配下のJarファイルを読み込み有無を設定してください.
    Jarファイルを読み込む場合は true
    -->
    <scanJarFile>true</scanJarFile>

    <!--
    コマンドクラスの検索対象とするクラスパス配下のJarファイルを設定してください.
    -->
    <scanTargetJarFiles>
        <scanTargetJarFile>
        </scanTargetJarFile>
    </scanTargetJarFiles>

</root>

```




## Code
[BitBucket](https://bitbucket.org/vermeerlab/apt-core/overview)


## Version

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
