#!/usr/bin/env groovy

pipeline {
    agent any
    triggers {
        cron('H H(13-14) * * *')
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/branddcast/quarkus-ex.git'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw -U clean install -DskipTests'
            }
        }
        stage('Test JVM') {
            options {
                timeout(time: 120, unit: 'MINUTES')
            }
            steps {
                sh './mvnw -fn clean verify -Ddocker -Dtest-mariadb -Dtest-mssql -Dtest-mysql -Dtest-postgresql -Dtest-gelf -Dtest-neo4j -Dtest-kafka -Dtest-elasticsearch -Dtest-vault -Dtest-dynamodb -Dtest-keycloak -DskipDocs'
            }
            post {
                always {
                    junit '**/target/*-reports/TEST*.xml'
                }
            }
        }
        stage('Test Native') {
            options {
                timeout(time: 4, unit: 'HOURS')
            }
            steps {
                sh 'GRAALVM_HOME=$JAVA_HOME ./mvnw -fn clean verify -Dnative -Ddocker -Dtest-mariadb -Dtest-mssql -Dtest-mysql -Dtest-postgresql -Dtest-gelf -Dtest-neo4j -Dtest-kafka -Dtest-elasticsearch -Dtest-vault -Dtest-dynamodb -Dtest-keycloak -DskipDocs'
            }
            post {
                always {
                    junit '**/target/*-reports/TEST*.xml'
                }
            }
        }
        stage('Reports') {
            parallel {
                stage('Disk usage') {
                    steps {
                        sh 'du -cskh */*'
                    }
                }
                stage('Archive artifacts') {
                    steps {
                        archiveArtifacts artifacts: '**/target/*-reports/TEST*.xml', fingerprint: false
                    }
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}