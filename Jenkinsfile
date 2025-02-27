pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/iamsaurav-karki/Multi-Tier-With-Database.git'
            }
        }
        stage('Compile') {
            steps {
              sh "mvn compile"
            }
        }
        
        stage('Unit Test cases') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        
        stage('Trivy File Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        
        stage('Sonar Qube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                     -Dsonar.projectKey=multitier \
                     -Dsonar.projectName=multitier \
                     -Dsonar.java.binaries=target \
                     -Dsonar.sources=.
                    '''
            }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
               sh "mvn deploy -DskipTests=true"
            }
            }
        }
        
        stage('Docker Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh "docker build -t sauravkarki/bakapp:latest ."
                 }
                }
            }
        }
        
         stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html sauravkarki/bakapp:latest"
            }
        }
        
         stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh "docker push sauravkarki/bakapp:latest"
                 }
                }
            }
        }
        
         stage('Deploy to Kubernetes') {
            steps {
                steps {
                     withKubeConfig(caCertificate: '', clusterName: 'saurav-cluster', contextName: ' ', credentialsId: 'k8s-login' , namespace: 'webapps', restrictKubeConfigAccess:false, serverUrl:<clusterapiurl>') {
                    sh "kubectl apply -f Manifests/ds.yml -n webapps"
                
            }
                }
            }
        }
        
         stage('Verify Deployment') {
            steps {
                steps {
                     withKubeConfig(caCertificate: '', clusterName: 'saurav-cluster', contextName: ' ', credentialsId: 'k8s-login' , namespace: 'webapps', restrictKubeConfigAccess:false, serverUrl:<clusterapiurl>') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                
            }
                }
            }
        }
    }
}
