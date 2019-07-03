#!/usr/bin/env groovy

pipeline {
    agent any

    tools {
            jdk 'openjdk11'
        }

    environment {
        APPLICATION_NAME = 'syfobtsysmock'
        DOCKER_SLUG='syfo'
    }

    stages {
        stage('initialize') {
            steps {
                init action: 'default'
                script {
                    sh(script: './gradlew clean --stacktrace')
                    def applicationVersionGradle = sh(script: './gradlew -q printVersion', returnStdout: true).trim()
                    env.APPLICATION_VERSION = "${applicationVersionGradle}.${env.BUILD_ID}-${env.COMMIT_HASH_SHORT}"
                }
            }
        }
        stage('build') {
            steps {
                sh './gradlew build -x test'
            }
        }
        stage('run tests (unit & intergration)') {
            steps {
                sh './gradlew test'
            }
        }
        stage('create uber jar') {
            steps {
                sh './gradlew shadowJar'
                slackStatus status: 'passed'
            }
        }
        stage('push docker image') {
             steps {
                 dockerUtils action: 'createPushImage'
             }
        }
        stage('deploy to preprod') {
            steps {
                deployApp action: 'kubectlDeploy', cluster: 'preprod-fss'
            }
        }
    }
    post {
        always {
            postProcess action: 'always'
            archiveArtifacts artifacts: 'build/reports/rules.csv', allowEmptyArchive: true
        }
        success {
            postProcess action: 'success'
        }
        failure {
            postProcess action: 'failure'
        }
    }
}
