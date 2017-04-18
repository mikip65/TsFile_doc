## Installation

Before started, maven should be installed. See <a href="https://maven.apache.org/install.html">How to install maven</a>

There are two ways to use TsFile in your own project.

* Using as jars:
	* Compile the source codes and build to jars
	
		```
		git clone https://github.com/thulab/tsfile.git
		cd tsfile/
		sh package.sh
		```
		Then, all the jars can be get in folder named `lib/`. Import `lib/*.jar` to your project.
	
* Using as a maven dependency: 

        Comiple source codes and deploy to your local repository in three steps:

	* Get the source codes
	
		```
		git clone https://github.com/thulab/tsfile.git
		```
	* Compile the source codes and deploy 
		
		```
		cd tsfile/
		mvn clean install -Dmaven.test.skip=true
		```
	* add dependencies into your project:
	
	  ```
		<dependency>
		   <groupId>com.corp.tsfile</groupId>
		   <artifactId>tsfile</artifactId>
		   <version>0.1.0</version>
		</dependency>
	  ```