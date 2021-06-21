pipeline {
    agent any
    
    environment {
        REPO_NAME = 'JenCmUtCppTestRepository'
        CHAT_ID = '676352139'
        TELEGRAM_API_CREDENTIALS_ID = 'telegram-api-token'
    }

    stages {
        stage('Checkout repository') {
            steps {
                dir(env.REPO_NAME){
                    // Clean workspace
                    cleanWs()
                    // To clone and checkout scm job repo
                    checkout scm
                }
            }
        }
        stage('Build') {
            steps{
                dir(env.REPO_NAME){
                    bat "nuget restore TestSolution\\TestSolution.sln"
		            bat "msbuild TestSolution\\TestSolution.sln"
                }
            }
        }
        stage('Unit Tests') {
            steps{
                dir(env.REPO_NAME){
                    bat "vstest.console TestSolution\\x64\\Debug\\LibraryUnitTests.dll"
                    bat "TestSolution\\x64\\Debug\\LibraryGoogleTests.exe"
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: "${env.REPO_NAME}/TestSolution/x64/Debug/*", fingerprint: true
            script {
                sendToTelegram(
                    env.CHAT_ID,
                    env.TELEGRAM_API_CREDENTIALS_ID,
                    "<b>SUCCESS!</b>%0A<i>branch: ${env.GIT_BRANCH}</i>%0A<a href=\"${env.BUILD_URL}\">build ${env.BUILD_NUMBER}</a>"
                )
            }
        }

        failure {
            script{
                sendToTelegram(
                    env.CHAT_ID,
                    env.TELEGRAM_API_CREDENTIALS_ID,
                    "<b>FAILURE!</b>%0A<i>branch: ${env.GIT_BRANCH}</i>%0A<a href=\"${env.BUILD_URL}\">build ${env.BUILD_NUMBER}</a>"
                )
            }
        }
    }

}

def sendToTelegram(chatId, credentialsId, messageText) {
    withCredentials([string(credentialsId: 'telegram-api-token', variable: 'TOKEN')]){
        sh """
            curl -X POST \
                 -H "Authorization: Bearer $TOKEN" \
                 -d "chat_id=${chatId}" \
                 -d 'message=${messageText}' \
                 -d "parse_mode=HTML" \
                 http://35.193.109.110:5000/sendtotelegram/ >> /dev/null
        """
    }
}
