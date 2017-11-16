Pluggable Annotation Processing API with JavaPoet
===

## Description

`JavaPoet`を用いてコード生成をするための基底クラスである
`org.vermeerlab.apt.javapoet.AbstractJavaPoetProcessorCommand`を継承したクラスが[annotation-processor-core](https://github.com/vermeerlab/maven/tree/mvn-repo/org/vermeerlab/annotation-processor-core)の処理対象のコマンドとなり、コンパイル都度 AnnotationProcessorで実行されます.


本プロジェクトは あくまで基底となる機能を提供するだけで 実際の実行コマンドではありません。

## Usage

### コマンドクラス（基底クラス拡張）

実行コマンド本体として`org.vermeerlab.apt.javapoet.AbstractJavaPoetProcessorCommand`を継承したクラスを作成します.


#### `#getTargetAnnotation` 

コマンドの取得ルートとなるElementとなるアノテーションを戻り値となるように実装します.


#### `#getExtentionCommandConfigScannerClasses`

実行コマンドで定義用XMLから情報を取得するクラスを戻り値となるように実装します.


#### `#execute`

ラウンド毎の処理を実装します.



### 定義用XML読み込みクラス

当該コマンドの実行時に外部から値を指定したい場合 定義用XMLに独自のパスを設けることで拡張ができます.

定義用XMLを読み込む共通インターフェース（ `org.vermeerlab.apt.config.CommandConfigScannerInterface`）を実装したクラスを作成します.

#### `#readConfig`

定義用XMLから`xpath`で指定して要素を取得します.

##### Note

AnnotationProcessorの`#init`にて、定義用XMLを読み込んだインスタンスを保持します.

コマンド側で必要に応じて`org.vermeerlab.apt.javapoet.AbstractJavaPoetProcessorCommand#getCommandConfigScanner`により情報を保持したインスタンスを取得して参照をします.



### アノテーション要素を扱うヘルパークラス

要素毎の情報取得を補助する簡単なヘルパークラスも提供しています.

#### 要素を参照するクラス

`org.vermeerlab.apt.element.CategorizedElementKind`をFactoryとして アノテーションされている要素に適するインスタンスを生成します.


#### 一括検証とインターフェース

`org.vermeerlab.apt.command.CommandValidator`に、検証インターフェース`org.vermeerlab.apt.command.ValidationInterface`で実装したクラスをパラメータとして渡すことで取得した要素の検証を一括して行えます.





### 定義用XML

コマンドで参照する定義用XMLには

* 全コマンドの共通設定

* コード生成コマンド毎の共通設定

があり、それに加えて 独自のタグを設けることも出来ます.


#### 共通設定

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <javaPoetCommand>
        <licenseTemplate><![CDATA[
Copyright [CopyRightYear]  [name of copyright owner]
]]></licenseTemplate>
        <copyRightYear>1111</copyRightYear>
        <ownerName>LICENSE-OWNER-NAME</ownerName>
        <classJavaDoc><![CDATA[Class Detail Comment $S
]]></classJavaDoc>

        <sinces>
            <since>1.0</since>
        </sinces>
        <versions>
            <version>1.0</version>
        </versions>
        <authors>
            <author>AUTHOR-NAME</author>
        </authors>
    </javaPoetCommand>

    <javaPoetCommand value="org.vermeerlab.apt.element.test.command.javadoc.GenerateJavaDocCommand">
        <licenseTemplate><![CDATA[
Copyright [CopyRightYear]  [name of copyright owner]
MIT License
]]></licenseTemplate>
        <copyRightYear>2222</copyRightYear>
        <ownerName>LICENSE-OWNER-NAME-GenerateCode</ownerName>

        <!-- JavaPoet による置換可能な文字列が指定できます -->
        <classJavaDoc><![CDATA[Class Detail Comment-GenerateCodeTestCommand $S
]]></classJavaDoc>

        <sinces>
            <since>1.1</since>
        </sinces>
        <versions>
            <version>1.1</version>
        </versions>
        <authors>
            <author>AUTHOR-NAME-GenerateCode</author>
        </authors>
    </javaPoetCommand>

    <!-- Annotation Processor JavaPoet end -->
</root>

```

##### 全てのコマンドに共通した設定

何も設定をしない.

```xml
<javaPoetCommand> 
```

##### 実行コマンドに共通した設定

`value`にコマンドクラスの完全修飾子で指定してください.

```xml
<javaPoetCommand value="command class path">
```
`sinces`,`versions`,`authors` は「全コマンド共通設定」に追記をします.



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

    <processorcommand.classpath>/processor-command.xml</processorcommand.classpath>
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
        <groupId>${project.groupId}</groupId>
        <artifactId>annotation-processor-command-propertyfile-enum</artifactId>
        <version>0.1.0</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest-core</artifactId>
        <version>1.3</version>
        <scope>test</scope>
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

### 定義用XMLのパス指定

コマンドを使用するプロダクト側では、定義用XMLのパスを SystemPropertyの`processorcommand.classpath`に指定します.

pom.xmlで指定する例

```xml
<properties>
    <processorcommand.classpath>/processor-command.xml</processorcommand.classpath>
</properties>


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
```

拡張プロジェクト側で指定は不要です.



### テスト

定義用XMLをテストで使用するために指定できます.

```java
public void test() {
    Truth.assert_()
            .about(JavaSourceSubjectFactory.javaSource())
            
            // (1) 参照するクラスのパス
            .that(JavaFileObjects.forResource(Resources.getResource(
                    "org/vermeerlab/apt/command/propertyfileenum/LocaleSetSample.java"
            )))

            // (2) デバッグ出力と定義用XMLパス
            .processedWith(new AnnotationProcessorController(false, "/processor-command.xml"))
            .compilesWithoutError()
            .and()

            // (3) 検証するクラスのパス
            .generatesSources(SourceFileReader.of(Resources.getResource(
                    "org/vermeerlab/apt/command/propertyfileenum/localesetsample/Message3.java")
            ).toJavaFileObject());
}
```

#### (1) 参照するクラスのパス

AnnotationProcessorで参照するクラスパスとして、`test/resources/` 以降の相対パスを指定してください.


#### (2) デバッグ出力と定義用XMLパス

デバッグ出力（boolean）を`true`にすると、標準出力に デバッグ情報を出力します.

定義用XMLのパスとして、`test/resources/` 以降の相対パスを指定してください.

#### (3) 検証するクラスのパス

コマンドの目的がコード生成の場合、結果として期待するコードファイルを指定することで検証できます.

クラスパスとして、`test/resources/` 以降の相対パスを指定してください.



---

## Sample

実際に拡張したコマンドのプロジェクトをサンプルとして実装の詳細を示します.

### Code

[PropertyファイルからEnumクラスを生成](https://bitbucket.org/vermeerlab/apt-propertyfile-enum)


### コマンドクラス

`GenerateEnumFromPropertyFileCommand`

#### `getTargetAnnotation`

AnnotationProcessorのトリガーとなるアノテーション(`GenerateEnumFromPropertyFile.class`)を戻り値となるように実装します.

```java
public Class<? extends Annotation> getTargetAnnotation() {
    return GenerateEnumFromPropertyFile.class;
}
```


#### `getExtentionCommandConfigScannerClasses`

定期用XML情報の取得クラスのリストが戻り値となるように実装します.

```java
protected List<Class<? extends CommandConfigScannerInterface>> getExtentionCommandConfigScannerClasses() {
    return Arrays.asList(ConfigScanner.class);
}
```

##### Note

定義用XMLから取得する情報にあわせてクラスを作成することを踏まえて複数指定できる実装にしています.


#### `execute`

コマンドのメインとなる機能の実装です.

```java
public void execute(ProcessingEnvironment processingEnvironment, Element element, Boolean isDebug) {

    // (1) アノテーションからルート要素を取得
    ClassElement classElement = CategorizedElementKind.createCategorizedElement(
            processingEnvironment, element, this.getTargetAnnotation());
    
    // (2) デバッグ用のクラス
    DebugUtil debugUtil = new DebugUtil(isDebug);

    // (3) アノテーションされたクラスからの情報取得
    TargetPropertyFileConverter targetPropertyFileConverter = TargetPropertyFileConverter.of(classElement);
    EnumResourceLocaleConverter localeConverter = EnumResourceLocaleConverter.of(classElement);
    EnumResourceControlConverter controlConverter = EnumResourceControlConverter.of(classElement);

    // (4) 検証実施
    CommandValidator.of(this.getTargetAnnotation())
            .validate(targetPropertyFileConverter, controlConverter, localeConverter);

    // (5) 定義用XMLから情報取得
    ConfigScanner scanner = this.getCommandConfigScanner(ConfigScanner.class).get();

    // (6) 定義用クラスから取得した情報で更新
    Config config = scanner.updateLocaleControlByAnnotation(localeConverter, controlConverter);

    // (7a) 出力クラスのパッケージ名を編集
    String packageName = this.toPackageName(classElement);
    
    targetPropertyFileConverter.values().stream()
            // (8) PropertyFile単位でJavaCodeを生成
            .map(targetPropertyFileFieldElement -> {
                return JavaFileGenerator.of(this, targetPropertyFileFieldElement, config, packageName).createJavaPoetFile();
            })
            // (9) テスト用の標準出力
            .peek(javaPoetFile -> debugUtil.print(javaPoetFile.toString()))

            // (10) コード出力
            .forEach(javaPoetFile -> javaPoetFile.writeTo(processingEnvironment));
}

//(7b)
String toPackageName(ClassElement classElement) {
    Optional<GenerateEnumFromPropertyFile> generateEnumFromPropertyFile = classElement.getAnnotation(
            GenerateEnumFromPropertyFile.class);
    String packageName = classElement.toPackageName(
            generateEnumFromPropertyFile.get().basePackageName(),
            generateEnumFromPropertyFile.get().subPackageName()
    );
    return packageName;
}
```


##### (1) アノテーションからルート要素を取得

コマンドクラスの`#getTargetAnnotation`で指定したアノテーションを付与した要素からELementの情報を取得します.

このアノテーションで特定したクラスを本コマンドにおける「定義用クラス」として扱います.

`GenerateEnumFromPropertyFile.class`はClass属性の要素にのみ注釈できるインターフェースなので、抽出要素がClass属性であることが自明であるので、戻り値の型を `ClassElement`と指定しています.


##### (2) デバッグ用のクラス

テスト時に標準出力制御をしたい場合に使用するクラスのインスタンス化です.

「(9) テスト用の標準出力」のように使用します.


##### (3) アノテーションされたクラスからの情報取得

アノテーションにより取得した情報を保持するクラスに `org.vermeerlab.apt.command.ValidationInterface`を介して検証ロジックも併せて実装します.

###### `TargetPropertyFileConverter`

EnumクラスのインプットとなるPropertyFileのアノテーション `org.vermeerlab.apt.command.propertyfileenum.annotaion.TargetPropertyFile`で取得した情報を扱うクラスです.

```java
static TargetPropertyFileConverter of(ClassElement classElement) {
    FieldElements fieldElements = classElement.children(TargetPropertyFile.class).filter(ElementKind.FIELD);
    TargetPropertyFileFieldElements targetFieldElements = TargetPropertyFileFieldElements.of(fieldElements);
    return new TargetPropertyFileConverter(targetFieldElements);
}
```

インスタンス時に、ルートとして取得したクラスから `TargetPropertyFile`が注釈されているフィールドから情報を取得しています.


```java
public ValidationResult validate() {
    ValidationResult result = this.targetPropertyFileFieldElements.validateRequire();
    if (result.isValid() == false) {
        return result;
    }
    return this.targetPropertyFileFieldElements.validateDuplicate();
}
```

検証ロジックの実装です.

* 必須判定
* 項目値が重複検証

をしています.


###### `EnumResourceLocaleConverter`

Enumクラスで使用するLocaleを指定するアノテーション `org.vermeerlab.apt.command.propertyfileenum.annotaion.EnumResourceLocale`で取得した情報を扱うクラスです.



```java
static EnumResourceLocaleConverter of(ClassElement classElement) {
    MethodElements methodElements = classElement.children(EnumResourceLocale.class).filter(ElementKind.METHOD);
    return new EnumResourceLocaleConverter(methodElements);
}
```

インスタンス時に、ルートとして取得したクラスから `EnumResourceLocale`が注釈されているメソッドから情報を取得しています.

```java
public ValidationResult validate() {
    ValidationResult result = ValidationResult.create();
    if (this.methodElements.isEmpty()) {
        return result;
    }
    result.append(this.methodElements.validateTargetFieldOneOrLess());
    if (result.isValid() == false) {
        return result;
    }
    result.append(this.methodElements.validateReturnTypeIsSame(Locale.class));
    return result;
}
```

検証ロジックの実装です.

* 注釈メソッドが１つ以下
* メソッドの型検証

をしています.


```java
Locale value() {
    if (this.methodElements.isEmpty()) {
        return null;
    }
    return (Locale) this.methodElements.values().get(0).value();
}
```

メソッドを動的にコンパイルして実行した結果を返却しています.



```java
String codeBlock() {
    if (this.methodElements.isEmpty()) {
        return null;
    }
    return this.methodElements.values().get(0).getMethodBlock();
}
```

メソッドの実装ソースコードを返却しています.

（`EnumResourceControlConverter`は ほぼ同じなので割愛します.）


###### (4) 検証実施

`org.vermeerlab.apt.command.ValidationInterface`の実装クラスを一括して検証します.

エラーがあった場合は、全ての検証結果を保持した `AnnotationProcessorException`を返却します.


###### (5) 定義用XMLから情報取得

当該コマンド用に追加した定義用XMLの情報を取得します.


###### (6) 定義用クラスから取得した情報で更新

定義用XMLから取得した情報に、定義用クラスで設定した情報を上書きします.

これにより 共通設定は定義用XML、個別指定を定義用クラスで指定する、という機能を実現しています.


###### (7) 出力クラスのパッケージ名を編集

定義用クラスからパッケージ名に関する情報を取得して編集します.

定義用クラス名、`org.vermeerlab.apt.command.propertyfileenum.annotaion.GenerateEnumFromPropertyFile`による指定 から導出しています.


###### (8) PropertyFile単位でJavaCodeを生成

定義用クラスで`TargetPropertyFile`により指定したPropertyFileを全て処理対象としています.

１つの定義用クラスで複数のPropertyFileを指定する事で、複数のEnumクラスを一度に作成できます.


###### (9) テスト用の標準出力

コンソールに生成コードを出力して結果確認ができます.


###### (10) コード出力

編集したコードをJavaファイルとして出力します.



### 定義用XMLと参照クラス

#### 定義用XML（拡張）

`<propertyFileEnum>`以降が拡張したXML定義です.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <javaPoetCommand>
    <!-- 省略 -->
    </javaPoetCommand>

    <propertyFileEnum>
        <!--
        true: return key and value, false: return value only
        default is "false"
        -->
        <withKey>true</withKey>

        <!--
        Property KeyをEnum列挙子に変換する際に付与する文字列
        "prefix" + property-key + "suffix"
        -->
        <propertyKey>
            <prefix>【</prefix>
            <suffix>】</suffix>
        </propertyKey>

        <!--
        If exception throws when getting a resource bundle
        true: return key and value, false: return key only
        defaut is "false"
        -->
        <withValueWhenException>true</withValueWhenException>

        <!--
        true: default locale(user enviroment locale)
        false: Enum code comment's locale set by xml or annotaion.
        default is "true"
        -->
        <withCodeCommentDefaultLocale>false</withCodeCommentDefaultLocale>

        <!--
        Locale.ROOT: edit <locale>/<language>
        Locale.default: no edit <locale>
        -->
        <locale>
            <!-- ex: ja, en ...-->
            <language>en</language>
            <!-- ex: JP, US ...-->
            <country>US</country>
        </locale>
    </propertyFileEnum>
</root>
```

#### 参照クラス

定義用XMLの拡張部分を参照するために`CommandConfigScannerInterface`を実装します.

`configXmlReader.scanTextValue`は`xpath`で定義用XMLから情報を取得するメソッドです.

`#readConfig`の戻り値は`this`にしてください.

情報取得はAnnotationProcessorの`#init`で取得して、`GenerateEnumFromPropertyFileCommand`で保持した状態なので`#getCommandConfigScanner`で参照クラス（'ConfigScanner.class'）を指定して参照します.


```java
public class ConfigScanner implements CommandConfigScannerInterface {

    private Config config;

    /**
     * {@inheritDoc }
     */
    @Override
    public CommandConfigScannerInterface readConfig(ConfigXmlReader configXmlReader) {
        Iterator<String> keyPrefixList
                         = configXmlReader.scanTextValue("//propertyFileEnum/propertyKey/prefix/text()").iterator();
        String keyPrefix = keyPrefixList.hasNext()
                           ? keyPrefixList.next()
                           : "";

        Iterator<String> keySuffixList
                         = configXmlReader.scanTextValue("//propertyFileEnum/propertyKey/suffix/text()").iterator();
        String keySuffix = keySuffixList.hasNext()
                           ? keySuffixList.next()
                           : "";

        Iterator<String> withKeyList
                         = configXmlReader.scanTextValue("//propertyFileEnum/withKey/text()").iterator();
        Boolean withKey = withKeyList.hasNext()
                          ? Boolean.valueOf(withKeyList.next())
                          : false;

        Iterator<String> withValueWhenExceptionList
                         = configXmlReader.scanTextValue("//propertyFileEnum/withValueWhenException/text()").iterator();
        Boolean withValueWhenException = withValueWhenExceptionList.hasNext()
                                         ? Boolean.valueOf(withValueWhenExceptionList.next())
                                         : false;

        Boolean hasLanguage = configXmlReader.hasNode("//propertyFileEnum/locale/language");
        String localeLanguage = null;
        String localeCountry = null;
        if (hasLanguage) {
            Iterator<String> localeLanguageList
                             = configXmlReader.scanTextValue("//propertyFileEnum/locale/language/text()").iterator();

            localeLanguage = localeLanguageList.hasNext()
                             ? localeLanguageList.next()
                             : "";

            Iterator<String> localeCountryList
                             = configXmlReader.scanTextValue("//propertyFileEnum/locale/country/text()").iterator();
            localeCountry = localeCountryList.hasNext()
                            ? localeCountryList.next()
                            : "";
        }

        Iterator<String> withCodeCommentDefaultLocaleList
                         = configXmlReader.scanTextValue("//propertyFileEnum/withCodeCommentDefaultLocale/text()").iterator();
        Boolean withCodeCommentDefaultLocale = withCodeCommentDefaultLocaleList.hasNext()
                                               ? Boolean.valueOf(withCodeCommentDefaultLocaleList.next())
                                               : true;

        this.config = new Config(withKey, keyPrefix, keySuffix, withValueWhenException,
                                 localeLanguage, localeCountry, null, null, null, withCodeCommentDefaultLocale);
        return this;
    }

    //
    Config updateLocaleControlByAnnotation(EnumResourceLocaleConverter localeConverter, EnumResourceControlConverter controlConverter) {
        return config.updateLocaleControlByAnnotation(localeConverter, controlConverter);
    }

}
```


---
## Code
[BitBucket](https://bitbucket.org/vermeerlab/apt-javapoet)


## Version
* 0.1.0  
初回リリース

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)

