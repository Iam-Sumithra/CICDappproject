pipeline {
    agent any
    
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Iam-Sumithra/CICDappproject.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Trivy FS scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('Sonarqube Analysis') {
            steps {
              withSonarQubeEnv('sonar-server') {
              sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
              -Dsonar.java.binaries=target'''
                }  
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Publish Artifact') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk21', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Docker Build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t sumithrab/bloggingappsumithra:v1 ."
                    }   
                }
            }
        }
        
        stage('Trivy image scan') {
            steps {
                sh 'trivy image --format table -o image.html sumithrab/bloggingappsumithra:latest '
            }
        }
        
        stage('Docker push image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push sumithrab/bloggingappsumithra:latest"
                    }   
                }
            }
        }
        
        stage('K8-Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9DCCF397CE921D82FA5AC32D1228D82A.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f deployment-service.yml'
                    sleep 20
                }
            }
        }
        
        stage('K8-verify deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9DCCF397CE921D82FA5AC32D1228D82A.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
    }
}
