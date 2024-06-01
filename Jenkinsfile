pipeline {
    agent {
        label 'test' // Replace with the label of your Jenkins agent
    }
    

    environment {
        DOCKER_REGISTRY = "container-registry.registry.svc.local:32000"
        //SONARQUBE_SCANNER_HOME = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation' 
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/iahmad-khan/docker-frontend-backend-db.git'
            }
        }

        stage('SAST') {
            parallel {
                stage('NPM aduit - Frontend') {
                    steps {
                        script {
                             echo 'Running audit frontend'
                            
                        }
                    }
                }
                stage('npm audit - Backend') {
                    steps {
                        script {
                            
                                echo "running npm audit backend"                            
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        script {
                            sh ''' 
                            rm -rf frontend/package-lock.json
                            docker build -t ${DOCKER_REGISTRY}/frontend:latest frontend
                            
                            '''
                        }
                    }
                }
                stage('Build Backend') {
                    steps {
                        script {
                            sh '''
                            rm -rf backend/package-lock.json
                            docker build -t ${DOCKER_REGISTRY}/backend:latest backend
                            
                            '''
                        }
                    }
                }
            }
        }
        stage('Docker Image Scanning for CVEs') {
            parallel {
                stage('Scan Frontend Image using Trivy') {
                    steps {
                        script {
                            sh ''' 
                             trivy image ${DOCKER_REGISTRY}/frontend:latest || True
                            
                            '''
                        }
                    }
                }
                stage('Scan Backend Image using Trivy') {
                    steps {
                        script {
                            sh '''
                            trivy image ${DOCKER_REGISTRY}/backend:latest || True
                            
                            '''
                        }
                    }
                }
            }
        }   
        stage('Push Docker Images') {
            parallel {
                stage('Push Frontend') {
                    steps {
                        script {
                            sh 'docker push ${DOCKER_REGISTRY}/frontend:latest'
                        }
                    }
                }
                stage('Push Backend') {
                    steps {
                        script {
                            sh 'docker push ${DOCKER_REGISTRY}/backend:latest'
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            parallel {
                stage('Deploy Frontend') {
                    steps {
                       
                        script {
                            echo 'Running Helm Deploy for Frontend'
                            //sh 'helm upgrade --install frontend ./charts/frontend --set image.repository=${DOCKER_REGISTRY}/frontend --set image.tag=latest'
                        }
                    }
                }
                stage('Deploy Backend') {
                    steps {
                        script {
                            echo 'running Helm deploy for Backend'
                            //sh 'helm upgrade --install backend ./charts/backend --set image.repository=${DOCKER_REGISTRY}/backend --set image.tag=latest'
                        }
                    }
                }
            }
        }

        stage('Run DAST with OWASP ZAP') {
            steps {
                script {
                   echo 'Running DAST with OWASP ZAP'

                   // def targetUrl = "http://your-api-endpoint" // Replace with your actual target URL

                   // zapStart zapHome: '/path/to/ZAP' // Adjust the path to your ZAP installation if necessary

                   // zapAttack target: targetUrl, attackMode: 'quick'

                    //zapReport reportName: 'zap-report.html', reportType: 'HTML'

                    //zapStop()
                }
            }
        }

        stage('API Load Testing') {
            steps {
                script {
                    echo 'Running Load Test'
                    //echo "Load Test using Hey"
                    // Assuming you have a load testing tool installed (e.g., hey)
                    //sh 'hey -z 30s -c 100 http://your-api-endpoint'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
