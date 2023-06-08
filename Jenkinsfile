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
            nodejs('node18'){
                npm install
        }

    }
}