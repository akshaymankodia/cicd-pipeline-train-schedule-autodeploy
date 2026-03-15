pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "akshaymankodia/train-schedule"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                bat './gradlew zip --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('CanaryDeploy') {
            steps {
                script {
                    bat "kubectl apply -f train-schedule-kube-canary.yml"
                }
            }
        }

        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)

                script {
                    bat "kubectl apply -f train-schedule-kube-canary.yml"
                    bat "kubectl apply -f train-schedule-kube.yml"
                }
            }
        }
    }
}
