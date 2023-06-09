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

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "memulai CI/CD pada Branch : ${env.JOB_NAME}, Build Number: ${env.BUILD_NUMBER} link: (<${env.BUILD_URL}|Open>)")

            }
        }

        stage("npm install & build") {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }

        }

        stage("kirim folder dist ke S3 bucket") {
            steps {
                s3Upload (
                    bucket: 'demo-frontend1234',
                    excludedFile: '',
                    flatten: false,
                    gzipFiles: false,
                    keepForever: false,
                    managedArtifacts: false,
                    noUploadOnFailure: true,
                    selectedRegion: 'ap-southeast-1',
                    showDirectlyInBrowser: false,
                    sourceFile: 'dist/**',
                    storageClass: 'STANDARD',
                    uploadFromSlave: false,
                    useServerSideEncryption: false
                )

            }

        }

    }

    post {
        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

    }
}