pipeline {
    agent any

    tools {
        nodejs 'node18'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {
        stage('Send notification to Slack') {
            steps {
                slackSend message: """\
                Starting CI/CD on job: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                <${env.BUILD_URL}console|View Output> || <${env.JOB_URL}|View Job> || <${env.JOB_DISPLAY_URL}/${env.BRANCH_NAME}| Open Blue Ocean>
                """
            }
        }

        stage('Npm install & build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

    }

    post {
        aborted {
            slackSend(color: "warning", message: "PIPELINE MANUALLY ABORTED")
        }
        failure {
            slackSend(color: "danger", message: "BUILD FAILED, PLEASE CHECK OUTPUT")
        }
        success {
            slackSend(color: "good", message: "BUILD SUCCEEDED")
        }
    }
}