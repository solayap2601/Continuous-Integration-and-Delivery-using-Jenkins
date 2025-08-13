pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'chmod +x scripts/build.sh && ./scripts/build.sh'
            }
        }
        
        stage('Test') {
            steps {
                sh 'chmod +x scripts/test.sh && ./scripts/test.sh'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker build -t nodemain:v1.0 .'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t nodedev:v1.0 .'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh '''
                            docker stop $(docker ps -q --filter "ancestor=nodemain:v1.0") || true
                            docker rm $(docker ps -aq --filter "ancestor=nodemain:v1.0") || true
                            docker run -d --expose 3000 -p 3000:3000 nodemain:v1.0
                        '''
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh '''
                            docker stop $(docker ps -q --filter "ancestor=nodedev:v1.0") || true
                            docker rm $(docker ps -aq --filter "ancestor=nodedev:v1.0") || true
                            docker run -d --expose 3001 -p 3001:3000 nodedev:v1.0
                        '''
                    }
                }
            }
        }
    }
}