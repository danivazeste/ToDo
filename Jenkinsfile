pipeline {
        agent any

        stages{
        stage('Get Code') {
            steps {
                // Obtener código del repositorio
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/danivazeste/ToDo.git']]])
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
                            sam deploy --stack-name todo-list-aws --capabilities "CAPABILITY_IAM" --s3-bucket aws-sam-cli-managed-default-samclisourcebucket-ox3ehbwe02gi --s3-prefix todo-list-aws --region us-east-1 --parameter-overrides Stage="production"
                        '''
                    }
                }
            }
        }

        stage('Rest Test') {
            steps {
                script {
                    try {
                        // Ejecutar pruebas de integración con curl o pytest
                        dir('test/integration') {
                            sh 'pytest todoApiTest.py'
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }

}
}
