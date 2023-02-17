pipeline {
	agent any
	environment {
		/* permet de créer des varialbes d'environnement pouvant être utilisé dans les stages */
		SONARQUBE_HOST='http://localhost'
		SONARQUBE_PORT='9000'
		
		// This can be nexus3 or nexus2
        	NEXUS_VERSION = 'nexus3'
        	// This can be http or https
        	NEXUS_PROTOCOL = 'http'
        	// Where your Nexus is running. In my case:
        	NEXUS_URL = 'localhost:8081'
        	// Repository where we will upload the artifact
        	NEXUS_REPOSITORY = 'maven-snapshots'
        	// Jenkins credential id to authenticate to Nexus OSS
        	NEXUS_CREDENTIAL_ID = 'NEXUS_USER_CREDENTIALS'
	}
	stages {
		/* etape de récupération du code. le projet GIT est défini dans la configu du job jenkins */
		stage('SCM') {
			steps {
				checkout scm
			}
		}
		/* etape de compilation du code */
		stage('build') {
			agent {
				docker {
					image 'maven:3.6.0-jdk-8-alpine'
					reuseNode true
				}
			}
			steps {
				sh 'mvn clean compile'
			}
		}
		/* etape d'execution des tests unitaires */
		stage('test') {
			steps {
				sh 'mvn test'
			}
			post {
				always {
					junit 'target/surefire-reports/*.xml'
				}
			}
		}
		/* etape d'execution du checkstyle */
		stage('checkstyle') { 
            		steps {
                		sh 'mvn checkstyle:checkstyle' 
            		}
            		post {
                		always {
                    			recordIssues enabledForFailure: true, tool: checkStyle()
                		}
	            	}
        	}
		/* etape d'execution de l'analyse sonar */
		stage('sonar') {
			/* permet de lancer le stage uniquement sur certaines branches. ici master et sonar */
			when {
				anyOf {branch 'sonar'}
			}
			steps {
				withCredentials([string(credentialsId: 'TOKEN_SONAR', variable: 'TOKEN_SONAR')]) {
					sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=mon-appli -Dsonar.host.url=$SONARQUBE_HOST:$SONARQUBE_PORT -Dsonar.login=$TOKEN_SONAR'
				}
			}
		}
		/* lance findbugs */
		stage('findbugs') {
			steps {
           			sh 'mvn findbugs:findbugs' 
       			}
       			post {
           			always {
               				recordIssues enabledForFailure: true, tool: spotBugs(pattern: '**/target/findbugsXml.xml')
           			}
	     		}
		}
		/* deploiement dans nexus */
		stage('Deploy Artifact To Nexus') {
            steps {
                script {
                    pom = readMavenPom file: 'pom.xml'
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
			echo "target/*.${pom.packaging}"
			echo "${filesByGlob}"
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifact
			Exists = fileExists artifactPath
                    if (artifactExists) {
                        nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging
                            ],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: 'pom.xml',
                            type: 'pom'
                            ]
                        ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
	}
}
