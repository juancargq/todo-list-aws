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
                    flake8 --exit-zero --format=pylint src > flake8.out
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                }
                
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
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