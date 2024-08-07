pipeline {
    agent any
    environment {
        RECIPIENT_EMAIL = 'npestov9@gmail.com'
        AWS_DEFAULT_REGION = 'ap-southeast-2' // Set the AWS region to ap-southeast-2
        SONAR_TOKEN = credentials('sonarqube-token')
        PATH = "${env.PATH}:/usr/local/bin:/opt/homebrew/bin" // Add the path to docker-compose and AWS CLI
    }
    tools {
        maven 'Maven 3.9.8'
    }
    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building the code using Maven"
                    sh 'mvn clean package'
                }
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                script {
                    echo "Listing contents of the target directory"
                    sh 'ls -l target'
                }
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
                    withSonarQubeEnv('LocalSonarQube') {
                        sh 'mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}'
                    }
                }
            }
        }
        stage('Verify Docker Compose') {
            steps {
                script {
                    echo "Verifying Docker Compose installation"
                    sh 'docker-compose --version'
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
                        sh 'aws s3 cp target/SampleJavaProject-1.0-SNAPSHOT.jar s3://nikita-pestov-2003-test/SampleJavaProject-1.0-SNAPSHOT.jar'
                        def deploymentId = sh(script: 'aws deploy create-deployment --application-name my-application --deployment-group-name my-deployment-group --s3-location bucket=nikita-pestov-2003-test,key=SampleJavaProject-1.0-SNAPSHOT.jar,bundleType=zip --query "deploymentId" --output text', returnStdout: true).trim()
                        echo "Created deployment with ID: ${deploymentId}"
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
