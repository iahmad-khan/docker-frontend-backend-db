pipeline {
    agent {
        label 'test' // Replace with the label of your Jenkins agent
    }
    

    environment {
        DOCKER_REGISTRY = "container-registry.registry.svc.local:3200"
        //SONARQUBE_SCANNER_HOME = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo.git' // Replace with your repo URL
            }
        }

        stage('SAST') {
            parallel {
                stage('NPM aduit - Frontend') {
                    steps {
                        script {
                            
                                //sh "pushd frontend && npm audit && popd"
                            
                        }
                    }
                }
                stage('npm audit - Backend') {
                    steps {
                        script {
                            
                                //sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=backend -Dsonar.sources=backend"
                            
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
                            sh 'docker build -t ${DOCKER_REGISTRY}/frontend:latest frontend'
                        }
                    }
                }
                stage('Build Backend') {
                    steps {
                        script {
                            sh 'docker build -t ${DOCKER_REGISTRY}/backend:latest backend'
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
                        echo 'Running Helm Deploy for Frontend'
                        script {

                            //sh 'helm upgrade --install frontend ./charts/frontend --set image.repository=${DOCKER_REGISTRY}/frontend --set image.tag=latest'
                        }
                    }
                }
                stage('Deploy Backend') {
                    steps {
                        echo 'running Helm deploy for Backend'
                        script {
                            //sh 'helm upgrade --install backend ./charts/backend --set image.repository=${DOCKER_REGISTRY}/backend --set image.tag=latest'
                        }
                    }
                }
            }
        }

        stage('Run DAST with OWASP ZAP') {
            steps {
                echo 'Running DAST with OWASP ZAP'
                script {
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
                echo 'Running Load Test'
                script {
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
