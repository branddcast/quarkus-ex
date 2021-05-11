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
