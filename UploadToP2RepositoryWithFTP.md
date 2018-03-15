# How to deploy Eclipse feature to P2 repository with FTP

I assume that you have your that your update site project is on and working.

## POM configuration.
In your `pom.xml` inside you repository project put this configuration:

### pom.xml of repository project
```xml
<build>
	<extensions>
		<!-- Enabling the use of FTP -->
		<extension>
			<groupId>org.apache.maven.wagon</groupId>
			<artifactId>wagon-ftp</artifactId>
			<version>1.0</version>
		</extension>
	</extensions>
</build>
<profiles>
	<!-- This profile is used to upload the repository, you can have more that one -->
	<profile>
		<id>uploadRepo</id>
		<properties>
			<!-- Put your FPT server address -->
			<ftp.url>ftp://MY.FTP.SERVER.COM</ftp.url>
			<!-- Where to put your files on this server-->
			<ftp.toDir>/WHERE/TO/PUT/MY/FILES</ftp.toDir>
			<!-- Files that you are deploying -->
			<repo.path>${project.build.directory}/repository/</repo.path>
		</properties>

		<build>
			<plugins>
				<plugin>
					<groupId>org.codehaus.mojo</groupId>
					<artifactId>wagon-maven-plugin</artifactId>
					<version>1.0</version>
					<executions>
						<execution>
							<id>upload-repo</id>
							<phase>install</phase>
							<goals>
								<goal>upload</goal>
							</goals>
							<configuration>
								<fromDir>${repo.path}</fromDir>
								<includes>**</includes>
								<toDir>${ftp.toDir}</toDir>
								<url>${ftp.url}</url>
								<!-- Same ID need to be configured in settings.xml file, check next point. -->
								<serverId>ftp-repository</serverId>
							</configuration>
						</execution>
					</executions>
				</plugin>
			</plugins>
		</build>
	</profile>
</profiles>
```

### Access configuration in settings.xml
You need to update your `settings.xml` file to add informations like user name, password,... for your ftp server configuration, in this example called `ftp-repository`.
I have my file in `~/.m2/settings.xml`

This is example configuration:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                         https://maven.apache.org/xsd/settings-1.0.0.xsd">
     <localRepository/>
     <interactiveMode/>
     <usePluginRegistry/>
     <offline/>
     <pluginGroups/>
     <servers>
   		<server>
   			<!-- ID that you used in your pom.xml file-->
     		<id>ftp-repository</id>
     		<!-- FTP user name-->
     		<username>MY_FTP_USER_NAME</username>
     		<!-- FTP password-->
     		<password>MY_FTP_PASSWORD</password>
   		</server>
	</servers>
     <mirrors/>
     <proxies/>
     <profiles/>
     <activeProfiles/>
</settings>
```

### Upload
Run `mvn install -P ${profileName}`
