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
		            bat "msbuild TestSolution\\TestSolution.sln"
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
                    "success"
                )
            }
        }

        failure {
            script{
                sendToTelegram(
                    env.CHAT_ID,
                    env.TELEGRAM_API_CREDENTIALS_ID,
                    "failed"
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
                http://35.193.109.110:5000/sendtotelegram/\\?chat_id\\=${chatId}\\&message\\=${messageText} > /dev/null
        """
    }
}