pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'muzuinzu'
        BUILD_TAG = "v${env.BUILD_NUMBER}"
        KUBECONFIG_PATH = "C:\\Users\\HP\\.kube\\config"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/muzuinzu/microservices.git'
                bat 'dir'
            }
        }

        stage('SonarQube Analysis') {
            parallel {

                stage('Node + React') {
                    steps {
                        script {
                            def scanner = tool 'sonarscanner4'

                            withSonarQubeEnv('sonar-pro') {

                                dir('cart-microservice-nodejs') {
                                    bat "\"${scanner}\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=cart-nodejs"
                                }

                                dir('ui-web-app-reactjs') {
                                    bat "\"${scanner}\\bin\\sonar-scanner.bat\" -Dsonar.projectKey=ui-reactjs"
                                }
                            }
                        }
                    }
                }

                stage('Spring Boot') {
                    steps {
                        script {
                            def mvn = tool 'maven3'

                            withSonarQubeEnv('sonar-pro') {

                                dir('offers-microservice-spring-boot') {
                                    bat "\"${mvn}\\bin\\mvn.cmd\" clean verify sonar:sonar -Dsonar.projectKey=offers-spring"
                                }

                                dir('shoes-microservice-spring-boot') {
                                    bat "\"${mvn}\\bin\\mvn.cmd\" clean verify sonar:sonar -Dsonar.projectKey=shoes-spring"
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Build') {
            parallel {

                stage('UI') {
                    steps {
                        bat 'docker build -t ui:%BUILD_TAG% ui-web-app-reactjs'
                    }
                }

                stage('API') {
                    steps {
                        bat 'docker build -t api:%BUILD_TAG% zuul-api-gateway'
                    }
                }

                stage('Offers') {
                    steps {
                        bat 'docker build -t offers:%BUILD_TAG% offers-microservice-spring-boot'
                    }
                }

                stage('Shoes') {
                    steps {
                        bat 'docker build -t shoes:%BUILD_TAG% shoes-microservice-spring-boot'
                    }
                }

                stage('Cart') {
                    steps {
                        bat 'docker build -t cart:%BUILD_TAG% cart-microservice-nodejs'
                    }
                }

                stage('Wishlist') {
                    steps {
                        bat 'docker build -t wishlist:%BUILD_TAG% wishlist-microservice-python'
                    }
                }
            }
        }

        // ✅ NEW: Ensure Minikube is running
        stage('Ensure Minikube Running') {
            steps {
                bat '''
                minikube status || minikube start --driver=docker
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat '''
                set KUBECONFIG=%KUBECONFIG_PATH%

                kubectl config current-context
                kubectl get nodes

                kubectl create namespace ms --dry-run=client -o yaml | kubectl apply -f -

                kubectl apply -f kubernetes\\yamlfile

                kubectl get pods -n ms
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
