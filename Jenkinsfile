pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Test') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=95c0b21d6c6ed08e930d921ee071a88ae91bced1 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-teste') {
                    git credentialsId: 'GitHub', url: 'https://github.com/rafael-bcardoso/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('deploy-frontend') {
                    git credentialsId: 'GitHub', url: 'https://github.com/rafael-bcardoso/tasks-frontend'
                    bat 'mvn clean package -DskipTests=true'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'GitHub', url: 'https://github.com/rafael-bcardoso/taks-functional-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Prod') {
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(10 )
                dir('functional-test') {
                    bat 'mvn verify'
                }
            }
        }
    }
    post{
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-teste/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
    }
}

