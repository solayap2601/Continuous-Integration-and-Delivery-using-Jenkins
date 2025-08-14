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
                        sh 'docker build -t solayap26/nodemain:v1.0 .'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t solayap26/nodedev:v1.0 .'
                    }
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry-1.docker.io/v2/', 'docker-hub-credentials') {
                        if (env.BRANCH_NAME == 'main') {
                            sh 'docker push solayap26/nodemain:v1.0'
                        } else if (env.BRANCH_NAME == 'dev') {
                            sh 'docker push solayap26/nodedev:v1.0'
                        }
                    }
                }
            }
        }
        
        stage('Trigger Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main', wait: false
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev', wait: false
                    }
                }
            }
        }
    }
}