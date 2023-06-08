pipeline {
    agent any

    environment {
        AUTHOR = "Reiya Tenggara"
        USERNAME = "reiya24"
    }

    stages {

        stage("npm install") {
            steps{
                nodejs(nodeJSInstallationName: 'node18') {
                    sh 'npm install'
                }
            }
        }

    }
}