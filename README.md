# maven
vermeerlab maven repository library
# Usage
edit pom

```xml

    <properties>
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
            <artifactId>vermeerlab-base</artifactId> <!-- target artifactId is folder name -->
            <version>0.1.1</version> <!-- target version -->
        </dependency>
    </dependencies>

```
