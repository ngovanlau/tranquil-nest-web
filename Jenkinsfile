pipeline {
    agent any

    environment {
        DEPLOY_PATH = "tranquil-nest-web"
    }

    stages {
        stage('Clone Source') {
            steps {
                git branch: 'master', url: 'https://github.com/ngovanlau/tranquil-nest-web.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'yarn install'
            }
        }

        stage('Build') {
            steps {
                sh 'yarn build'
            }
        }

        stage('Archive Build') {
            steps {
                sh "tar -czf build.tar.gz .next public ecosystem.config.js package.json"
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def deployPath = env.DEPLOY_PATH
                    
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: "UbtService01",
                            transfers: [
                                sshTransfer(
                                    sourceFiles: "build.tar.gz",
                                    removePrefix: "",
                                    remoteDirectory: deployPath,
                                    execCommand: """
                                        mkdir -p ${deployPath} && 
                                        cd ${deployPath} && 
                                        tar -xzf build.tar.gz && 
                                        yarn install --production && 
                                        pm2 reload ecosystem.config.js || pm2 start ecosystem.config.js
                                    """.stripIndent(),
                                    execTimeout: 120000
                                )
                            ],
                            usePromotionTimestamp: false,
                            verbose: true
                        )
                    ])
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build and deploy successfully!'
        }

        failure {
            echo 'Build or deploy failed!'
        }
        
        cleanup {
            sh 'rm -f build.tar.gz'
        }
    }
}