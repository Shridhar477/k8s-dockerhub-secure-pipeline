pipeline {
    agent any

    tools {
        jdk 'Java'
        maven 'M3'
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git url: "https://github.com/Shridhar477/k8s-dockerhub-secure-pipeline.git", branch: "main"
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectName=register-app \
                        -Dsonar.projectKey=register-app
                    '''
                }
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Docker Build") {
            steps {
                withCredentials([usernamePassword(credentialsId: "DockerHub", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t register-app:latest .
                    """
                }
            }
        }
        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: "DockerHub", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag register-app:latest $DOCKER_USER/register-app:latest
                        docker push $DOCKER_USER/register-app:latest
                        docker logout
                    """
                }
            }
        }
        stage("Deploy using K8s manifest") {
            steps {
                sh "kubectl create -f  ."
            }
        }
    }
}
