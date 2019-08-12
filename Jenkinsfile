pipeline {
    agent any
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
		stage('Deploy to Server') {
			steps {
            sh label: '', script: 'mvn dependency:get -DremoteRepositories=http://localhost:8081/repository/maven-snapshots/ -DgroupId=${GROUPID} -DartifactId=${ARTIFACTID} -Dversion=${VERSION} -Dtransitive=false  -Ddest=/app/'
            }
        }
    }
    post { 
        always { 
            cleanWs()
             }
        }
}
