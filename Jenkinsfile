pipeline {
    agent any
    
    environment {
        REPO_NAME = 'JenCmUtCppTestRepository'
        CHAT_ID = '676352139'
        TELEGRAM_API_CREDENTIALS_ID = 'telegram-api-token'
        EMAIL_RECIPIENTS = 'volodymyr.seredovych@gmail.com'
    }

    stages {
        stage('Checkout repository') {
            steps {
                dir(env.REPO_NAME){
                    echo "Clean Workspace"
                    cleanWs()
                    echo "Clone and checkout scm job repository"
                    checkout scm
                }
            }
        }
        stage('Build') {
            steps{
                dir(env.REPO_NAME){
                    echo "restore nuget packages"
                    bat "nuget restore TestSolution\\TestSolution.sln"
                    echo "build solution"
		            bat "msbuild TestSolution\\TestSolution.sln"
                }
            }
        }
        stage('Static Analize with PVS-Studio')
        {
            steps{
                dir(env.REPO_NAME){
                    echo "Analize Solution"
                    bat "PVS-Studio_Cmd.exe -t TestSolution\\TestSolution.sln -o report.plog --progress"
                }
            }
        }
        stage('Unit Tests') {
            steps{
                dir(env.REPO_NAME){
                    echo "execute native unit tests"
                    bat "vstest.console TestSolution\\x64\\Debug\\LibraryUnitTests.dll"
                    echo "execute google unit tests"
                    bat "TestSolution\\x64\\Debug\\LibraryGoogleUnitTests.exe"
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: "${env.REPO_NAME}/TestSolution/x64/Debug/*", fingerprint: true
            script {
                mail to: env.EMAIL_RECIPIENTS,
                    subject: "SUCCESS!",
                    body:" branch: ${env.GIT_BRANCH} \n build: ${env.BUILD_NUMBER} \n ${env.BUILD_URL} "
                sendToTelegram(
                    env.CHAT_ID,
                    env.TELEGRAM_API_CREDENTIALS_ID,
                    "<b>SUCCESS!</b>%0A<i>branch: ${env.GIT_BRANCH}</i>%0A<a href=\"${env.BUILD_URL}\">build ${env.BUILD_NUMBER}</a>"
                )
            }
        }
        always {
            recordIssues enabledForFailure: true, tool: msBuild()
            recordIssues enabledForFailure: true, sourceCodeEncoding:'UTF-8',
                tool: PVSStudio(pattern: "${env.REPO_NAME}\\report.plog")
        }
        failure {
            script{
                mail to: env.EMAIL_RECIPIENTS,
                    subject: "FAILURE!",
                    body:" branch: ${env.GIT_BRANCH} \n build: ${env.BUILD_NUMBER} \n ${env.BUILD_URL} "
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
