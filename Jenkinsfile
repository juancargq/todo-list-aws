pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/juancargq/todo-list-aws.git'
                sh '''
                    wget https://github.com/juancargq/todo-list-aws-config/blob/production/samconfig.toml
                    ls -la
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    sam validate --region us-east-1
                    sam build
                    sam deploy --config-file samconfig.toml --config-env production
                '''
            }
        }
        
        stage('Rest Test') {
            steps {
                script {
                    def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text", returnStdout: true).trim()
                    withEnv(["BASE_URL=${BASE_URL}"]) {
                        sh '''
                            python -m pytest -m lecture --junitxml=junit-report.xml test/integration
                        '''
                    }
                }
                junit 'junit-report.xml'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}