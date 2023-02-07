def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools{
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    environment {
        registry = '868016059835.dkr.ecr.us-east-1.amazonaws.com/devop_repository'
        registryCredential = 'jenkins-ecr'
        dockerimage = ''
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/Niandre95/Qualitygate_AWS.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Checkstyle Analysis'){
          steps {
              sh 'mvn checkstyle:checkstyle'
          }

        }
        
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonarQube'
            }
            steps {
               withSonarQubeEnv('sonarQubes') {
                   sh '''mvn sonar:sonar -Dsonar.projectKey=Niandre95_simple_project \
                   -Dsonar.organization=niandre95 \
                   -Dsonar.projectName=Niandre95_simple_project \
                   -Dsonar.host.url=https://sonarcloud.io \
                   -Dsonar.login=5d6645735c446ba0da665a82856caae970c8a68d \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Image') {
            steps {
                script{
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                } 
            }
        }
        stage('Deploy image') {
            steps{
                script{ 
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}

