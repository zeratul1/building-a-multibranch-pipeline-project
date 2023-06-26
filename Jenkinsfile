pipeline {
    agent none
    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
        CHANGED_PROJECTS = ''
        ECR_URL = '647828570435.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REGION = 'us-east-1'
        ECR_READ_CREDENTIAL_ID = 'aws-ecr-credentials-reading-for-frontend'
        ECR_READ_WRITE_CREDENTIAL_ID = 'aws-ecr-credentials-reading-writing-for-frontend'
        FLASHWIRE_MOBILE_REPO_NAME = 'flashwire-mobile'
        DOCKER_BUILD_TAG = "${params.image_type}-${BUILD_NUMBER}-${BRANCH_NAME}-${GIT_COMMIT}-${currentBuild.timeInMillis}"
    }
    parameters {
        booleanParam(
            name: 'skip_build',
            defaultValue: false,
            description: 'Set to true to skip the build stage'
        )
        booleanParam(
            name: 'skip_test',
            defaultValue: false,
            description: 'Set to true to skip the test stage'
        )
        choice(
            name: 'image_type', 
            choices: [
                'development', 'staging', 'production'
            ], 
            description: 'choose the type of image'
        )
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
            when {
                expression {
                    return params.skip_build != true
                }
            }
            steps {
                echo "${DOCKER_BUILD_TAG}"
                echo "${WORKSPACE}"
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
            when {
                expression {
                    return params.skip_test != true
                }
            }
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Query image') {
            agent {
                node {
                    label 'node=Flashwire-staging-agent || node=hkdev-agent-node01'
                }
            }
            when {
                branch 'development'
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "$ECR_READ_CREDENTIAL_ID",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh 'aws ecr list-images --region $ECR_REGION --repository-name $FLASHWIRE_MOBILE_REPO_NAME'
                    echo "${DOCKER_BUILD_TAG}"
                }
            }
        }
        stage('Get image') {
            agent {
                node {
                    label 'node=Flashwire-staging-agent || node=hkdev-agent-node01'
                }
            }
            when {
                branch 'production'
                expression {
                    return NODE == NODE_NAME
                }
            }
            input {
                message "Should we continue?"
                ok "Yes, we should."
                parameters {
                    choice(
                        name: 'NODE', 
                        choices: [
                            'hkdev-agent-node01', 'Flashwire-staging-agent'
                        ], 
                        description: 'Which node should I work on?'
                    )
                }
            }
            steps {
                echo "Hello, ${NODE}. [${NODE_NAME}]"
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
                sh "docker image ls |grep $FLASHWIRE_MOBILE_REPO_NAME"
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
                expression {
                    return params.continue_deploy == true
                }
            }
            input {
                message 'Continue to deploy?'
                ok 'Submit'
                parameters {
                    booleanParam(
                        name: 'continue_deploy', 
                        defaultValue: true
                    )
                }
            }
            steps {
                echo "${WORKSPACE}"
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
                input(
                    message: 'Continue to deploy?',
                    ok: 'Submit',
                    parameters: [
                        booleanParam(
                            name: 'continue_deploy', 
                            defaultValue: true
                        )
                    ]
                )
                script {
                    if(params.continue_deploy) {
                        echo "${WORKSPACE}"
                        sh 'npm install'
                        sh './jenkins/scripts/deploy-for-production.sh'
                        input message: 'Finished using the web site? (Click "Proceed" to continue)'
                        sh './jenkins/scripts/kill.sh'
                        return
                    }
                }
            }
        }
    }
}