pipeline {
    agent any
    
    tools {
        nodejs 'Node 7.8.0'
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
                    sh "docker build -t ${imageName}:v1.0 ."
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def port = env.BRANCH_NAME == 'main' ? '3000' : '3001'
                    def imageName = env.BRANCH_NAME == 'main' ? 'nodemain' : 'nodedev'
                    
                    // stop and remove existing containers for this branch
                    sh "docker ps -q --filter ancestor=${imageName}:v1.0 | xargs -r docker stop"
                    sh "docker ps -aq --filter ancestor=${imageName}:v1.0 | xargs -r docker rm"
                    
                    // run new container
                    sh "docker run -d --expose ${port} -p ${port}:3000 ${imageName}:v1.0"
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
