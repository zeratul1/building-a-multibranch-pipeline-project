pipeline {
    agent none
    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'node=inbound-agent-node01'
                    args '-p 3000-3100:3000 -p 5000-5100:5000' 
                }
            }        
            steps {
                sh "echo ${WORKSPACE}"
                sh 'npm install'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'node=inbound-agent-node01'
                    args '-p 3000-3100:3000 -p 5000-5100:5000' 
                }
            }        
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver for development') {
            agent {
                docker {
                    image 'node:18-alpine'
                    label 'node=hkdev-agent-node01'
                    args '-p 3000-3100:3000 -p 5000-5100:5000' 
                }
            }
            when {
                beforeAgent true
                branch 'development'
            }
            steps {
                sh "echo ${WORKSPACE}"
                sh 'npm install'
                sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
        stage('Deploy for staging') {
            agent { 
                docker {
                    image 'node:18-alpine'
                    label 'node=Flashwire-staging-agent'
                    args '-p 3000-3100:3000 -p 5000-5100:5000' 
                }
            }
            when {
                beforeAgent true
                branch 'production'
            }
            steps {
                sh "echo ${WORKSPACE}"
                sh 'npm install'
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}