# terminological snapshots repository and parent pom.xml

[![Maven Central Version](https://img.shields.io/maven-central/v/io.github.terminological/m2repo)](https://central.sonatype.com/artifact/io.github.terminological/m2repo)

This repository is the distribution point for snapshot Maven packages from 
terminological. It is also the home of the parent pom.xml for projects deployed 
in the terminological namespace. Please feel free to fork and adapt.

## The Maven repository

When using this as a Maven snapshot repository use settings in consumer pom.xml

```XML
<repositories>
	<repository>
		<id>terminological</id>
		<url>https://maven.pkg.github.com/terminological/m2repo</url>
	</repository>
	...
</repositories>
```

## Deployment to the terminological Maven repository

A minimal pom.xml for projects being deployed into this repository follows. The
parent pom.xml here is configured with 2 profiles, the default to deploy direct
to this repository as a github package. The second to deploy using the Maven
release plugin to Maven Central using the [sonatype central portal](https://central.sonatype.com/). 

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<artifactId> ... </artifactId>
	<packaging>jar</packaging>
	<version> ... </version>
	<name> ... </name>
	<description> ... </description>
	
	<parent>
		<groupId>io.github.terminological</groupId>
		<artifactId>m2repo</artifactId>
		<version>0.0.3</version>
	</parent>

	<properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<git.repo>terminological/%{project.artifactId}</git.repo>
	</properties>
	
	<!-- IF NEEDED TO OVERRIDE DEFAULTS:
	<developers>
		<developer>
			<name>...</name>
			<email>...</email>
			<organization>...</organization>
			<organizationUrl>...</organizationUrl>
		</developer>
	</developers>
	-->

	<url>https://github.com/${git.repo}</url>

	<scm>
		<connection>scm:git:git@github.com:${git.repo}.git</connection>
		<developerConnection>scm:git:git@github.com:${git.repo}.git</developerConnection>
		<url>https://github.com/${git.repo}</url>
		<tag>HEAD</tag>
	</scm>
	
	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.7.0</version>
				<configuration>
					<source>${maven.compiler.source}</source>
					<target>${maven.compiler.target}</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

Before this works you must make sure that the following config is in your 
`~/.m2/settings.xml` file:

- Credentials for github, generated as github personal access token (classic), 
[described here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) 
and [here](https://github.com/settings/tokens) ticking the boxes for 
`read:packages`, `write:packages`, `delete:packages` and `repo`.
- Maven central portal credentials are set up in the [Sonatype central portal](https://central.sonatype.com/account).
- A GPG signing key set up is working [see here](https://central.sonatype.org/publish/requirements/gpg/). 

The following is a minimal example that may work but this depends on your gpg setup.

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

With this configuration deploying a snapshot version as a github Maven package:

```
mvn deploy
```

To deploying a release version to central, and automatically publish it.

```
mvn release:clean release:prepare release:perform
# or non-interactive:
# mvn -B release:clean release:prepare release:perform
```

