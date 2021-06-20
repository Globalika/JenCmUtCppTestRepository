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
        stage('Unit Tests') {
            steps{
                dir(env.REPO_NAME){
                    bat "vstest.console TestSolution\\x64\\Debug\\LibraryUnitTests.dll"
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
                    "\\<b\\>SUCCESS!\\</b\\>\\%0A\\<i\\>branch_${env.GIT_BRANCH}\\</i\\>\\%0A\\<i\\>build_${env.BUILD_NUMBER}\\</i\\>\\%0A\\<i\\>${env.BUILD_URL}\\</i\\>"
                )
            }
        }

        failure {
            script{
                sendToTelegram(
                    env.CHAT_ID,
                    env.TELEGRAM_API_CREDENTIALS_ID,
                    "\\<b\\>FAILED!\\</b\\>\\%0A\\<i\\>branch_${env.GIT_BRANCH}\\</i\\>\\%0A\\<i\\>build_${env.BUILD_NUMBER}\\</i\\>\\%0A\\<i\\>${env.BUILD_URL}\\</i\\>"
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
                http://35.193.109.110:5000/sendtotelegram/\\?chat_id\\=${chatId}\\&message\\=${messageText}\\&parse_mode\\=HTML >> /dev/null

        """
    }
}