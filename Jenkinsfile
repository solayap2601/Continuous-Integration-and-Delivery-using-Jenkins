pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }
    
    stages {
        stage('Dockerfile Lint') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint < Dockerfile'
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image 'node:7.8.0'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
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
        
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress solayap26/nodemain:v1.0", returnStdout: true).trim()
                        echo "Vulnerability Report:\n${vulnerabilities}"
                    } else if (env.BRANCH_NAME == 'dev') {
                        def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress solayap26/nodedev:v1.0", returnStdout: true).trim()
                        echo "Vulnerability Report:\n${vulnerabilities}"
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