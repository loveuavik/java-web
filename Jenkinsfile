pipeline {
    agent none
	  environment {
		pom = readMavenPom file: 'initial/pom.xml'
		ARTIFACTID = "${pom.artifactId}"
		VERSION = "${pom.version}"
		GROUPID = "${pom.groupId}"
		}
    stages {
        stage('Build') {
            steps {
                sh label: '', script: 'mvn -f initial/pom.xml package'
            }
        }
        stage("SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('sonar') {
                script {
                sh 'mvn -f initial/pom.xml clean package sonar:sonar -Dsonar.host.url=http://${sonarurl}:9000'
              }
             }
            }
          }
          stage("Quality Gate") {
            steps {			  
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage('Deploy to Repo') {
            steps {
                sh label: '', script: 'mvn -f initial/pom.xml clean deploy'
            }
        }
		stage('Invoking Deployment Java Pipeline Job') {
			steps {
				build job: 'deployment-java', parameters: [
						string(name: 'ARTIFACTID', value: env.ARTIFACTID),
						string(name: 'VERSION', value: env.VERSION),
						string(name: 'GROUPID', value: env.GROUPID)
				], wait: true
            }
        }
    }
    post { 
        always { 
            cleanWs()
             }
        }
}
