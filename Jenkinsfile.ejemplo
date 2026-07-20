pipeline {
    agent any
    
    tools {
        maven 'mvn.3.9.9'
    }
    triggers {
        githubPush()
    }
    
    stages {
        stage('Compile') {
            steps {
                sh 'mvn clean compile -B -ntp'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test -B -ntp'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Coverage') {
            steps {
                sh 'mvn jacoco:report -B -ntp'
            }
            post {
                always {
                    recordCoverage(tools: [[parser: 'JACOCO']])
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests -B -ntp'
            }
        }
    }
    
    post {
        always {
            // ✅ PRIMERO archivar, DESPUÉS limpiar
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            cleanWs()
        }
    }
}