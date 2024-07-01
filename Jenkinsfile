pipeline {
        agent any

        stages{
        stage('Get Code') {
            steps {
                // Obtener código del repositorio
                cleanWs()
                git url: 'https://github.com/danivazeste/ToDo.git', branch: 'master'
            }
        }
}
}
