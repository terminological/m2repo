# Github Maven repository 

This repository is the distribution point for maven packages from terminological.

Repository settings in consumer pom.xml

```XML
<repositories>
  <repository>
    <id>terminological</id>
    <url>https://maven.pkg.github.com/terminological/m2repo</url>
  </repository>
  ...
</repositories>
```

pom.xml for projects being deployed in this repository

```XML

```


```
mvn deploy
```



```
mvn -B release:clean release:prepare release:perform
```

```XML
<settings>
  <servers>
    <server>
      <id>github</id>
      <username> # GITHUB USERNAME # </username>
      <password> # GITHUB CLASSIC TOKEN # </password>
    </server>
    <server>
        <id>central</id>
        <username> # MAVEN CENTRAL TOKEN USERNAME # /username>
        <password> # MAVEN CENTRAL TOKEN PASSWORD # </password>
    </server>
  </servers>

  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg2</gpg.executable>
        <gpg.keyname> # GPG PASSPHRASE # </gpg.keyname>
      </properties>
    </profile>
  </profiles>

</settings>



```
