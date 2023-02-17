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
		stage('sonar') {
			steps {
				sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=mon-appli -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_2562be20f11fed54c2f378a2a1a578820729d0df'
			}
		}
			
	}
}
