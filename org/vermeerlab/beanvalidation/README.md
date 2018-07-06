Custom BeanValidation
===

BeanValidationを使いやすいように独自拡張するライブラリ.

BeanValidationにより検証をするにあたって機能拡張をするライブラリです。

* 検証とメッセージ変換を分けて行えるようにします
* Form（画面要素）を検証した際に項目名ラベルを付与し、その値をメッセージに適用します
* 複数のリソースを指定できます

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
        <artifactId>beanvalidation</artifactId>
        <version>0.1.0</version> <!-- target version -->
    </dependency>
</dependencies>
```


### 実装例

実装例はテストコードを参考にしてください。


[TestCode](https://bitbucket.org/vermeerlab/beanvalidation/src/e065b792a38088e6a3ffd6f48aade5dfff2fe080/src/test/java/org/vermeerlab/beanvalidation/it/ValidationTest.java?at=master&fileviewer=file-view-default)

#### 検証結果からメッセージ変換

```java

Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

Set<ConstraintViolation<TargetView>> results = validator.validate(view, FormValidation.class);


// メッセージ変換に使用するResourceBundleを指定します
MessageInterpolatorFactory interpolatorFactory
                            = MessageInterpolatorFactory.of("Messages", "FormMessages", "FormLabels");

// 変換インスタンスを生成します（ロケールは未指定）
MessageInterpolator interpolator = interpolatorFactory.create();

for (ConstraintViolation<TargetView> result : results) {
    // 検証結果からメッセージを変換します.
    String convertedMessage = interpolator.toMessage(result);
    System.out.println(convertedMessage);
}

```

## Code
[BitBucket](https://bitbucket.org/vermeerlab/beanvalidation)

## Version

* 0.3.2
プロパティをValueObjectからStringに変更  

* 0.1.0  
初期リリース

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)

