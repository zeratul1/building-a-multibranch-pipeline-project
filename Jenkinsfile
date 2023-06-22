pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            label 'node=inbound-agent-node01'
            args '-p 3000-3100:3000 -p 5000-5100:5000' 
        }
    }
    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }
    stages {
        stage('Build') {
            steps {
                sh "echo ${WORKSPACE}"
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver for development') {
            agent { 
                node {
                    label 'node=hkdev-agent-node01'
                } 
            }
            when {
                branch 'development'
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
        stage('Deploy for staging') {
            agent { 
                node {
                    label 'node=Flashwire-staging-agent'
                } 
            }
            when {
                branch 'production'
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}