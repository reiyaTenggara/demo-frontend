pipeline {
    agent any

    tools {
      nodejs 'node18'
    }

    environment {
        AUTHOR = "Reiya Tenggara"
        USERNAME = "reiya24"
    }

    stages {
        stage("npm install") {
            steps {
                sh 'npm install'
            }
        }

    }
}