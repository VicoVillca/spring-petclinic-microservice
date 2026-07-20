pipeline {
    agent {
        docker {
            image 'maven:3.8.8-eclipse-temurin-21'
        }
    }
    
    triggers {
        githubPush()
    }
    
    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=${WORKSPACE}/.m2"
        SONAR_USER_HOME = "${WORKSPACE}/.sonar"
    }
    
    stages {
        // ============================================
        // STAGE 1: COMPILE
        // ============================================
        stage('Compile') {
            steps {
                echo '📦 COMPILANDO PROYECTO...'
                sh 'mvn clean compile -B -ntp'
                echo '✅ Compilación completada'
            }
        }
        
        // ============================================
        // STAGE 2: TEST
        // ============================================
        stage('Test') {
            steps {
                echo '🧪 EJECUTANDO TESTS...'
                sh 'mvn test -B -ntp'
                echo '✅ Tests completados'
            }
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        // ============================================
        // STAGE 3: COVERAGE
        // ============================================
        stage('Coverage') {
            steps {
                echo '📊 GENERANDO REPORTE DE COBERTURA...'
                sh 'mvn jacoco:report -B -ntp'
                echo '✅ Reporte de cobertura generado'
            }
            post {
                success {
                    recordCoverage(tools: [[parser: 'JACOCO']])
                }
            }
        }
        
        // ============================================
        // STAGE 4: SONARQUBE
        // ============================================
        stage('SonarQube') {
            steps {
                echo '🔍 ANALIZANDO CÓDIGO CON SONARQUBE...'
                withSonarQubeEnv('sonarqube') {
                    script {
                        if (env.CHANGE_ID) {
                            sh """
                                mvn sonar:sonar -B -ntp \
                                -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                                -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                                -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
                            """
                        } else {
                            def branchName = GIT_BRANCH.replaceFirst('^origin/', '')
                            echo "Branch name: ${branchName}"
                            sh "mvn sonar:sonar -B -ntp -Dsonar.branch.name=${branchName}"
                        }
                    }
                }
                echo '✅ Análisis SonarQube completado'
            }
        }
        
        // ============================================
        // STAGE 5: PACKAGE + DEPLOY A ARTIFACTORY
        // ============================================
        stage('Forma 1 - Artifactory - RtMaven') {
            steps {
                echo 'Forma 1 - Artifactory - RtMaven'

                script {

                    sh 'env | sort'
                    sh 'mvn --version'
                    env.MAVEN_HOME = '/usr/share/maven'

                    def releases = 'spring-petclinic-rest-release'
                    def snapshots = 'spring-petclinic-rest-snapshot'

                    def server = Artifactory.server 'artifactory'

                    def rtMaven = Artifactory.newMavenBuild()
                    rtMaven.deployer server: server, releaseRepo: releases, snapshotRepo: snapshots

                    rtMaven.deployer
                        .addProperty('build.url', env.RUN_DISPLAY_URL)
                        .addProperty('build.user', env.USER)

                    def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -B -ntp -DskipTests'
                    server.publishBuildInfo buildInfo                    
                }
            }
        } 
    }
    
    // ============================================
    // POST - ACCIONES FINALES
    // ============================================
    post {
        success {
            echo '🎉 PIPELINE COMPLETADO EXITOSAMENTE!'
            echo '📦 Artefacto disponible en Artifactory'
        }
        failure {
            echo '❌ El pipeline falló. Revisa los logs para más detalles.'
        }
        always {
            cleanWs()
        }
    }
}