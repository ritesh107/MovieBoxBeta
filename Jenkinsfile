pipeline {
    agent any
    parameters {
        choice(name: 'skip_test', choices: ['Yes', 'No'], description: 'Select the environment')
    }
    environment {
        APP_CENTER_API_TOKEN = credentials('appcenter02')
    }
    tools {
        git 'Default'
        jdk 'java'
        gradle 'gradle'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ritesh107/MovieBoxBeta.git']])
            }
        }
        stage('Test') {
            when { expression { params.skip_test == 'No' } }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'gradle test'
                    }
                }
                // You can add more parallel stages here if needed
            }
        }
        stage('Build') {
            steps {
                sh '''
                    gradle clean
                    gradle build
                '''
            }
        }
        stage('Archive APK') {
            steps {
                archiveArtifacts artifacts: '**/*debug.apk', followSymlinks: false
            }
        }
        stage('Deploy to App Center') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'appcenter02', variable: 'APP_CENTER_API_TOKEN')]) {
                        appCenter apiToken: APP_CENTER_API_TOKEN, appName: 'MovieBox', branchName: '', buildVersion: '1.1', commitHash: '', distributionGroups: 'app-release', mandatoryUpdate: false, notifyTesters: true, ownerName: 'abhiverma', pathToApp: '**/*debug.apk', pathToDebugSymbols: '', pathToReleaseNotes: '', releaseNotes: ''
                    }
                }
            }
        }
    }
    // Post-build actions
    post {
        success {
            slackSend channel: 'jenkins_report', color: '#439FE0', message: "JOB_NAME: ${env.JOB_NAME} BUILD_ID: ${env.BUILD_ID} WORKSPACE: ${env.WORKSPACE} Successful", teamDomain: 'opstree', tokenCredentialId: 'slack-tokenn', username: 'Ritesh Kumar'
            emailext body: 'Build Success', recipientProviders: [developers()], replyTo: 'riteshkuma19325@gmail.com', subject: 'Pipeline 04', to: 'riteshkumar19325@gmail.com'
        }
        failure {
            slackSend channel: 'jenkins_report', color: '#439FE0', message: "JOB_NAME: ${env.JOB_NAME} BUILD_ID: ${env.BUILD_ID} WORKSPACE: ${env.WORKSPACE} Un-Successful", teamDomain: 'opstree', tokenCredentialId: 'slack-tokenn', username: 'Ritesh Kumar'
        }
    }
}
