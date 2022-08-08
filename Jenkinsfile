pipeline {
    agent any

    environment {

        AWS_ACCESS_KEY_ID     = credentials('ahmed-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('ahmed-aws-secret-access-key')

        AWS_S3_BUCKET = "ahmed-belt2d2-artifacts-123456"
        ARTIFACT_NAME = "hello-world.jar"
        AWS_EB_APP_NAME = "java-app1"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Java-app1-env"

        SONAR_IP = "54.226.50.200"
        SONAR_TOKEN = "sqp_d780c4fbce371fd81d22660309692d27bb28c75f"

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
               -Dsonar.projectKey=online-ahmed-asiri-B2D2 \
               -Dsonar.host.url=http://52.23.193.18 \
               -Dsonar.login=sqp_d780c4fbce371fd81d22660309692d27bb28c75f

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