pipeline {
    agent any
    
    tools {
        nodejs 'Node 7.8.0'
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'
                    sh "docker build -t domasm97/domasm:${imageName}-v1.0 ."
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    def imageName = env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    sh "docker push domasm97/domasm:${imageName}-v1.0"
                }
            }
        }
        
        stage('Trigger Deployment') {
            steps {
                script {
                    def jobName = env.BRANCH_NAME == 'main' ? 'Deploy_to_main' : 'Deploy_to_dev'
                    def imageTag = env.BRANCH_NAME == 'main' ? 'nodemain-v1.0' : 'nodedev-v1.0'
                    build job: jobName, parameters: [
                        string(name: 'IMAGE_TAG', value: imageTag)
                    ]
                }
            }
        }
    }
    
    post {
        always {
            sh "docker logout"
            cleanWs()
        }
    }
}

