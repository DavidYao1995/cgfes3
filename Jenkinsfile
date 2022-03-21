pipeline {
    agent any

    environment {
        ENVIRONMENT = 'uat'
        BRANCH_UAT = 'master'
        S3_CREDENTIAL = 'S3'
        S3_REGION = 'ap-southeast-2'
        BUCKET_NAME = 'creativegang-s3-frontend'
        WORKSPACE_PATH = '/var/jenkins_home/workspace/gc-frontend-s3_publisher/build' 
    }

    options {
        // Keep maximum 10 archieved artifacts
        buildDiscarder(logRotator(numToKeepStr:'10', artifactNumToKeepStr:'10'))
        // No simultaneous builds
        disableConcurrentBuilds()
        durabilityHint('PERFORMANCE_OPTIMIZED') //MAX_SURVIVABILITY or SURVIVABLE_NONATOMIC
    }

    stages {
        
        stage ('git checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/DavidYao-Z/P3-TestRepo-FE.git']]])
            }
        }

        stage('Install packages') {
            steps {
                echo "Installing packages ..."
                //Install the packages from package.json
                sh 'npm install'
            }
        }
        stage('Build') {
            steps {
                echo "Building ..."
                echo "Running job: ${env.JOB_NAME}\n Build: ${env.BUILD_ID} - ${env.BUILD_URL}\nPepeline: ${env.RUN_DISPLAY_URL}"
                sh 'npm run build'
            }
        }

        stage('Install AWS CLI') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo "Installing AWS CLI ..."
                sh 'apt-get update'
                sh 'apt install python3-pip -y'
                sh 'pip3 install awscli --upgrade'
            }
        }

        stage('Deploy to UAT') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                deployToS3(ENVIRONMENT)

            }
        }
    }

    post {
        success {
            echo "WELL DONE!!"
        }
        failure {
            echo "FAILED"
        }
    }
}

 def deployToS3(environment) {
    echo 'Deploying to ' + environment + ' ...'
    withAWS(credentials: S3_CREDENTIAL, region: S3_REGION) {
        // Empty the UAT bucket
        sh 'aws s3 rm "${BUCKET_NAME}" --recursive' // ${BUCKET_NAME} is also fine
        // Copy the static files from workspace to the S3 bucket
        sh 'aws s3 cp "${WORKSPACE_PATH}" "${BUCKET_NAME}" --recursive --acl public-read'
    }
}

