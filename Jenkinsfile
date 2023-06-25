pipeline {
    agent none
    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
        CHANGED_PROJECTS = ''
        // BRANCH_NAME = 'feature_flashwire_mobile'
        // BRANCH_NAME = 'staging'
        ECR_URL = '647828570435.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REGION = 'us-east-1'
        ECR_READ_CREDENTIAL_ID = 'aws-ecr-credentials-reading-for-frontend'
        ECR_READ_WRITE_CREDENTIAL_ID = 'aws-ecr-credentials-reading-writing-for-frontend'
        FLASHWIRE_MOBILE_REPO_NAME = 'flashwire-mobile'
        FLASHWIRE_MOBILE_PROJ_NAME = 'flashwire-mobile'
       // DOCKER_BUILD_VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
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
        stage('Get image for staging') {
            agent {
                node {
                    label 'node=Flashwire-staging-agent'
                }
            }
            when {
                beforeAgent true
                branch 'production'
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "$ECR_READ_CREDENTIAL_ID",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh 'aws ecr get-login-password --region $ECR_REGION | docker login --username AWS --password-stdin $ECR_URL'
                }
                sh "docker pull $ECR_URL/$FLASHWIRE_MOBILE_REPO_NAME:test1"
                sh 'pwd && ls -alh'
                sh 'uname -a'
                sh "docker image ls |grep $FLASHWIRE_MOBILE_PROJ_NAME"
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