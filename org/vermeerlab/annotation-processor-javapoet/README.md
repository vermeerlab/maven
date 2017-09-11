Pluggable Annotation Processing API with JavaPoet
===

## Description

`annotation-processor-core`[^footnote1]の実行コマンドで`JavaPoet`を用いてコード生成を行います.

## Usage

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
        <artifactId>annotation-processor-javapoet</artifactId>
        <version>0.1.0</version> <!-- target version -->
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
### JavaPoetによるコードの生成

`JavaPoet`のファイル出力に対するラッパークラスの`JavaPoetFile`を用いてファイル生成を行います.
インスタンス生成時に指定する`JavaPoetConfigManager`はクラス生成時に共通して行う、ライセンスコメントやクラスコメントを編集するためのクラスです.


※JavaPoetそのものの使用方法については 本パッケージでは説明いたしません.

クラス定義用クラスは`JavaPoetConfigInterface`を実装します.

```java
public class JavaPoetConfig implements JavaPoetConfigInterface {

    @Override
    public List<Class<?>> generatedClass() {
        return Arrays.asList(GenerateCodeTestCommand.class);
    }

    @Override
    public List<String> authors() {
        return Arrays.asList("AUTHOR-NAME");
    }

    @Override
    public List<String> versions() {
        return Arrays.asList("1.1");
    }

    @Override
    public List<String> sinces() {
        return Arrays.asList("1.0");
    }

    @Override
    public String ownerName() {
        return "LICENSE-OWNER-NAME";
    }

    @Override
    public String copyright() {
        return "2017";
    }
}

```

`JavaPoetFile`によるコード編集（簡易）

```java

// 編集用のラッパークラスのインスタンス化
JavaPoetConfigInterface javaPoetConfig = new JavaPoetConfig();
JavaPoetConfigManager.of(javaPoetConfig);
JavaPoetFile javaPoetFile = JavaPoetFile.of(JavaPoetConfigManager.of(javaPoetConfig));

// クラスコード編集（JavaPoetの機能を使用）
// （クラス宣言のみの超簡易）
TypeSpec.Builder typeBuilder = TypeSpec.classBuilder("Test");
typeBuilder.addJavadoc("$N\n\n", "Class Comment");

// 生成クラスのパッケージ宣言とコードオブジェクトを設定して出力
javaPoetFile.setCode("org.vermeerlab.apt.element.test.command.generatecode", typeBuilder).writeTo(
        processingEnvironment);

```

出力先に指定している`processingEnvironment`は Annotation Processor で取得した環境情報です.

実行した結果は、以下の通りです.
`JavaPoetConfigInterface`の実装が、どこまでの対応をしているのか おおよそのイメージは出来るかと思います.

```java
// Copyright 2017  LICENSE-OWNER-NAME
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
package org.vermeerlab.apt.element.test.command.generatecode;

import javax.annotation.Generated;

/**
 * Class Comment
 *
 * @author AUTHOR-NAME
 * @version 1.1
 * @since 1.0
 */
@Generated({"org.vermeerlab.apt.AnnotationProcessorController", "org.vermeerlab.apt.element.test.command.generatecode.GenerateCodeTestCommand"})
class Test {
}

```

詳細はテストコードパッケージ
`org.vermeerlab.apt.element.test.command.generatecode`
を参照してください.

---
### 要素の参照

Processorのラウンドにて取得した`Element`を参照して、コードを編集するためのヘルパークラスも いくつか準備しています.

簡単なサンプルとして、ルートとなるAnnotation配下の情報を参照する例を以下に示します.

参照されるクラス

（様々な要素種別にアノテーションが付与されています）

```java
package org.vermeerlab.apt.element.test.command.other;

@ElementOther
public class Test1 {

    @ElementOther
    public String fieldTest1;

    @ElementOther
    public void methodTest1() {
        String test = "";
    }
}
```


コマンドクラス

```java
public class OtherElementTestCommand implements ProcessorCommandInterface {

    @Override
    public Class<? extends Annotation> getTargetAnnotation() {
        return ElementOther.class;
    }

    @Override
    public void execute(ProcessingEnvironment processingEnvironment, Element element, Boolean isDebug) {
        CategorizedElementInterface mixElement = CategorizedElementFactory.create(
                processingEnvironment, element, this.getTargetAnnotation());
        CategorizedElementsInterface mixingPossibilityElements = mixElement.children(this.getTargetAnnotation());

        Boolean hasAnnotation = mixElement.getAnnotation(ElementOther.class).isPresent();

        DebugUtil debugUtil = new DebugUtil(isDebug);

        debugUtil.print(
                "packageName:" + mixElement.getPackagePath() + ":"
                + "simpleName:" + mixElement.getSimpleName() + ":"
                + "hasAnnotation:" + hasAnnotation + ":"
                + "childrenCount:"
                + String.valueOf(
                        mixingPossibilityElements.values().size())
        );
    }

```

1. ルート情報から要素クラスの生成

要素をルートから参照するために、`CategorizedElementFactory`でインスタンスを生成します.

```java
CategorizedElementInterface mixElement = CategorizedElementFactory.create(
        processingEnvironment, element, this.getTargetAnnotation());
```

対象となるアノテーション（`@ElementOther`）は、対象要素の種別が複数に渡っているので、戻り値の型は汎用的なインターフェース（`CategorizedElementInterface`）にしています.

2. 配下の要素を取得

```java
CategorizedElementsInterface mixingPossibilityElements = mixElement.children(this.getTargetAnnotation());
Boolean hasAnnotation = mixElement.getAnnotation(ElementOther.class).isPresent();
```

要素が`Class`の場合は、配下にフィールドとメソッドがあるので戻り値が存在しますが、要素が`Field`や`Method`の場合は配下の要素が存在しません.

詳細はテストコードパッケージ
`org.vermeerlab.apt.element.test.command.other`
を参照してください.
（テスト結果の期待値の `childrenCount:`の数値で要素数の違いは確認できます.）


### 準備しているヘルパークラス

|ElementKind| item|collection|
|:----------|:------------|:------------|
|PACKAGE|`PackageElement`||
|CLASS |`ClassElement`|`ClassElements`|
|METHOD |`MethodElement`|`MethodElements`|
|FIELD|`FieldElement`|`FieldElements`|
|else|`CategorizedElement`|`CategorizedElements`|



[^footnote1]:annotation-processor-core(https://github.com/vermeerlab/maven/tree/mvn-repo/org/vermeerlab/annotation-processor-core)

## Code
[BitBucket](https://bitbucket.org/vermeerlab/apt-javapoet/overview)


## Version
* 0.1.0
初回リリース

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)

