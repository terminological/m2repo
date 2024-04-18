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

pom.xml for projects being deployed using this repository configuration

```XML
    ...
	<parent>
		<groupId>io.github.terminological</groupId>
		<artifactId>m2repo</artifactId>
		<version>0.0.2</version>
	</parent>
    ...
    <properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<git.repo>...organisation.../...repository...</git.repo>
	</properties>
	...
```
Deploying a snapshot version as a github maven package.

```
mvn deploy
```

Deploying a release version to maven central, and automatically deploy it.

```
mvn -B release:clean release:prepare release:perform
```


Credentials for github must be generated as github personal access token (classic)

Maven central portal credentials similarly are given when you set up a token in mavel central portal.

GPG signing key set up is also required. The following is a minimal example that may work but this depends on your setup.

```XML
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>...GITHUB USERNAME...</username>
      <password>...GITHUB CLASSIC TOKEN...</password>
    </server>
    <server>
        <id>central</id>
        <username>...MAVEN CENTRAL TOKEN USERNAME.../username>
        <password>...MAVEN CENTRAL TOKEN PASSWORD...</password>
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
        <gpg.keyname>...GPG KEY...</gpg.keyname>
      </properties>
    </profile>
  </profiles>

</settings>



```
