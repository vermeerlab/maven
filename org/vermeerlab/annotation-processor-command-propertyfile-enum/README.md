Generate EnumClass from Properties ProcessorCommand
===

## Description

`Pluggable Annotation Processing API`を使用して、ファイルからEnumクラスを生成します.

本プロジェクトを使用することで以下のことが実現できます.

* Propertiesファイルへの参照を統一的に行えます
* Propertiesファイルを参照する実装がタイプセーフになります
* 実装中にPropertiesの値を確認できます（IDEによるJavaDoc参照を経由）
* 国際化対応（実装中、稼働中）の制御ができます
* 指定した置換文字数がPropertiesの値の内容と相違している場合は実行時例外にします


### 統一的な参照

生成したコード内にて `ResourceBundle`により`PropertyFile`への参照を行うので
`ResourceBundle`による参照ユーティリティを独自に作成する必要はありません.

### タイプセーフ

`PropertyKey`がEnumの列挙子になることで文字列で指定する必要がなくなります.

* old

```java
ResourceBundle bundle = ResourceBundle.getBundle("message.properties");
System.out.println(bundle.getString("msg001"));
```

* new

```java
System.out.println(Message.MSG001);
```

### JavaDoc

Property値をJavaDocに出力するので、IDEの補完機能により メッセージを管理する `message.properties`本体を確認しなくても内容が確認できます.

```java
public enum Message {
/**
 * メッセージ００１
 * <p>
 * parameter count = 0
 */
MSG001("msg001", 0, "メッセージ００１")
```

### 国際化対応

#### 生成コードにおける国際化

JavaDocコメントの国際化です.

生成コードのJavaDocを開発者に合わせて国際化できると
JavaDocのコメントが母語でない場合のストレスの軽減が期待されます.

デフォルトはデフォルトロケールなので、ビルド都度 JavaDocは 開発者のロケールにあわせて作成されます.

`Locale.JAPANESE`
```java
/**
 * メッセージ００１
 * <p>
 * parameter count = 0
 */
MSG001("msg001", 0, "メッセージ００１")
```

`Locale.ENGLISH`
```java
/**
 * message001
 * <p>
 * parameter count = 0
 */
MSG001("msg001", 0, "message001")
```


#### 実行参照時の国際化

未指定の場合はコード生成と同様にデフォルトロケールです.

実行時のEnumクラスに対して、静的なロケール指定と Enum列挙子への参照都度 適用する 動的なロケール指定も可能です.


### 置換文字数が不一致の場合の実行時例外

メッセージを出力した際にパラメータ数が異なっているために不適当なメッセージが出力されるケースがあります.
メッセージは時にシステムの内部情報を暴露することになりかねません. 実行時例外にすることで そのような状況を防御できることが期待されます.


---

## 使用方法

### 設定例（最小構成）

最小限の指定で実現される機能.

* 生成されるクラスは、自動生成トリガーのJavaクラスを小文字にした文字をパッケージにして`PropertyFile`名をクラス名にする
* `Licence`記載は無し
* クラスJavaDoc記載は無し
* Emun列挙子のJavaDocはデフォルトロケールで作成
* 出力する値は `PropertyFile` の `value`のみ（`Key`は出力しない）
* 実行時のLocaleの初期設定はデフォルトロケール（実行環境のLocale）
* 実行時例外で出力する情報は `PropertyFile` の `key`のみ

#### Code

`https://bitbucket.org/vermeer_etc/resource-enum-user.git`

cloneしてビルドをすると コードの自動生成も含めて実行されます.

ビルド前は 自動生成するコードの参照箇所がエラーになっていますが、ビルド後は解消されます.

#### 定義用Javaクラス

参照する`PropertyFile`に関する情報および、出力生成に関する情報を記述するクラスです.

```java

@GenerateEnumFromPropertyFile
public class Config {

    @TargetPropertyFile
    final String resourceName = "message";

}

```

##### @GenerateEnumFromPropertyFile

`PropertyFile`から Enumクラスを作成する定義用Javaクラスに注釈してください.


##### @TargetPropertyFile

参照する`PropertyFile`のベース名（Localeを含まない`PropertyFile`名）を指定します（必須）.


#### 生成されるクラス

デフォルトでは、
パッケージパスは「自動生成したクラスのパス ＋ 自動生成クラス名（小文字変換） ＋ リソース名（先頭１文字を大文字）」です.

`org.vermeerlab.sample.resourceenum.Config`で `message`を指定した場合、
生成されるクラスは `org.vermeerlab.sample.resourceenum.config.Message`

生成されたクラスは 以下のフォルダに作成されます.

```
target/generated-sources/apt/org/vermeerlab/sample/resourceenum/config
```

#### 生成したクラスの利用例

使用例としてテストクラスを示します.

```java
@Test
public void messageToString() {
    Assert.assertThat(Message.MSG001.toString(), is("メッセージ００１"));
}

@Test
public void messageFormatEnglish() {
    Assert.assertThat(Message.MSG001.format(Locale.ENGLISH), is("message001"));
}
```

`#toString`は デフォルトロケール（日本語）を参照した値になります.

`#format`は パラメータで指定したLocaleから `PropertyFile`を参照しています.


---
### 詳細説明

`xml`で共通設定を行い、定義用Javaクラスで個別設定が行えます.

クラスの設定は `xml`の設定より優先されます.


#### 定義用Xmlファイルの説明

pom.xml で参照する定義ファイルパスを指定してください.

```xml
<properties>
    <processorcommand.classpath>/processor-command.xml</processorcommand.classpath>
</properties>
```

記述しない場合のデフォルト値は各タグのコメントを参照してください.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <!-- Annotation Processor JavaPoet start -->
    <!--
    コード生成に関する基本設定
    -->
    <!-- 共通設定 -->
    <javaPoetCommand>

        <!-- 生成するコードのヘッダーに記載するライセンス情報 -->
        <licenseTemplate><![CDATA[
Copyright [CopyRightYear]  [name of copyright owner]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
]]></licenseTemplate>


        <!-- ライセンス情報の ` [CopyRightYear]` を置換する文字列 -->
        <copyRightYear></copyRightYear>

        <!-- ライセンス情報の ` [name of copyright owner] を置換する文字列 -->
        <ownerName></ownerName>

        <!--
        Class JavaDoc Comment
        -->
        <classJavaDoc><![CDATA[]]></classJavaDoc>

        <sinces>
            <since></since>
        </sinces>

        <versions>
            <version></version>
        </versions>

        <authors>
            <author></author>
        </authors>

    </javaPoetCommand>

    <!-- コマンド個別設定 -->
    <!--
    `value`で指定したProcessor Commandによる処理対象にのみ適用される定義
    -->

    <javaPoetCommand value="org.vermeerlab.apt.command.propertyfileenum.GenerateEnumFromPropertyFileCommand">
        <!-- $S replace resource name -->
        <classJavaDoc><![CDATA[PropertyFile Enum
<P>
Base resource name is $S.
]]></classJavaDoc>
    </javaPoetCommand>

    <!-- Annotation Processor JavaPoet end -->

    <!--
    GenerateEnumFromPropertyFileCommand にて使用するパラメータ
    -->
    <propertyFileEnum>
        <!--
        true: return key and value
        false: return value only
        default is "false"
        -->
        <withKey></withKey>

        <!--
        "keyPrefix" + property-key + "keySuffix"
        -->
        <propertyKey>
            <prefix></prefix>
            <suffix></suffix>
        </propertyKey>

        <!--
        If exception throws when getting a resource bundle.
        true: return key and value
        false: return key only
        defaut is "false"
        -->
        <withValueWhenException></withValueWhenException>

        <!--
        true: default locale(user enviroment locale)
        false: Enum code comment's locale set by xml or annotaion.
        default is "true"
        -->
        <withCodeCommentDefaultLocale></withCodeCommentDefaultLocale>

        <!--
        Locale.ROOT: edit <locale>/<language>
        Locale.default: no edit <locale>
        -->
        <locale>
            <!-- ja, en ...-->
            <language></language>
            <!-- JP, US ...-->
            <country></country>
        </locale>

    </propertyFileEnum>

</root>

```

##### javaPoetCommand

コード生成に関する設定要素です.

###### `value`

未指定は共通設定です.

実行コマンド毎に異なる設定をする場合は、`value`に実行コマンドのクラスパスを記述します.


###### `licenseTemplate`、`copyRightYear`、`ownerName`

クラスファイルヘッダーに共通して記述しておきたいライセンス情報を設定します.

実行コマンドで指定した場合は、共通設定を 上書きします.

###### `classJavaDoc`、`sinces`、`versions`、`authors`

クラスのJavaDocに関する設定です.

`classJavaDoc` は共通設定を上書きします.

記述内容は`JavaPoet`の仕様に則って要素の置換（`$S` などによる置換制御）が可能です. 置換制御はコマンドクラスの実装に依存します.

`sinces`、`versions`、`authors` は共通設定に追記します.

クラスのJavaDocについては、生成ファイル毎にヘッダーコメントを編集する事ができます（後述）.



##### propertyFileEnum

Enumクラス生成コマンドの実行に使用する定義です.

###### 出力するキーおよび値への共通制御

* `withKey`

出力値に `Property Key`を付与したい場合に使用します.

例えば、出力メッセージにID（`Property key`）を付与することを想定しています.

`message.properties`

```
msg001=message1
```

true: `msg001message1`

false: `message1`


（後述の接頭・接尾文字を付与すれば可読性は向上します）


* `prefix`、`suffix`

キー値に付与する文字を指定します.

例えば、出力メッセージにID（`Property key`）を付与する際、メッセージとの区切りを設けたい場合に使用します.

`prefix`: `【`

`suffix`: `】`

出力情報: `【msg001】message1`


* `withValueWhenException`

`ResourceBundle`による`PropertyFile`参照時に例外が発生した場合に出力する情報に Enumクラスに保持している値を出力したい場合に指定します.

値を出力する事で 例外が発生した際の情報が多くなるメリットがありますが、Webメッセージのように利用者の目にする情報に文字列が置換されていない情報が出力されるというデメリットがあります.
そのようなケースを想定して、出力メッセージに値を出力しない（ID（`Property key`）のみを出力する）制御を行えるようにしており、デフォルトは `false`（出力メッセージに値を出力しない）です.

###### 生成コードのコメントLocale

* `withCodeCommentDefaultLocale`

生成したコードのEnum列挙子のコメントに適用するLocaleに Enumクラス参照時と同じLocale指定を使用したい場合に設定します.

`true` の場合は 生成コードのコメントおよび `value` は デフォルトロケールで取得した値です.

`false`の場合は 指定したLocaleに従って取得した値を コメントおよび `value`に設定します.


###### Localeの設定

`xml`の設定は 生成されるEnumクラス全てに適用されます.

生成後のクラスが参照するLocaleの初期設定にも使用します.

* 基本

`language`: 言語の指定（ja, en ...）
<!-- JP, US ...-->
`country`: 国の指定（JP, US ...）

* Locale.ROOT

無印の`PropertyFile`を指定したい場合は空タグを記述してください.

```xml
<locale>
    <language></language>
    <country></country>
</locale>
```


* Locale.Default

デフォルトロケール（実行環境のロケール）を使用する場合は、属性を記述しないでください.




#### 定義用Javaクラスの説明

クラスの設定は`xml`による設定よりも優先されます.

全てのアノテーションを設定した記載例です.

```java
@TargetPropertyFile(classCommentTitle = "commentHeader", className = "Config", classNamePrefix = "Prefix", classNameSuffix = "Suffix")
@PropertyValueWithKey(true)
@PropertyKeyPrefix("【")
@PropertyKeySuffix("】")
@PropertyValueWithValueWhenException(true)
final String resourceName = "message";

@EnumResourceLocale
final Locale locale() {
    return Locale.ENGLISH;
}

@EnumResourceControl
final Control control() {
    return java.util.ResourceBundle.Control.getNoFallbackControl(
            java.util.ResourceBundle.Control.FORMAT_DEFAULT);
}

```

* Code

```
https://bitbucket.org/vermeer_etc/resource-enum-user-allset.git
```

##### @GenerateEnumFromPropertyFile

クラスに付与することで、本コマンドの参照対象と認識されます.

パラメータにより作成するクラスのパッケージを変更できます.

デフォルトは クラス名を小文字にしたパッケージをサブパッケージしたものです.

例：

定義クラス：`hoge.Fuga.java`、参照PropertyFile：`message.properties`

生成クラス：`hoge.fuga.Message.java`



##### @TargetPropertyFile

生成時に参照する`PropertyFile`を指定します.

必須です.

ベース名（Locale記述の無い）ファイルパス（フォルダに格納していたらフォルダ名も含める）を指定してください. 拡張子（`.properties`）は不要です.


###### classCommentTitle

`xml`で指定した `classJavaDoc` の先頭に記載するコメントを指定できます.

`PropertyFile`のファイル名だけでは判別しづらい分類（用途）について クラスのJavaDocに識別できる情報を出力することで 実装時の手助けになります.


###### クラス名

`classNamePrefix`、`className`、`classNameSuffix` を使用すれば `PropertyFile`の先頭大文字にした文字列以外のクラス名で出力できます.

##### その他

複数の `PropertyFile` を１つの定義クラスで指定できます. フィールド名は自動生成時に使用しませんので どのような値でも問題ありません.

```java

@TargetPropertyFile
final String resourceName1 = "message1";

@TargetPropertyFile
final String resourceName2 = "message2";

```

##### xml設定を上書き

`xml`によるコマンド実行全体の共通設定を `PropertyFile`単位で変更したい場合に指定します.

`@PropertyValueWithKey`：`<withKey>`

`@PropertyKeyPrefix`：`<prefix>`

`@PropertyKeySuffix`：`<suffix>`

`@PropertyValueWithValueWhenException`：`<withValueWhenException>`


##### @EnumResourceLocale

コード生成時に`ResourceBundle`で`PropertyFile`を参照する際に使用する`Locale`を指定できます.

本指定は参照するLocaleの指定を限定したい場合に使用します. 例えば、プロダクト資産のビルドなど 開発環境の設定に依存させたくない場合に、xml設定の `<withCodeCommentDefaultLocale>`と組み合わせて使用する事を想定しています.



```java
@EnumResourceLocale
public Locale getLocale() {
    return Locale.ENGLISH;
}

```

##### @EnumResourceControl

コード生成時に`ResourceBundle`で`PropertyFile`を参照する際に使用する`Control`を指定できます.

`fallback`の指定など、対象ロケールのファイルが存在しない場合の制御に、Controlで制御したいことを実装してください.

```java
@EnumResourceControl
public Control getControl() {
    return ResourceBundle.Control.getNoFallbackControl(ResourceBundle.Control.FORMAT_DEFAULT);
}
```

##### 補足

`@EnumResourceLocale` と `@EnumResourceControl` のメソッド内のクラス（型）は自動生成後のクラスに動的に展開しています. ビルドエラーになった場合は 完全修飾名に修正してください.


#### 生成されたEnumクラスの説明

##### setStaticLocale

Enumクラスで実行時に `PropertyFile`を参照する際に使用するLocaleを設定します.

`xml`や定義用Javaクラスで指定したLocaleはあくまで初期値であり、本メソッドで更新できます.

ビルド時に指定した初期値とは別の`Locale`に変更は出来ますが、staticフィールド値の更新なのでクラスローダー単位で変更されます. Webシステムなどのマルチスレッド環境で、メッセージ毎のロケール変更には使用しないでください.


##### setStaticControl

Enumクラスで実行時に `PropertyFile`を参照する際に使用するControlを設定します.

定義用Javaクラスで指定したControlはあくまで初期値であり、本メソッドで更新できます.

ビルド時に指定した初期値とは別の`Control`に変更は出来ますが、staticフィールド値の更新なのでクラスローダー単位で変更されます. Webシステムなどのマルチスレッド環境で、メッセージ毎のロケール変更には使用しないでください.


##### format

置換文字を指定して値の取得、編集をします.

Webシステムのようにクライアントロケールに合わせた出力をしたい場合に、LocaleやControlを都度指定できます.

```java
Locale locale = ServletRequest.getLocale();
System.out.println(Message.MSG001.format(locale));
```

##### toString

`Property Value`をそのまま出力します. Enumクラスに設定されている Locale、Controlにしたがった値を出力しますが 置換文字列があった場合は 置換せず元情報のまま出力します（置換文字パラメータの指定がないため）.

##### その他

`PropertyFile`参照する際、実行時例外が発生した場合は Enum列挙子の `key` を出力します.

ただし、`xml`の`<withValueWhenException>`が`true`の場合、または定義用Javaクラスの`@PropertyValueWithValueWhenException` で `true`を指定した場合は、
`key`と`value`を出力します.

---

#### クラス生成をしたプロジェクトを使用

`Annotation Processor` で作成したクラスは通常のクラスと同様に`jar`に格納されています.

`pom.xml` で依存プロジェクトとして指定すれば使用できます.

使用するだけであれば、生成プロジェクトにて指定した設定は不要です.

サンプルコードを使用する前提として、以下のコード（クラス生成をするプロジェクト）を事前にビルドしておいてください.

```
https://bitbucket.org/vermeer_etc/resource-enum-user.git
```

クラス生成をしたプロジェクトを使用するクライアントコード
```
https://bitbucket.org/vermeer_etc/resource-enum-user-client.git
```

pom.xmlで指定する依存情報

```xml
<dependency>
    <groupId>${project.groupId}</groupId>
    <artifactId>resource-enum-user</artifactId>
    <version>0.1.0</version>
</dependency>
```

`maven-compiler-plugin` に `Annotation Processor`に関する記述は不要です.

生成したクラスの参照

```java
public class Client {

    public String getMessageEnglish() {
        return Message.MSG001.format(Locale.ENGLISH);
    }

    public String getToString() {
        return Message.MSG001.toString();
    }
}
```

出力情報の確認のためのテストクラス

```java
@Test
public void messageEnglish() {
    Assert.assertThat(new Client().getMessageEnglish(), is("message001"));
}

@Test
public void messageToString() {
    //デフォルトロケールが選択される
    Assert.assertThat(new Client().getToString(), is("メッセージ００１"));
}
```

クライアントコードのプロジェクトに `PropertyFile`が無くても 依存先のプロジェクトから参照します（クラスパス上の資産を参照するため）.


---

## 実装例

想定される いくつかのユースケースのサンプルです.

### 出力メッセージにIDを付与

ID（`Property Key`）をメッセージに付与します.

`xml`で共通設定をして、クラスで資産ごとに異なる設定をしています.

#### Code

```
https://bitbucket.org/vermeer_etc/resource-enum-user-messagekey.git
```

#### 定義用Xmlファイルの説明

共通の設定は `xml` で行います

```xml
<propertyFileEnum>
    <withKey>true</withKey>
    <propertyKey>
        <prefix><![CDATA[<<]]></prefix>
        <suffix><![CDATA[>>]]></suffix>
    </propertyKey>
</propertyFileEnum>
```

#### 定義用Javaクラスの説明

`properties.message` は共通設定を そのまま適用し、`properties.message2` は接頭・接尾文字を変更しています.

```java
@TargetPropertyFile
final String resourceName = "properties.message";

@TargetPropertyFile
@PropertyKeyPrefix("【")
@PropertyKeySuffix("】")
final String resourceName2 = "properties.message2";
```

#### 生成クラスの動作確認

挙動の違いをテストにて示します.

指定の違いにより、Keyのカッコが異なっていることが確認できます.

```java
Assert.assertThat(Message.MSG001.toString(), is("<<msg001>>メッセージ００１"));
Assert.assertThat(Message.MSG001.format(Locale.ENGLISH), is("<<msg001>>message001"));
Assert.assertThat(Message2.MSG001.toString(), is("【msg001】ルートメッセージ２０１"));
Assert.assertThat(Message2.MSG001.format(Locale.ENGLISH), is("【msg001】ルートメッセージ２０１"));
```


### ライセンス記述とクラスコメントの設定

`xml`で生成する全てのコードに共通する記載を行い、実行コマンド毎に変更したいクラスコメントを編集します.

加えて リソース毎のクラスヘッダーも追記します.

#### Code

```
https://bitbucket.org/vermeer_etc/resource-enum-user-classjavadoc.git
```

#### 定義用Xmlファイルの説明

全ての自動生成に関する定義は `value`指定なし を使用

実行コマンド毎に関する定義は `value`に対象コマンドを指定

ここでは、全体共通定義として Licenceヘッダーを記述するようにして、クラスJavaDocをコマンド毎に指定しています.

実行コマンドの `GenerateEnumFromPropertyFileCommand` は クラスJavaDocの中の `$S` に対して参照リソースパスに文字列置換を行う内部実装をしています（`JavaPoet`を使用）. 

```xml
    <!-- 共通設定 -->
    <javaPoetCommand>

        <!-- 生成するコードのヘッダーに記載するライセンス情報 -->
        <licenseTemplate><![CDATA[
Copyright [CopyRightYear] [name of copyright owner]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
]]></licenseTemplate>


        <!-- ライセンス情報の ` [CopyRightYear]` を置換する文字列 -->
        <copyRightYear>2017</copyRightYear>

        <!-- ライセンス情報の ` [name of copyright owner] を置換する文字列 -->
        <ownerName><![CDATA[Your name]]></ownerName>

    </javaPoetCommand>

    <!-- コマンド個別設定 -->
    <!--
    `value`で指定したProcessor Commandによる処理対象にのみ適用される定義
    -->

    <javaPoetCommand value="org.vermeerlab.apt.command.propertyfileenum.GenerateEnumFromPropertyFileCommand">
        <!-- $S replace resource name -->
        <classJavaDoc><![CDATA[PropertyFile Enum
<P>
Base resource name is $S.
]]></classJavaDoc>
    </javaPoetCommand>
    
</root>

```

#### 定義用Javaクラスの説明

クラスコメントのヘッダーを編集できます.

参照資産の用途概要などを記述すると良いでしょう.

本指定の後に `xml` で指定したコメントが続きます.

また、LocaleやControlについて同じ指定で良い場合、複数のリソースを１つの定義用Javaクラスで指定できます.

```java

@TargetPropertyFile(classCommentTitle = "SystemA Message")
final String systemA = "properties.systema";

@TargetPropertyFile(classCommentTitle = "SystemB Message")
final String systemB = "properties.systemb";

```

#### 生成されたクラス

Licence
```java
//
// Copyright 2017  Your name
//
// Licensed under the Apache License, Version 2.0 (the "License");
...以下略
```


クラスJavaDoc
```java
/**
 * SystemA Message
 * <P>
 * PropertyFile Enum
 * <P>
 * Base resource name is "properties.systema".
 *
 */
```


### RootLocaleの指定（xml）

デフォルトロケールではなく（実行環境のロケールではなく）、ルートロケール（ロケール指定なしの`PropertyFile`）を指定したい場合の設定です.

#### Code
```
https://bitbucket.org/vermeer_etc/resource-enum-user-rootlocalexml.git
```

#### 定義用Xmlファイル による共通指定

```xml
<propertyFileEnum>
    <locale>
        <!-- ja, en ...-->
        <language></language>
        <!-- JP, US ...-->
        <country></country>
    </locale>
</propertyFileEnum>
```

タグのみを記載することで「空白文字列指定」（`Locale("","")`）と同じです.



### RootLocaleの指定（Java）

デフォルトロケールではなく（実行環境のロケールではなく）、ルートロケール（ロケール指定なしの`PropertyFile`）を指定したい場合の設定です.


#### Code
```
https://bitbucket.org/vermeer_etc/resource-enum-user-rootlocalejava.git
```

#### 定義用Javaクラスによる個別指定

資産毎に制御をしたい場合は `@EnumResourceLocale` で指定してください.

クラスファイル単位での指定になりますので 更に`PropertyFile`毎に指定をしたい場合は 新たに定義用Javaクラスファイルを作成してください.

```java
@EnumResourceLocale
final Locale locale() {
    return Locale.ROOT;
}
```

### Control（`fallback`）設定


`fallback`を指定すれば、指定したLocaleの`PropertyFile`が存在しない場合に、デフォルトロケールではなく、ルートロケール（ロケール指定なしの`PropertyFile`）を参照させることができます.


#### Code

```
https://bitbucket.org/vermeer_etc/resource-enum-user-control.git
```

#### 定義用Javaクラスの説明

記述するクラスは完全修飾名で記述してください.

```java
@EnumResourceControl
final Control control() {
    return java.util.ResourceBundle.Control.getNoFallbackControl(java.util.ResourceBundle.Control.FORMAT_DEFAULT);
}
```

#### 生成クラスの動作確認

存在しないロケールを指定した場合、デフォルトロケールではなく、ルートロケールが選択されていることが確認できます.

```java
@Test
public void デフォルトロケールを参照() {
    Assert.assertThat(Message.MSG001.toString(), is("メッセージ００１"));
}

@Test
public void 存在しないロケールを指定した場合はルートロケール() {
    Assert.assertThat(Message.MSG001.format(Locale.ITALIAN), is("ルートメッセージ００１"));
}
```


### Control（独自）設定

独自に実装した`Contorl`クラスも指定できます.

Link:[resource-bundle](https://github.com/vermeerlab/maven/tree/mvn-repo/org/vermeerlab/resource-bundle)

この例では `UTF-8`の`PropertyFile`を指定しています.

pom.xmlに依存を追記

```xml
<dependency>
    <groupId>org.vermeerlab</groupId>
    <artifactId>resource-bundle</artifactId>
    <version>0.1.0</version>
</dependency>
```

##### 定義用Javaクラスの説明

動的実行を行うのでクラスは完全修飾名で記述します.

```java
@EnumResourceControl
final Control control() {
    return org.vermeerlab.resourcebundle.CustomControl.builder()
            .charCode(java.nio.charset.StandardCharsets.UTF_8.toString())
            .build();
}
```


#### Localeを環境にあわせて変更

リソース参照時に実行時例外が発生した場合に出力するメッセージ（Enumクラスで保持している静的な値）の指定をしたい場合（＝プロダクト環境で生成するEnum列挙子のコメントを デフォルトロケールではなく 特定のロケールの指定したい場合）の設定です.

定義用Xmlで設定している要素であれば Locale以外の設定も 同じやり方で 切り替えができます.

##### Code

```
https://bitbucket.org/vermeer_etc/resource-enum-user-switchxml.git
```

##### 定義用Xmlファイルの説明

###### pom.xml

```xml
<profiles>
    <profile>
        <id>development</id>
        <properties>
            <processorcommand.classpath>/processor-command.xml</processorcommand.classpath>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>production</id>
        <properties>
            <processorcommand.classpath>/processor-command-production.xml</processorcommand.classpath>
        </properties>
    </profile>
</profiles>
```

###### 開発環境（デフォルト）

`<id>development</id>`

個別設定は全くしていません.


* 定義用Xmlファイル

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root></root>
```

* 生成されたクラス

```java
/**
* メッセージ００１
* <p>
* parameter count = 0
*/
MSG001("msg001", 0, "メッセージ００１"),
```


###### 開発環境（本番用）

`<id>production</id>`

ルートロケールを指定しています.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <propertyFileEnum>
        <withCodeCommentDefaultLocale>false</withCodeCommentDefaultLocale>
        <locale>
            <!-- ja, en ...-->
            <language></language>
            <!-- JP, US ...-->
            <country></country>
        </locale>
    </propertyFileEnum>
</root>
```
`<withCodeCommentDefaultLocale>false` とすることで、作成されるクラスの`value`にも反映します.

この設定をしないと `ResourceBundle`参照時に実行時例外が発生した場合に出力した値が 意図したロケールの値でなくなるためです
（ただし 今回の場合だと実行時例外で出力されるのは `key`のみなので影響はありません）.

* 生成されたクラス

```java
/**
* ルートメッセージ００１
* <p>
* parameter count = 0
*/
MSG001("msg001", 0, "ルートメッセージ００１"),
```


---
### Code

[BitBucket](https://bitbucket.org/vermeerlab/apt-propertyfile-enum)

---
## Version

* 0.1.0
初回リリース

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)

