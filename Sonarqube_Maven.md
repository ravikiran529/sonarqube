1. Setup the Sonarqube server and reset the password using url http://localhost:9000 
2. Configure a webhook URL in sonarqube server 
```https://jenkins_url:8080/sonarqube-webhook```
3. Open your project pom.xml and add the below content

```
<dependencies>
		        <dependency>
		            <groupId>junit</groupId>
		            <artifactId>junit</artifactId>
		            <version>4.12</version>
		            <scope>test</scope>
		        </dependency>
		</dependencies>
		
		<build>
		        <pluginManagement>
		            <plugins>
		                <plugin>
		                    <groupId>org.apache.maven.plugins</groupId>
		                    <artifactId>maven-compiler-plugin</artifactId>
		                    <version>3.8.1</version>
		                </plugin>
		                <plugin>
		                    <groupId>org.sonarsource.scanner.maven</groupId>
		                    <artifactId>sonar-maven-plugin</artifactId>
		                    <version>3.6.0.1398</version>
		                </plugin>
		                <plugin>
		                    <groupId>org.jacoco</groupId>
		                    <artifactId>jacoco-maven-plugin</artifactId>
		                    <version>0.8.4</version>
		                </plugin>
		            </plugins>
		        </pluginManagement>
		</build>
		
		<profiles>
		        <profile>
		            <id>sonar</id>
		            <activation>
		                <activeByDefault>true</activeByDefault>
		            </activation>
		            <properties>
		                <!-- Optional URL to server. Default value is http://localhost:9000 -->
		                <sonar.host.url>
		                    http://localhost:9000
		                </sonar.host.url>
		            </properties>
		        </profile>
		        <profile>
		            <id>coverage</id>
		            <activation>
		                <activeByDefault>true</activeByDefault>
		            </activation>
		            <build>
		                <plugins>
		                    <plugin>
		                        <groupId>org.jacoco</groupId>
		                        <artifactId>jacoco-maven-plugin</artifactId>
		                        <executions>
		                            <execution>
		                                <id>prepare-agent</id>
		                                <goals>
		                                    <goal>prepare-agent</goal>
		                                </goals>
		                            </execution>
		                            <execution>
		                                <id>report</id>
		                                <goals>
		                                    <goal>report</goal>
		                                </goals>
		                            </execution>
		                        </executions>
		                    </plugin>
		                </plugins>
		            </build>
		        </profile>
		</profiles>
    ```
    
    3. Write your pipeline file as below
    
    pipeline {
    agent any
    stages {
        stage('Clone sources') {
            steps {
                git url: 'https://github.com/XXXX/XXXX.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "mvn clean package sonar:sonar"
                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        // --> Deployment Stages here <-- //
      }
    }
    
