pipeline {
    agent any
    environment {

        AWS_ACCESS_KEY_ID     = credentials('aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')

        // Your S3 bucket name
        AWS_S3_BUCKET = "S3-BucketName" 
        ARTIFACT_NAME = "hello-world.jar"

        // Your EB App,Env name
        AWS_EB_APP_NAME = "EBApp-Name"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "EBEnv-Name" 

        // The SonarQube Project infos
        PROJECTNAME="Name of the Project"
        SONAR_IP = "SonarIP"
        SONAR_TOKEN = "SONAR TOKEN"
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

        stage('Test') {
            steps {

                sh "mvn test"

             }
            post {
                always {
                junit '**/target/surefire-reports/TEST-*.xml'
                 }
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                -Dsonar.projectKey=$PROJECTNAME \
                -Dsonar.host.url=http://$SONAR_IP \
                -Dsonar.login=$SONAR_TOKEN
                '''
             }
        }

        stage('Package') {
            steps {
                
                sh "mvn package"

            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false

                   
                }
            }
        }
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