pipeline {
    agent any

   tools {nodejs "node18"}
    environment {
        AUTHOR = "Reiya Tenggara"
        USERNAME = "reiya24"
    }

    stages {
    nodejs('node18'){
        npm install
        npm run prod
}

    }
}