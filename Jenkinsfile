pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/manishk1001/CICD_Kubernetes.git'
            }
        }

        stage('code review') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube-token'
                }
            }
        }

        stage('build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('build docker image') {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'pass')]) {
                    sh '''
                     docker build -t manish9973/app-1:${BUILD_NUMBER} .
                     docker login -u manish9973 -p $pass
                     docker image push manish9973/app-1:${BUILD_NUMBER}
                     docker rmi manish9973/app-1:${BUILD_NUMBER}
                     '''
                }
            }
        }

        stage('identifying misconfigs using datree') {
            steps {
                dir('kubernetes/') {
                    withEnv(['DATREE_TOKEN=b089272a-bd5b-4724-907a-526083824e8b']) {
                        sh 'helm datree test myapp/'
                    }
                }
            }
        }

        stage('manual approval') {
            steps {
                script {
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: 'k.manish9973@gmail.com'
                        input(id: 'Deploy Gate', message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }l̥
            }
        }

        stage('Deploying application on k8s cluster') {
            steps {
                script {
                    withKubeConfig(credentialsId: 'kubeconfig', restrictKubeConfigAccess: false) {
                        dir('kubernetes/') {
                            sh 'helm upgrade --install --set image.repository="manish9973/app-1" --set image.tag="${VERSION}" myjavaapp myapp/ '
                        }
                    }
                }
            }
        }

        stage('verifying app deployment') {
            steps {
                script {
                    withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myapp-service:8080'
                    }
                }
            }
        }
    }

    post {
        always {
            mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: 'k.manish9973@gmail.com'
        }
    }
}
