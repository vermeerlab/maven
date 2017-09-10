ClassPath配下の資産検索
===

ClassPath配下の資産を検索するクラス群です.  
Jarファイルおよび Jarにアーカイブされているクラスも検索対象です.

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
        <artifactId>vermeerlab-classpath-scanner</artifactId>
        <version>0.2.0</version> <!-- target version -->
    </dependency>
</dependencies>
```
---
### 基本

`xxxFinder#find`の戻り値の型が、`xxxScanner`の型です.  

#### 型ジェネリックを使用した理由
必ずしも戻り値が`Path`であることが最良ではなかったため.  
具体的には、絶対パスのファイル表記を使用するのではなく、取得したファイルをクラスの完全修飾名として編集して、
検索結果を文字列として扱った方が後続処理にて扱い易いユースケースがあったため.

#### ClassPath
デフォルトの検索クラス（`ClassPathFinder`）は、ファイル名に対して「`.class`」の後方一致のみを条件として検索します.

```java
ClassPathScanner<Path> scanner = new ClassPathScanner<>();
```

#### JarPath

クラスパス配下のJarファイル内のファイルを検索します.
デフォルトの検索クラス（`JarPathFinder`）は、ファイル名に対して「`.class`」の後方一致かつ「`$`」をファイルパスに含まないことを条件として検索します.

```java
JarPathScanner<Path> scanner = new JarPathScanner<>();
```
---
### 応用（検索条件の拡張）
※実際に別パッケージにて行った拡張です.  
[パッケージフォルダ](https://bitbucket.org/vermeerlab/apt-core/src/f3cc7e40c5add013f138d60c5ead204464ff4a0a/src/main/java/org/vermeerlab/apt/command/?at=0-3-0)  

#### ClassPath

##### 呼出先の実装

```java

class ProcessorCommandClassFinder<SCAN_RESULT_TYPE> extends ClassPathFinder<SCAN_RESULT_TYPE> {

    TypeAssignableFrom assignableFrom;

    public ProcessorCommandClassFinder() {
        this.assignableFrom = TypeAssignableFrom.of(ProcessorCommandInterface.class);
    }

    /**
     * {@inheritDoc }
     */
    @Override
    @SuppressWarnings("unchecked") //Generic Cast
    public Stream<SCAN_RESULT_TYPE> find(Object rootPath) throws IOException {
        Stream<SCAN_RESULT_TYPE> pathStream = super.find(rootPath)
                .map(filePath -> {
                    String classFilePath
                           = this.assignableFrom.substringClassFilePath(this.rootPath().toString(), filePath.toString());
                    return (SCAN_RESULT_TYPE) classFilePath.substring(0, classFilePath.length() - 6);
                });
        return pathStream;
    }

    /**
     * {@inheritDoc }
     */
    @Override
    protected boolean match(Path path, BasicFileAttributes attrs) {
        if (super.match(path, attrs) == false) {
            return false;
        }
        return assignableFrom.isValidAbsoluteFilePath(this.rootPath().toString(), path.toString());
    }
}

```
###### やっていること  

1. 取得したファイルの絶対パスから、ルートとなるパス文字列と「`.class`」を除外して、クラスの完全修飾名として編集をする
2. 編集したクラス名から`ProcessorCommandInterface.class`を継承したクラスか判定をする

###### ポイント
`SCAN_RESULT_TYPE`のキャスト元が`String`であること

```java
(SCAN_RESULT_TYPE) classFilePath.substring(0, classFilePath.length() - 6);
```

##### 呼出元の実装

`ProcessorCommandScanner#scan`

```java
ClassPathScanner<String> classScanner = new ClassPathScanner<>(new ProcessorCommandClassFinder<>());
```

###### ポイント
戻り値の型が基本では`Path`だったところが、このケースでは`String`.  


#### JarPath

##### 呼出先の実装

```java

class ProcessorCommandJarFinder<SCAN_RESULT_TYPE> extends JarPathFinder<SCAN_RESULT_TYPE> {

    TypeAssignableFrom assignableFrom;

    public ProcessorCommandJarFinder() {
        this.assignableFrom = TypeAssignableFrom.of(ProcessorCommandInterface.class);
    }

    /**
     * {@inheritDoc }
     */
    @Override
    protected boolean fileFilter(JarEntry jarEntry) {
        return super.fileFilter(jarEntry)
               ? this.assignableFrom.isValidClassFileName(jarEntry.getName())
               : false;
    }

    /**
     * {@inheritDoc }
     */
    @Override
    @SuppressWarnings("unchecked") //Generic Cast
    protected SCAN_RESULT_TYPE toResultValue(JarEntry entry) {
        return (SCAN_RESULT_TYPE) entry.getName();
    }
}

```

###### やっていること  

1. クラス名（`entry#getName`）から`ProcessorCommandInterface.class`を継承したクラスか判定をする
2. クラス完全修飾名を返却する


###### ポイント
`SCAN_RESULT_TYPE`のキャスト元が`String`であること

```java
return (SCAN_RESULT_TYPE) entry.getName();
```

##### 呼出元の実装

```java
ProcessorCommandJarPathScanner<String> jarScanner = new ProcessorCommandJarPathScanner<>(
        new ProcessorCommandJarFinder<>(),
        configXml);
```


###### ポイント
戻り値の型が基本では`Path`だったところが、このケースでは`String`. 

---
#### その他
Jarファイル名の検索条件を変更する（`JarPathScanner#jarFilter`）など、`xxxScanner`および`xxxFinder`の各メソッドを拡張が可能です.  
検索、編集のメソッドを`protected`にしているので用途に応じた拡張をしてください.

## Code
[BitBuckt](https://bitbucket.org/vermeerlab/classpath-scanner)


## Version
* 0.2.0  
初期リリース  
（他共通クラス群とあわせたバージョン割り当てにより、0.1.0はスキップ）


## Licence
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Author
[_vermeer_](https://twitter.com/_vermeer_)
