pipeline {
    agent any
    environment { //variables

        AWS_ACCESS_KEY_ID     = credentials('Raghads-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('Raghads-aws-secret-access-key')

        AWS_S3_BUCKET = "raghadalmutlaq-belt2d2-artifacts-123456"
        ARTIFACT_NAME = "hello-world.jar"
        AWS_EB_APP_NAME = "RaghadAlmutlaq-123456"
        AWS_EB_APP_VERSION = "${BUILD_ID}" // when you want to roll back
        AWS_EB_ENVIRONMENT = "	Raghadalmutlaq123456-env"
    }
    stages {
        stage('Validate') {
            steps {
                
                sh "mvn validate"

                sh "mvn clean"

            }
        }
         stage('Build') {
            steps {
                
                sh "mvn compile"

            }
        }
        stage('Package') {
            steps {
                
                sh "mvn package"

            }

            post {
                success { //when the step is successful archive the artfact
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false

                   
                }
            }
        }
        //5. Publish the artifacts
        stage('Publish artefacts to S3 Bucket') {
            steps {

                sh "aws configure set region us-east-1"

                sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
                
            }
        }

         stage('Deploy') {
            steps {

                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'

                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            
                
            }
        }
        
    }
}