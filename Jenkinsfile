pipeline {
    agent {
        label 'test' // Replace with the label of your Jenkins agent
    }
    

    environment {
        DOCKER_REGISTRY = "localhost:32000"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/iahmad-khan/docker-frontend-backend-db.git'

            }
        }

        stage('Get Git Commit SHA') {
            steps {
                script {

                    env.GIT_COMMIT_SH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
              
                }
            }
        }

        stage('SAST-NPM Audit') {
            parallel {
                stage('NPM aduit - Frontend') {
                    steps {
                        script {
                            echo 'Running npm audit on frontend'
                            sh '''#!/bin/bash
                            pushd frontend
                            npm audit || true
                            popd
                            '''
                            
                            
                            
                        }
                    }
                }
                stage('NPM audit - Backend') {
                    steps {
                        script {
                            
                            echo "Running npm audit on backend" 
                            sh '''#!/bin/bash
                            pushd backend
                            npm audit || true
                            popd
                            '''
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
                            docker build -t ${DOCKER_REGISTRY}/frontend:${GIT_COMMIT_SH} frontend
                            
                            '''
                        }
                    }
                }
                stage('Build Backend') {
                    steps {
                        script {
                            sh '''
                            rm -rf backend/package-lock.json
                            docker build -t ${DOCKER_REGISTRY}/backend:${GIT_COMMIT_SH} backend
                            
                            '''
                        }
                    }
                }
            }
        }

        stage('SAST - Docker Image Scanning CVEs') {
            parallel {
                stage('Scan Frontend Image using Trivy') {
                    steps {
                        script {
                            sh ''' 
                             trivy image ${DOCKER_REGISTRY}/frontend:${GIT_COMMIT_SH} || true
                            
                            '''
                        }
                    }
                }
                stage('Scan Backend Image using Trivy') {
                    steps {
                        script {
                            sh '''
                            trivy image ${DOCKER_REGISTRY}/backend:${GIT_COMMIT_SH} || true
                            
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
                            sh 'docker push ${DOCKER_REGISTRY}/frontend:${GIT_COMMIT_SH}'
                        }
                    }
                }
                stage('Push Backend') {
                    steps {
                        script {
                            sh 'docker push ${DOCKER_REGISTRY}/backend:${GIT_COMMIT_SH}'
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes Dev') {
           steps {
                script {
                    echo 'Running Helm Deployments'
                    sh '''
                      sed -i "s/latest/${GIT_COMMIT_SH}/" charts/backend/environments/dev/values.yaml
                      sed -i "s/latest/${GIT_COMMIT_SH}/" charts/frontend/environments/dev/values.yaml

                      helm upgrade --install backend -f charts/backend/environments/dev/values.yaml charts/backend/
                      helm upgrade --install frontend -f charts/frontend/environments/dev/values.yaml charts/frontend/
                      sleep 10
                      helm status backend
                      helm status frontend
                      kubectl rollout status deploy backend-backend
                      kubectl rollout status deploy frontend-frontend
                      kubectl get pods -A

                    '''

                 }
            }
        }
                
        stage('Run DAST with OWASP ZAP') {
            steps {
                script {
                   echo 'Running DAST with OWASP ZAP'

                  sh '''
                     FRONTEND_PORT=$(kubectl get svc frontend-frontend -o jsonpath='{.spec.ports[?(@.nodePort)].nodePort}')
                     docker run --net=host --name owasp-zap -dt zaproxy/zap-stable /bin/bash
                     docker exec owasp-zap  mkdir /zap/wrk
                     docker exec owasp-zap zap-baseline.py -t http://localhost:$FRONTEND_PORT -r freport.html -I 
                     docker cp owasp-zap:/zap/wrk/freport.html ${WORKSPACE}/freport.html

                  '''

                }
            }
        }

        stage('API Load Testing') {
            steps {
                script {
                    echo 'Running Load Test'
                    sh '''

                    API_PORT=$(kubectl get svc backend-backend -o jsonpath='{.spec.ports[?(@.nodePort)].nodePort}')
                
                    hey -z 1m -q 100 -c 100 -t 1 http://localhost:$API_PORT/api/todos

                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            script {
                sh 'docker kill owasp-zap && docker rm owasp-zap'
          }
        }
    }
}
