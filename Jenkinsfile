pipeline {
    agent any
    
    tools {
    	nodejs 'nodejs16'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MTC0D3/dumbmerch-app.git'
            }
        }
        
         stage('Frontend Compilation') {
            steps {
                dir('frontend') {
                   sh 'find . -name "*.js" -exec node --check {} +'
                }
            }
        }
        
        stage('Gitleaks') {
            steps {
                sh 'gitleaks detect --source ./frontend --exit-code 1'
            }
        }
        
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }


        stage('Build-Tag & Push Frontend Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('client') {
                            sh 'docker build -t mtc0d3/frontend:latest .'
                            sh 'trivy image --format table -o frontend-image-report.html mtc0d3/frontend:latest '
                        }
                    }
                }
            } 
        }  

        stage('Docker Deploy via Compose') {
            steps {
                script {
                    sh 'docker-compose up -d'
                }
            }
        }
        
    }
}
