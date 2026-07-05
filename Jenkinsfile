pipeline {
    agent { label 'worker' }

    environment {
        IMAGE_NAME  = 'emc-nodejs-app:v1'
        DOCKER_REPO = 'boom04/emc-nodejs-app2'
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Build Checkout') {
            steps {
                git branch: 'main', credentialsId: 'Git-Token', url: 'https://github.com/Boom-28/emc-nodejs-app.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=emc-nodejs-app \
                          -Dsonar.sources=. \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('push to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker tag $IMAGE_NAME $DOCKER_REPO:v1
                        docker push $DOCKER_REPO:v1
                    '''
                }
            }
        }

        stage('Docker pull') {
            steps {
                sh 'docker pull $DOCKER_REPO:v1'
            }
        }

        stage('Deploy app') {
            steps {
                sh '''
                    docker rm -f emc-node-app || true
                    docker run -d -p 80:3000 --name emc-node-app $DOCKER_REPO:v1
                '''
            }
        }
    }
}
