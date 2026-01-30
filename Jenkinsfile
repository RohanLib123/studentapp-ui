pipeline {
    agent any

    tools {
        jdk 'jdk-17'
        maven 'maven-3.9'
    }

    environment {
        APP_NAME = "studentapp-ui"
        S3_BUCKET = "test-jenkins-artifact-bucket-01"
        AWS_REGION = "us-east-1"
        DEV_INSTANCE_ID = "i-07277384225113940"
        TEST_INSTANCE_ID = "i-0c727ff82d65d83f8"
        PROD_INSTANCE_ID = "i-0d49b1b656895a7f2"
    }

    stages {

        /* ---------------- CHECKOUT ---------------- */
        stage('Checkout - Dev') {
            steps {
                git branch: 'dev',
                    credentialsId: 'github-token',
                    url: 'https://github.com/RohanLib123/studentapp-ui.git'
            }
        }

        /* ---------------- BUILD ---------------- */
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        /* ---------------- UNIT TEST ---------------- */
        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        /* ---------------- PACKAGE ---------------- */
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        /* ---------------- UPLOAD ARTIFACT ---------------- */
        stage('Upload Artifact to S3') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins-creds'
                ]]) {
                    sh '''
                    aws s3 cp target/*.jar \
                    s3://${S3_BUCKET}/${APP_NAME}/${BUILD_NUMBER}/app.jar \
                    --region ${AWS_REGION}
                    '''
                }
            }
        }

        /* ---------------- DEPLOY TO DEV ---------------- */
        stage('Deploy to Dev') {
            steps {
                sh """
                aws ssm send-command \
                --instance-ids ${DEV_INSTANCE_ID} \
                --document-name "AWS-RunShellScript" \
                --parameters commands=[
                    "aws s3 cp s3://${S3_BUCKET}/${APP_NAME}/${BUILD_NUMBER}/app.jar /opt/app/app.jar",
                    "systemctl restart ${APP_NAME}"
                ] \
                --region ${AWS_REGION}
                """
            }
        }

        /* ---------------- MANUAL APPROVAL ---------------- */
        stage('Approve for Test') {
            steps {
                input message: 'Approve deployment to TEST?'
            }
        }

        /* ---------------- DEPLOY TO TEST ---------------- */
        stage('Deploy to Test') {
            steps {
                sh """
                aws ssm send-command \
                --instance-ids ${TEST_INSTANCE_ID} \
                --document-name "AWS-RunShellScript" \
                --parameters commands=[
                    "aws s3 cp s3://${S3_BUCKET}/${APP_NAME}/${BUILD_NUMBER}/app.jar /opt/app/app.jar",
                    "systemctl restart ${APP_NAME}"
                ] \
                --region ${AWS_REGION}
                """
            }
        }

        /* ---------------- MANUAL APPROVAL ---------------- */
        stage('Approve for Prod') {
            steps {
                input message: 'Approve deployment to PROD?'
            }
        }

        /* ---------------- DEPLOY TO PROD ---------------- */
        stage('Deploy to Prod') {
            steps {
                sh """
                aws ssm send-command \
                --instance-ids ${PROD_INSTANCE_ID} \
                --document-name "AWS-RunShellScript" \
                --parameters commands=[
                    "aws s3 cp s3://${S3_BUCKET}/${APP_NAME}/${BUILD_NUMBER}/app.jar /opt/app/app.jar",
                    "systemctl restart ${APP_NAME}"
                ] \
                --region ${AWS_REGION}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
