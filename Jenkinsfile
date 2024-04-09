COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3.9.6"
    }

    stages {
        stage("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/moscos1991/web-app.git'
            }
        }
   
        stage("Build with Maven") {
            steps {
                sh "mvn clean"
            }
        }

        stage("Testing with Maven") {
            steps {
                sh "mvn test"
            }
        }

        stage("Package with Maven") {
            steps {
                sh "mvn package"
            }
        }
        
        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonar5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }
    
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        
        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/first-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id1', groupId: 'com.mt', nexusUrl: '54.87.215.41:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.0.6-Release'
            }
        }

        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat4(credentialsId: 'tomcat-id1', path: '', url: 'http://54.224.153.33:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
    }

    post {
        success {
            slackSend channel: 'team3-africa', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'team3-africa', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }

}

