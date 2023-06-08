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
        stage("npm npman") {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }

        }

        stage("bucket") {
            steps {
                s3CopyArtifact buildSelector: lastSuccessful(), excludeFilter: '', filter: './dist', flatten: false, optional: false, projectName: '', target: ''
            }

        }

    }
}