pipeline {
        agent any

        stages{
        stage('Get Code') {
            steps {
                // Obtener código del repositorio
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/develop']], userRemoteConfigs: [[url: 'https://github.com/danivazeste/ToDo.git']]])
            }
        }

        stage('Static Analysis') {
            parallel {
                stage('flake8') {
                    steps {
                        sh '''
                            flake8 --exit-zero --format=pylint src > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [
                            [threshold: 10, type: 'TOTAL', unstable: false],
                            [threshold: 12, type: 'TOTAL', unstable: false]
                        ]
                    }
                }
                stage('bandit') {
                    steps {
                        sh '''
                            bandit --exit-zero -r src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [
                            [threshold: 1, type: 'TOTAL', unstable: false],
                            [threshold: 2, type: 'TOTAL', unstable: false]
                        ]
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Construir, validar y desplegar usando AWS SAM
                    sh 'sam build'
                    sh 'sam validate'
                    sh 'sam deploy --no-confirm-changeset --no-fail-on-empty-changeset --stack-name my-stack --capabilities CAPABILITY_IAM'
                }
            }
        }

        stage('Deplo') {
            steps {
                script {
                    def already_deployed = sh(script: '''
                        aws cloudformation describe-stacks --stack-name todo-list-aws-production --region us-east-1
                    ''', returnStatus: true)

                    if (already_deployed != 0) {
                        sh '''
                            sam build
                            sam validate --config-file samconfig.toml --region us-east-1
                            sam deploy --stack-name todo-list-aws-production --capabilities "CAPABILITY_IAM" --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-nqyfysup146y --s3-prefix todo-list-aws --region us-east-1 --parameter-overrides Stage="production"
                        '''
                    }
                }
            }
        }
}
}
