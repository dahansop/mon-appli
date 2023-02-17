pipeline {
	agent any
	stages {
		stage('SCM') {
			steps {
				checkout scm
			}
		}
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
				sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=mon-appli -Dsonar.host.url=$SONARQUBE_HOST:$SONARQUBE_PORT -Dsonar.login=$TOKEN_SONAR'
			}
		}
	}
}
