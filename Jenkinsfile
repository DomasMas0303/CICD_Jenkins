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
        
        stage('Deploy') {
            steps {
                script {
                    def port = env.BRANCH_NAME == 'main' ? '3000' : '3001'
                    def imageName = env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'
                    
                    // Stop and remove existing containers for this branch
                    sh "docker ps -q --filter publish=${port} | xargs -r docker stop"
                    sh "docker ps -aq --filter publish=${port} | xargs -r docker rm"
                    
                    // Run new container
                    sh "docker run -d --expose ${port} -p ${port}:3000 domasm97/domasm:${imageName}-v1.0"
                }
            }
        }
        
        stage('Trigger Deployment') {
            steps {
                script {
                    def jobName = env.BRANCH_NAME == 'main' ? 'Deploy_to_main' : 'Deploy_to_dev'
                    build job: jobName, parameters: [
                        string(name: 'IMAGE_TAG', value: "${env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'}-v1.0")
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
