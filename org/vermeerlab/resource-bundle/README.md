Extention Resource Bundle
===

拡張`Resource Bundle`です.

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
        <artifactId>resource-bundle</artifactId>
        <version>0.1.0</version> <!-- target version -->
    </dependency>
</dependencies>
```

### パラメーター実装

#### デフォルト指定

文字コード ASCII、取得対象検索順：class・properties・xml、Localeはデフォルトロケールです.
（デフォルトリソースではありません）

デフォルトロケールが存在しない場合は、デフォルトリソース（ロケール記載の無いリソースファイル）を検索します.

```java
CustomControl control = CustomControl.builder().build();
```

#### 文字コードを指定する場合

```java
 CustomControl control = CustomControl.builder().charCode(StandardCharsets.UTF_8.toString()).build();

 CustomControl control = CustomControl.builder().charCode("UTF-8").build();

 CustomControl control = CustomControl.builder().charCode("SJIS").build();
````

#### 参照するリソースの順番を変える場合

デフォルト（class・properties・xml）から properties・xml・classに変更する.

```java
 CustomControl control = CustomControl.builder()
  .formats(CustomControl.FORMAT_DEFAULT)
  .formats(CustomControl.FORMAT_XML)
  .build();
```

#### 参照するリソースを固定する場合

XMLのみの場合

```java
 CustomControl control = CustomControl.builder()
  .formats(CustomControl.FORMAT_XML)
  .build();
```

#### 検索ロケール順を指定する場合

必ず、デフォルトLocale.ROOTを入れておくと、リソース検索エラーになりにくくなることが期待されます.

事例は検索を適用するロケールが日本語（JAPANESE）で、検索順をUS・JAPAN・デフォルトリソースの順とした記述（通常、こういうことはしないが実装例として記載）

```java
 CustomControl control = CustomControl.builder()
 .targetCandidateLocalePair(

  TargetCandidateLocalePair.builder()
      .targetLocale(Locale.JAPANESE)
      .candidateLocale(Locale.US)
      .candidateLocale(Locale.JAPAN)
      .candidateLocale(Locale.ROOT)
      .build()
  .build();
```

#### 検索を適用するロケールを複数指定した場合

```java
 CustomControl control = CustomControl.builder()
 .charCode("UTF-8")
 .targetCandidateLocalePair(

  TargetCandidateLocalePair.builder()
      .targetLocale(Locale.US)
      .candidateLocale(Locale.Locale.ENGLISH)
      .candidateLocale(Locale.ROOT)
      .build()

 .targetCandidateLocalePair(
  TargetCandidateLocalePair.builder()
      .targetLocale(Locale.JAPANESE)
      .candidateLocale(Locale.JAPAN)
      .candidateLocale(Locale.ROOT)
      .build()
  .build();
```

nullの場合は、ResourceBundleで指定したロケールと同じロケールに置き換えます.

```java
 CustomControl control = CustomControl.builder()
 .targetCandidateLocalePair(
  TargetCandidateLocalePair.builder()
      .targetLocale(Locale.JAPANESE)
      .candidateLocale(Locale.US)
      .candidateLocale(null)
      .candidateLocale(Locale.ROOT)
      .build()
  .build();
```

#### データキャッシュを設定する場合

キャッシュする：CustomControl.TTL_NO_EXPIRATION_CONTROL（デフォルト）

```java
 CustomControl control = CustomControl.builder().timeToLive(CustomControl.TTL_NO_EXPIRATION_CONTROL).build();
```

キャッシュしない・都度リソースを読み込む：
```java
CustomControl.TTL_DONT_CACHE
```

```java
 CustomControl control = CustomControl.builder().timeToLive(CustomControl.TTL_DONT_CACHE).build();
```

ResourceBundleの有効期限(0またはキャッシュ格納時刻からの正のミリ秒オフセット)を指定します.

```java
 CustomControl control = CustomControl.builder().timeToLive(1000L).build();
```




全ての設定は組み合わせて使用することが出来ます.
## Code
[BitBucket](https://bitbucket.org/vermeerlab/resource-bundle)


## Version
* 0.1.0
初期リリース

## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)

