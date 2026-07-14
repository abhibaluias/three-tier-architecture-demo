pipeline {
    agent none 

    environment {
        // Docker Hub Credentials & Registry Constants
        DOCKER_HUB_USER  = "your-dockerhub-username" // 👈 Replace with your actual Docker Hub ID
        DOCKER_CREDS_ID  = "dockerhub-credentials-id" // Managed via Jenkins Credentials Store
        
        // AWS Settings for Kubernetes Deployment Target
        AWS_CREDENTIALS  = "aws-credentials-id" // Managed via Jenkins Credentials Store
        AWS_REGION       = "us-east-1"
        EKS_CLUSTER_NAME = "robot-shop-cluster"
        
        // Quality Scan Configurations
        SONAR_SCANNER    = "SonarScanner"
    }

    stages {
        stage('Security Analysis & Microservices Build') {
            parallel {
                
                // --- 1. NODE.JS CORE SERVICE (Cart & Catalogue) ---
                stage('Cart Service') {
                    agent { docker { image 'node:18-alpine' } }
                    stages {
                        stage('SAST Cart') {
                            steps {
                                dir('cart') {
                                    withSonarQubeEnv('SonarQube-Server') {
                                        sh "sonar-scanner -Dsonar.projectKey=rs-cart -Dsonar.sources=."
                                    }
                                }
                            }
                        }
                        stage('Build & Push Cart') {
                            agent { agent any }
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('cart') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-cart:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-cart:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-cart:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-cart:latest"
                                    }
                                }
                            }
                        }
                    }
                }

                // --- 2. PYTHON SERVICE (Payment processing Engine) ---
                stage('Payment Service') {
                    agent { docker { image 'python:3.11-slim' } }
                    stages {
                        stage('SAST Payment') {
                            steps {
                                dir('payment') {
                                    withSonarQubeEnv('SonarQube-Server') {
                                        sh "sonar-scanner -Dsonar.projectKey=rs-payment -Dsonar.sources=."
                                    }
                                }
                            }
                        }
                        stage('Build & Push Payment') {
                            agent { agent any }
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('payment') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-payment:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-payment:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-payment:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-payment:latest"
                                    }
                                }
                            }
                        }
                    }
                }

                // --- 3. GOLANG RUNTIME SERVICE (Dispatch / Logistics) ---
                stage('Dispatch Service') {
                    agent { docker { image 'golang:1.21-alpine' } }
                    stages {
                        stage('SAST Dispatch') {
                            steps {
                                dir('dispatch') {
                                    withSonarQubeEnv('SonarQube-Server') {
                                        sh "sonar-scanner -Dsonar.projectKey=rs-dispatch -Dsonar.sources=."
                                    }
                                }
                            }
                        }
                        stage('Build & Push Dispatch') {
                            agent { agent any }
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('dispatch') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-dispatch:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-dispatch:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-dispatch:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-dispatch:latest"
                                    }
                                }
                            }
                        }
                    }
                }

                // --- 4. JAVA SPRING BOOT PIPELINE (Shipping Logistics Management) ---
                stage('Shipping Service') {
                    agent { docker { image 'maven:3.9-eclipse-temurin-17' } }
                    stages {
                        stage('SAST Shipping') {
                            steps {
                                dir('shipping') {
                                    withSonarQubeEnv('SonarQube-Server') {
                                        sh "mvn clean install sonar:sonar -Dsonar.projectKey=rs-shipping"
                                    }
                                }
                            }
                        }
                        stage('Build & Push Shipping') {
                            agent { agent any }
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('shipping') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-shipping:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-shipping:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-shipping:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-shipping:latest"
                                    }
                                }
                            }
                        }
                    }
                }

                // --- 5. PHP APACHE APP LAYER (Ratings Integration) ---
                stage('Ratings Service') {
                    agent { docker { image 'php:8.2-cli' } }
                    stages {
                        stage('SAST Ratings') {
                            steps {
                                dir('ratings') {
                                    withSonarQubeEnv('SonarQube-Server') {
                                        sh "sonar-scanner -Dsonar.projectKey=rs-ratings -Dsonar.sources=."
                                    }
                                }
                            }
                        }
                        stage('Build & Push Ratings') {
                            agent { agent any }
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('ratings') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-ratings:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-ratings:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-ratings:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-ratings:latest"
                                    }
                                }
                            }
                        }
                    }
                }

                // --- 6. NGINX FRONTEND WEB PROXY (User Gateway Presentation Tier) ---
                stage('Web Frontend Service') {
                    agent { agent any }
                    stages {
                        stage('Build & Push Web') {
                            steps {
                                withDockerRegistry([credentialsId: "${DOCKER_CREDS_ID}", url: ""]) {
                                    dir('web') {
                                        sh "docker build -t ${DOCKER_HUB_USER}/rs-web:${BUILD_NUMBER} -t ${DOCKER_HUB_USER}/rs-web:latest ."
                                        sh "docker push ${DOCKER_HUB_USER}/rs-web:${BUILD_NUMBER}"
                                        sh "docker push ${DOCKER_HUB_USER}/rs-web:latest"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Quality Gate Enforcement') {
            agent { agent any }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    // Block subsequent stages if Quality Gate standards drop below acceptable limits
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Amazon EKS Cluster') {
            agent { agent any }
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"
                    
                    // Maps straight into the repo's Helm deployment directory
                    dir('EKS') {
                        sh "helm upgrade --install robot-shop ./helm \
                            --set image.repo=${DOCKER_HUB_USER} \
                            --set image.version=${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('OWASP ZAP Scan (DAST)') {
            agent { agent any }
            steps {
                script {
                    // Pull web gateway ingress hostname from EKS dynamically inside a scripted block
                    def targetUrl = sh(script: "kubectl get svc web -o 
                    jsonpath='{.status.loadBalancer.ingress.hostname}'", returnStdout: true).trim()
                    sh "docker run --rm -v ${WORKSPACE}:/zap/wrk/:rw owasp/zap2docker-stable zap-baseline.py -t http://${targetUrl} -r zap_report.html || true"
                }
            }
        }
    }
    post {
        always {
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'zap_report.html',
                reportName: 'OWASP ZAP DAST Report'
            ])
            cleanWs()
        }
    }
}