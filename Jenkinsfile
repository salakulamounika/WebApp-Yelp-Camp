pipeline {
    agent any
    
    tools {
        nodejs 'node21'
    }
    environment { 
        SCANNER_HOME= tool 'sonar-scanner' 
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/salakulamounika/WebApp-Yelp-Camp'
            }
        }
    
    
        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }
    
    
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
    
    
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
    
    
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
            
        }
    
    
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t salakulamounika/camp1:latest ."
                    }
                }
            }
        }
    
    
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html salakulamounika/camp1:latest"
            }
        }
    
    
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push salakulamounika/camp1:latest"
                    }
                }
                
            }
        }
    
   
        stage('Deploy To EKS cluster') {
            steps {
                
                   withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks-5', contextName: '', credentialsId: 'kube-cred', namespace: 'webapps', serverUrl: 'https://33710B57D81DA597604EBC2D905CF991.gr7.ap-south-1.eks.amazonaws.com']]) {
                        sh "kubectl apply -f Manifests/"
                        sleep 60
                    } 
                
            }
        }
        stage('Verify the deployment') {
            steps {
                
                   withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks-5', contextName: '', credentialsId: 'kube-cred', namespace: 'webapps', serverUrl: 'https://33710B57D81DA597604EBC2D905CF991.gr7.ap-south-1.eks.amazonaws.com']]) {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get pods -n webapps"
                        
                    } 
                
            }
        }
    
    
    }
}
