# Customize the living application theme

::: info
**Note:** For Enterprise, Performance, Efficiency, and Teamwork editions only.
:::

This page explains how to create a theme project in the Studio.

A theme is an archive of files used to define and share CSS definitions and images for an application.
It enables you to specify the same style for all pages and layout of an application.
Each application can have its own theme.

## New Theme...

When you create a new theme (Right click in Project Explorer > New > Theme...), you start with a default project layout based on the Bonita theme.

```
myCustomTheme
|__ dist //Contains all resources to package
|	|__ fonts
|	|__ icons
|	|__ images
|	| theme.css //The css file built from scss sources and optimized, do not modify this file directly
|__ node //Node and npm binaries retrieved after first build
|__ node_modules //Npm modules dependencies retrieved after first build
|__ src
|	|__ scss //Contains all your scss source files
|   		| _bonita_pager.scss
|		| _bonita_variables.scss //Contains all the overridden variables for the Bonita theme
|		| main.scss //Main entry point
|__ target //Maven output folder, contains the theme archive after a build
|__ test
|	|__ css
|	|__ js
|	| README.md
|	| index.html //A static test page to preview your theme
| content.xml //Maven assembly descriptor
| package.json //NPM descriptor
| page.properties //Custom page descriptor
| pom.xml //Maven project descriptor
```

This project is using [Sass](https://sass-lang.com/) to easily create and customize a bootstrap *3.3.7* theme.
To compile the scss it relies on [Node](https://nodejs.org/en/) and [NPM](https://www.npmjs.com/). See the package.json file below for more detailed.

`package.json (NPM descriptor)`
```json
{
  "name": "myCustomTheme", 
  "version": "0.0.1",
  "description": "My custom theme description",
  "license": "GPL-2.0-or-later",
  "scripts": {
    "build": "node-sass --precision 8 --output-style compressed --omit-source-map-url true --include-path ./node_modules/bootstrap-sass/assets/stylesheets/ src/scss/main.scss target/theme.noprefix.css && postcss target/theme.noprefix.css --no-map --use autoprefixer -b \"last 2 versions\" -o dist/theme.css"
  },
  "devDependencies": {
    "node-sass": "4.11.0",
    "postcss-cli": "6.1.2",
    "autoprefixer": "9.5.0",
    "bootstrap-sass": "3.3.7" //Supported version of bootstrap in Bonita
  }
}
```

By default, a _build_ npm script is defined. It runs `node-sass` to compile the `src/scss/main.scss` file. The build command includes the bootstrap-sass stylesheets in order to have clean `@import` statements in the _*.scss_ files.
In addition, postcss-cli and autoprefixer are used to add vendor prefixes for a better browser compatibility.

The maven descriptor is responsible to run the npm build and package the result as a Theme custom page archive. See the pom.xml file below for more detailed.
`pom.xml`
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.company.theme</groupId>
	<artifactId>myCustomTheme</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>pom</packaging>

	<name>My custom theme</name>
	<description>My custom theme description</description>

	<properties>
		<node.version>v10.15.3</node.version>
		<npm.version>6.9.0</npm.version>
	</properties>

	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>com.github.eirslett</groupId>
					<artifactId>frontend-maven-plugin</artifactId>
					<version>1.7.5</version>
					<configuration>
						<installDirectory>${session.executionRootDirectory}</installDirectory>
						<nodeVersion>${node.version}</nodeVersion>
						<npmVersion>${npm.version}</npmVersion>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<plugins>
			<plugin>
				<groupId>com.github.eirslett</groupId>
				<artifactId>frontend-maven-plugin</artifactId>
				<executions>
					<execution>
						<id>install node and npm</id>
						<goals>
							<goal>install-node-and-npm</goal>
							<goal>npm</goal>
						</goals>
					</execution>
					<execution>
						<id>npm build</id>
						<goals>
							<goal>npm</goal>
						</goals>
						<phase>prepare-package</phase>
						<configuration>
							<arguments>run build</arguments>
						</configuration>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<executions>
					<execution>
						<id>page-content</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
						<inherited>false</inherited>
						<configuration>
							<ignoreDirFormatExtensions>true</ignoreDirFormatExtensions>
							<appendAssemblyId>false</appendAssemblyId>
							<descriptors>
								<descriptor>content.xml</descriptor>
							</descriptors>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

The `page.properties` file contains the metadata for the theme (the name used in the URL, the display name, and a description). For example: 
```properties
displayName=My custom theme
name=custompage_myCustomTheme
description=My custom theme description
contentType=theme
```
