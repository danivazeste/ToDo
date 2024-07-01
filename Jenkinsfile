pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                // Obtener código del repositorio
                git 'https://github.com/danivazeste/hellounir.git'
            }
        }

	stage('Static') {
            steps {
		catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE'){
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
		    '''
		recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates : [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unhealthy: true]]}
            }
        }

	stage('Security') {
            steps {
		catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''
		recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],  qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true],  [threshold: 4, type: 'TOTAL', unhealthy: true]]}
            }
        }

        stage('Unit') {
            steps {
                //Test UNIT
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                           set PYTHONPATH=%WORKSPACE%
                           pytest --junitxml=result-unit.xml test/unit
                        '''
                }
            }
        }
        
        stage('Rest') {
            steps {
        	//Test REST con lanzamiento de microservicios
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                     bat '''
                            set FLASK_APP=app\\api.py
                            start flask run
                            start java -jar C:\\Users\\yog19\\Desktop\\CP1UNIR\\software\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir test\\wiremock
                            
                            set PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-rest.xml test/rest
                         '''
                        }
                    }
                }

	stage('Cobertura (Coverage)') {
	   steps {
	       bat '''
		      coverage run --branch --source=app --omit=app\\__init.py__,app\\api.py -m pytest test\\unit
		      coverage report
		      coverage xml -o coverage.xml
		   '''
	       catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
		      cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '90,85,80', lineCoverageTargets: '95,85,80', onlyStable: false
	       }
	   }
	}

	stage('Perfomance') {
            steps {
		// JMeter
                bat 'C:\\Users\\yog19\\Desktop\\CP1UNIR\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
		perfReport sourceDataFiles : 'flask.jtl'
            }
        }
    
    }
}
