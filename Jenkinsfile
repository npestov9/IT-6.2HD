pipeline {
    agent any
    environment {
        RECIPIENT_EMAIL = 'npestov9@gmail.com'
        AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
    }
    tools {
        maven 'Maven 3.9.8' // Ensure this matches the Maven version configured in Jenkins
    }
    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building the code using Maven"
                    sh 'mvn clean package'
                }
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        stage('Test') {
            steps {
                script {
                    echo "Running unit tests using JUnit"
                    sh 'mvn test'
                }
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Quality Analysis') {
            steps {
                script {
                    echo "Running code quality analysis using SonarQube"
                    withSonarQubeEnv('SonarQube') { 
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                script {
                    echo "Deploying to staging environment using Docker Compose"
                    sh 'docker-compose -f docker-compose-staging.yml up -d'
                }
            }
        }
        stage('Deploy to Production') {
            steps {
                script {
                    echo "Deploying to production environment using AWS CodeDeploy"
                    withAWS(credentials: 'aws-credentials', region: "${env.AWS_DEFAULT_REGION}") {
                        // Upload artifact to S3
                        sh 'aws s3 cp target/my-app.jar s3://my-bucket/my-app.jar'
                        
                        // Create deployment
                        def deploymentId = sh(script: 'aws deploy create-deployment --application-name my-application --deployment-group-name my-deployment-group --s3-location bucket=my-bucket,key=my-app.jar,bundleType=zip --query "deploymentId" --output text', returnStdout: true).trim()
                        echo "Created deployment with ID: ${deploymentId}"
                        
                        // Wait for deployment to complete
                        sh "aws deploy wait deployment-successful --deployment-id ${deploymentId}"
                    }
                }
            }
        }
        stage('Monitoring and Alerting') {
            steps {
                script {
                    echo "Setting up monitoring and alerting"
                    sh './setup-monitoring.sh'
                }
            }
        }
    }
    post {
        always {
            emailext subject: "Pipeline Notification",
                     body: "Pipeline has completed with status: ${currentBuild.currentResult}",
                     to: "${env.RECIPIENT_EMAIL}",
                     attachLog: true
        }
    }
}
