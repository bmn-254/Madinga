pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'your-docker-image'
        SONARQUBE_SERVER = 'http://your-sonarqube-server'
        SONARQUBE_TOKEN = 'your-sonarqube-token'
        AWS_REGION = 'your-aws-region'
        S3_BUCKET = 'your-s3-bucket'
        CODEDEPLOY_APP = 'your-codedeploy-app'
        CODEDEPLOY_GROUP = 'your-codedeploy-group'
        NEWRELIC_API_KEY = 'your-newrelic-api-key'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    docker.build("$DOCKER_IMAGE:${env.BUILD_ID}")
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.image("$DOCKER_IMAGE:${env.BUILD_ID}").inside {
                        sh 'mvn test'
                    }
                }
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=your-project-key -Dsonar.host.url=$SONARQUBE_SERVER -Dsonar.login=$SONARQUBE_TOKEN'
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                script {
                    docker.image("$DOCKER_IMAGE:${env.BUILD_ID}").push()
                    sh "aws s3 cp docker-compose.yml s3://$S3_BUCKET/staging/docker-compose.yml"
                    sh "aws deploy create-deployment --application-name $CODEDEPLOY_APP --deployment-group-name $CODEDEPLOY_GROUP --s3-location bucket=$S3_BUCKET,bundleType=yml,key=staging/docker-compose.yml"
                }
            }
        }
        stage('Release to Production') {
            steps {
                script {
                    sh "aws s3 cp docker-compose.yml s3://$S3_BUCKET/production/docker-compose.yml"
                    sh "aws deploy create-deployment --application-name $CODEDEPLOY_APP --deployment-group-name $CODEDEPLOY_GROUP --s3-location bucket=$S3_BUCKET,bundleType=yml,key=production/docker-compose.yml"
                }
            }
        }
        stage('Monitoring and Alerting') {
            steps {
                script {
                    // Configure New Relic monitoring
                    sh "curl -X POST 'https://api.newrelic.com/v2/alerts_conditions/policies.json' \
                        -H 'X-Api-Key:$NEWRELIC_API_KEY' \
                        -H 'Content-Type: application/json' \
                        -d '{\"condition\": {\"name\": \"High Error Rate\", \"type\": \"apm_app_metric\", \"entities\": [\"YOUR_APP_ID\"], \"metric\": \"error_percentage\", \"runbook_url\": \"http://my-runbook.com\", \"terms\": [{\"duration\": \"5\", \"operator\": \"above\", \"priority\": \"critical\", \"threshold\": \"5\", \"time_function\": \"all\"}], \"user_defined\": {\"metric\": \"\", \"value_function\": \"average\"}, \"violation_close_timer\": \"24\", \"enabled\": true, \"expected_groups\": \"0\", \"ignore_overlap\": false, \"nrql\": {\"query\": \"SELECT count(*) FROM Transaction WHERE appName = 'My Application'\", \"since_value\": \"3\"}, \"policy_id\": \"YOUR_POLICY_ID\"}}'"
                }
            }
        }
    }
    post {
        failure {
            mail to: 'nkongebryan44@gmail.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something is wrong with ${env.JOB_NAME}."
        }
    }
}
