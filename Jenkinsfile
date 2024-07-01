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
}
}
