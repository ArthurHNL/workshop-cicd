pipeline {
    agent {
        label 'master'
    }
    stages {
        stage('Prepare') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm install'
                }
                dir('code/backend'){
                    sh 'npm install'
                }
            }
        }
        stage('Build') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm run build'
                }
                dir('code/backend'){
                    sh 'npm run build'
                }  
            }
        }
        stage('Static Analysis') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm run lint'
                }
                dir('code/backend'){
                    sh 'npm run lint'
                }
            }
        }
        stage('Unit Test') {
            agent {
                docker { image 'node:alpine' }
            }
            steps {
                dir('code/frontend'){
                    sh 'npm run test'
                }
                dir('code/backend'){
                    sh 'npm run test'
                }
            }
        }
        stage('e2e Test') {
            steps {             
                dir ('ci/code') {
                    sh 'docker-compose -f docker-compose-e2e.yml build'
                    sh 'docker-compose -f docker-compose-e2e.yml up -d frontend backend'
                    script {
                        sh 'docker-compose -f docker-compose-e2e.yml up e2e'
                        status_code = sh ( script: "docker inspect code_e2e_1 --format='{{.State.ExitCode}}'", returnStdout: true).trim();
                        if (status_code == '1'){
                            error('e2e test failed.')
                       }
                    }
                }
            }
            post {
                always {
                    sh 'docker-compose -f docker-compose-e2e.yml down --rmi=all -v'
                }
            }
        }
        stage('Deploy') {
            dir('ci/code') {
                script {
                    sh 'docker-compose -f docker-compose.yml build'
                    sh 'docker-compose up -d'
                }
            }
        }
    }
}
