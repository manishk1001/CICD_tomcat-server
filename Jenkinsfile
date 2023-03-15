pipeline{
    agent any
    stages{
        stage('git checkout'){
            steps{
                git 'https://github.com/manishk1001/CICD_tomcat-server.git'
            }
        }

        stage('build'){
           steps{
                bat 'mvn clean package'
           }
        }
        
        stage('code review'){
            steps{
               withSonarQubeEnv('sonarqube') {
                bat 'mvn sonar:sonar'
                  }
            }
        }

        stage('upload artifact'){
            steps{
               nexusArtifactUploader artifacts: [[artifactId: 'onlinebookstore', classifier: '', file: 'target/onlinebookstore.war', type: 'war']], credentialsId: 'nexus-credentials', groupId: 'project', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'cicd', version: '0.0.1-SNAPSHOT'
            }
        }
        
        stage('deploy'){
            steps{
                bat 'copy C:\\jenkinshome\\workspace\\git-mvn\\target\\onlinebookstore.war C:\\tomcat\\webapps'
            }
        }
        
    }
}
