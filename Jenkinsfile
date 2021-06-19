pipeline {
    agent any
    
    environment {
        REPO_BRANCH = 'master'
        // REPO_URL = 'https://github.com/Globalika/JenCmUtCppTestRepository.git'
        REPO_NAME = 'JenCmUtCppTestRepository'
        GITHUB_CREDENTIALS = 'github-token'
    }
    
    //parameters {
    //    
    //}
    // env.MY_ENV_VAR
    // params.MY_PARAM
    stages {
        stage('Git Clone') {
            steps {
                dir(env.REPO_NAME){
                    cleanWs()
                    checkout scm
                    // git(
                    //     branch: env.REPO_BRANCH,
                    //     credentialsId: env.GITHUB_CREDENTIALS,
                    //     url: env.REPO_URL
                    // )
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
        }
    }

}