pipeline{
    agent any
    environment{
        DOCKER_CREDS = credentials('docker-credentials')
        }
        stages{
            stage('build'){
                agent {
                    docker {
                        image 'maven:3.6.3-openjdk-11-slim'
                    }
                }   
                steps {
                    script {                        
                        sh 'mvn clean package'
                        sh 'mvn -B verify'                        
                    }                    
                }
                post{
                    success {
                        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                    }
                }   
            }
            stage('SonarQube'){
                agent {
                    docker { image 'maven:3.6.3-openjdk-11-slim' }
                }
                steps {
                    script{
                        def scannerHome = tool 'scanner-default'
                        withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=labmaven01 \
                            -Dsonar.projectName=labmaven01 \
                            -Dsonar.sources=src/main \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.language=java \
                            -Dsonar.tests=src/test \
                            -Dsonar.junit.reportsPath=target/surefire-reports \
                            -Dsonar.surefire.reportsPath=target/surefire-reports \
                            -Dsonar.jacoco.reportPath=target/jacoco.exec \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.coveragePlugin=jacoco \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.xml \
                            -Dsonar.exclusions=**/*IT.java,**/*TEST.java,**/*Test.java,**/src/it**,**/src/test**,**/gradle/wrapper** \
                            -Dsonar.java.libraries=target/*.jar"
                        }
                    }                    
                }
            }
            stage('build image'){
                steps {
                    copyArtifacts filter: 'build/libs/labmaven-*-SNAPSHOT.jar',
                                    fingerprintArtifacts: true,
                                    projectName: '${JOB_NAME}',
                                    flatten: true,
                                    selector: specific('${BUILD_NUMBER}'),
                                    target: 'build/libs/'
                    sh 'docker --version'
                    sh 'docker-compose --version'
                    sh 'docker-compose build'
                }
            }
            stage('push docker'){
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker tag msmicroservice ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker push ${DOCKER_CREDS_USR}/msmicroservice2:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
            stage('run container'){
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker rm galaxyLab -f'
                        sh 'docker run -d -p 8080:8080 --name galaxyLab ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        //sh 'docker run -d -p 8080:8080 ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
        }
}
