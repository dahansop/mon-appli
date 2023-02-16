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
					image 'mave:3.6.9-jdk-8-alpine'
					reuseNode true
				}
			}
			steps {
				sh 'mnv clean compile'
			}
		}
	}
}
