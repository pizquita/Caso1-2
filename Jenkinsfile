pipeline {
    agent any
	options {
        skipDefaultCheckout true
    }
    stages {
        stage('Get Code') {
            steps {
                cleanWs() // Limpia el workspace
                bat 'echo %WORKSPACE%' // Imprime el directorio de trabajo actual
                git 'https://github.com/pizquita/Caso1-2.git'
                // Verifica los archivos descargados
                bat '''
                    dir
                    echo %WORKSPACE%
                '''
            }
        }
        stage('Unit and Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    //lanza coverage en las pruebas unitarias para poder utilizar pytest una única vez
                    bat '''
                        SET PYTHONPATH=%WORKSPACE%
                        coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit --junitxml=result-unit.xml
                        coverage xml
                    '''
                    junit 'result-unit.xml'
                }
                
                bat 'coverage report -m'
                cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,80,90', lineCoverageTargets: '100,85,95'
            }
        }
        stage('Static') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					bat 'flake8 --format=pylint --exit-zero app >flake8.out'
                    recordIssues(
                        tools: [flake8(name: 'Flake8', pattern: 'flake8.out')],
                        qualityGates: [
                            [threshold: 8, type: 'TOTAL', unstable: true],  // Marcar como UNSTABLE si hay 8 o más hallazgos
                            [threshold: 10, type: 'TOTAL']     // Marcar como UNHEALTHY si hay 10 o más hallazgos
                        ]
                    )
                }
            }
        }
        stage('Security'){
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat 'bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}'
                    recordIssues(
                        tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                        qualityGates: [
                            [threshold: 4, type: 'TOTAL', unstable: true],  
                            [threshold: 8, type: 'TOTAL']   
                        ]
                    )
                }
            }
        }
        stage('Performance')
        {
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                            SET FLASK_APP=app\\api.py
                            SET PYTHONPATH=%WORKSPACE%
                            start flask run
                            "C:\\Program Files\\apache-jmeter-5.6.3\\bin\\jmeter" -n -t "%WORKSPACE%\\test\\jmeter\\flask.jmx" -f -l flask.jtl
                        '''
                    perfReport sourceDataFiles: 'flask.jtl'
                }
            }
        }
    }
}