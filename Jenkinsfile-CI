pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/juancargq/todo-list-aws.git'
                sh 'ls -la'
            }
        }   
        
        stage('Static Test') {
            steps {
                sh '''
                    export PATH=/var/lib/jenkins/.local/bin:$PATH
                    flake8 --exit-zero --format=pylint src > flake8.out
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    
                    if [ ! -f flake8.out ] || [ ! -f bandit.out ]; then
                        echo "Error: Uno o ambos archivos de salida no se generaron."
                        exit 1
                    fi
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]

                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    sam validate --region us-east-1
                    sam build
                    sam deploy --config-file samconfig.toml --config-env staging
                '''
            }
        }
        
        stage('Rest Test') {
            steps {
                script {
                    def BASE_URL = sh(script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text", returnStdout: true).trim()
                    withEnv(["BASE_URL=${BASE_URL}"]) {
                        sh '''
                            python -m pytest --junitxml=junit-report.xml test/integration
                        '''
                    }
                }
                junit 'junit-report.xml'
            }
        }
        
        stage('Promote') {
            steps {
                withCredentials ([
                    gitUsernamePassword(credentialsId: 'github-credentials', gitToolName: 'Default')
                ]) {
                sh '''
                    git checkout master
                    git merge develop
                    git push origin master
                '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}