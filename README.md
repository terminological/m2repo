# terminological snapshots repository and parent pom.xml

[![Maven Central Version](https://img.shields.io/maven-central/v/io.github.terminological/m2repo)](https://central.sonatype.com/artifact/io.github.terminological/m2repo)

This repository is the distribution point for snapshot Maven packages from 
terminological. It is also the home of the parent pom.xml for projects deployed 
in the terminological namespace. Please feel free to fork and adapt.

## Using a library from the terminological Maven repository

When using this as a Maven snapshot repository use settings in consumer pom.xml

```XML
<repositories>
	<repository>
		<id>github</id>
		<url>https://maven.pkg.github.com/terminological/m2repo</url>
	</repository>
	...
</repositories>
```

Github requires you to be identified even to access public repositories. This 
is done in your `~/.m2/settings.xml` file which needs the following 
for credentials for github, generated as github personal access token (classic), 
[described here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) 
and [here](https://github.com/settings/tokens) ticking the boxes for 
`read:packages` (see also below).

```XML
<settings>
	<servers>
		<server>
			<id>github</id>
			<username>...GITHUB USERNAME...</username>
			<password>...GITHUB CLASSIC TOKEN...</password>
		</server>
		...
	</servers>
	...
</settings>
```

## Deployment to the terminological Maven repository

A minimal pom.xml for projects being deployed into this repository follows. The
parent pom.xml here is configured with 2 profiles, the default to deploy direct
to this repository as a github package. The second to deploy using the Maven
release plugin to Maven Central using the 
[sonatype central portal](https://central.sonatype.com/). 

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
		<git.repo>terminological/${project.artifactId}</git.repo>
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
</project>
```

Before this works you must make sure that the following config is in your 
`~/.m2/settings.xml` file:

- Credentials for github, generated as github personal access token (classic), 
[described here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) 
and [here](https://github.com/settings/tokens) ticking the boxes for 
`read:packages`, `write:packages`, `delete:packages` and `repo`.
- Maven central portal credentials are set up in the [Sonatype central portal](https://central.sonatype.com/usertoken).
- A GPG signing key set up is working [see here](https://central.sonatype.org/publish/requirements/gpg/).
  Check it has not expired (`gpg --list-secret-keys --keyid-format=long`) and that your public
  key is still retrievable from a keyserver - central validates both and rejects expired keys.

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
			<username>...MAVEN CENTRAL TOKEN USERNAME...</username>
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

### Getting the gpg passphrase to the build

`maven-gpg-plugin` always invokes gpg with `--batch --no-tty`, so gpg can never prompt you
on the terminal - `mvn -B` is not the cause and dropping it does not help. Either the
*gpg-agent* answers (a GUI pinentry, if the agent can reach your desktop session) or the
signing step dies with `No pinentry` / `Inappropriate ioctl for device`. Note also that
`release:perform` signs inside a forked `mvn` whose stdin is a pipe, so a terminal prompt
could not be answered there even if gpg allowed one. Pick one of:

**1. Let the gpg-agent cache it** - nothing is written to disk, nothing to configure in
Maven. Answer the prompt once from a desktop terminal, then release straight away:

```
echo test | gpg --local-user ...GPG KEY... --armor --sign -o /tmp/t.asc   # prompts once
cd ~/Git/m2repo && mvn -B release:clean release:prepare release:perform
```

The agent caches for 600s by default, usually enough. If it is not, put
`default-cache-ttl 3600` and `max-cache-ttl 3600` in `~/.gnupg/gpg-agent.conf` and run
`gpgconf --reload gpg-agent`. If no dialog appears at all the agent is using a tty
pinentry; set `pinentry-program /usr/bin/pinentry-qt` (or `pinentry-gtk-3`) there instead.

**2. Feed it from your password manager.** Requires the two `pinentry-mode loopback` args
in the `maven-gpg-plugin` config of the `central.profile` above to be uncommented, with
`-!-` changed back to `--` (the dashes are broken up because XML forbids `--` inside a
comment), then:

```
mvn -B -Dgpg.passphrase="$(pass show maven/gpg)" release:clean release:prepare release:perform
```

`pass`, `gopass`, `op read`, `security find-generic-password` and friends all work. The
passphrase is visible in `ps` to other users on a shared machine for the length of the
build; harmless on a laptop.

**3. Encrypt it inside `settings.xml`.** The plugin runs server credentials through Maven's
`SettingsDecrypter`, so `{...}` encrypted values are accepted:

```
mvn --encrypt-master-password   # paste the result into ~/.m2/settings-security.xml
mvn --encrypt-password          # -> the value below
```

```XML
<server>
	<id>...GPG KEY...</id>
	<passphrase>{...ENCRYPTED PASSPHRASE...}</passphrase>
</server>
```

The `<id>` must match `passphraseServerId`, i.e. it is the key name, not `gpg.passphrase`.
Loopback is needed here too. This is obfuscation rather than secrecy - the master password
sits in `~/.m2/settings-security.xml` alongside it - but it does keep the plaintext out of
anything you might paste or commit.

Never enable loopback without also supplying a passphrase: gpg then fails outright rather
than falling back to the agent, which is why the args ship commented out.

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

