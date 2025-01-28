pipeline {
    agent none
    options {
        skipDefaultCheckout true
    }
    stages {
        stage('Get Code') {
            agent { label 'Git' }
            steps {
                cleanWs()
                bat 'whoami'
                bat 'hostname'
                bat 'echo %WORKSPACE%'
                bat 'echo %WORKSPACE%'
                git 'https://github.com/pizquita/Caso1-2.git'
                bat 'dir && echo %WORKSPACE%'
                
                // Guarda el código descargado para futuras etapas
                stash name: 'source-code', includes: '**/*'
            }
        }
        stage('Tests and Analysis') {
            parallel {
                stage('Unit') {
                    agent { label 'Test' }
                    steps {
                        cleanWs()
                        bat 'whoami'
                        bat 'hostname'
                        bat 'echo %WORKSPACE%'
                        unstash 'source-code'  // Recupera el código del repositorio
                        
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                SET PYTHONPATH=%WORKSPACE%
                                coverage run --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit --junitxml=result-unit.xml
                                coverage xml
                            '''
                            
                            bat 'coverage report -m'
                        }
                        cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,90'
                        
                        // Guarda los resultados para futuras etapas
                        stash name: 'test-results', includes: 'result-unit.xml, coverage.xml'
                    }
                }
                stage('Static') {
                    agent { label 'Analysis' }
                    steps {
                        cleanWs()
                        bat 'whoami'
                        bat 'hostname'
                        bat 'echo %WORKSPACE%'
                        unstash 'source-code'
                        
                        bat 'flake8 --format=pylint --exit-zero app >flake8.out'
                        recordIssues(
                            tools: [flake8(name: 'Flake8', pattern: 'flake8.out')],
                            qualityGates: [
                                [threshold: 8, type: 'TOTAL', unstable: true],
                                [threshold: 10, type: 'TOTAL', failed: false]
                            ]
                        )
                    }
                }
                stage('Security') {
                    agent { label 'Test' }
                    steps {
                        cleanWs()
                        bat 'whoami'
                        bat 'hostname'
                        bat 'echo %WORKSPACE%'
                        unstash 'source-code'
                        
                        bat 'bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"'
                        recordIssues(
                            tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                            qualityGates: [
                                [threshold: 4, type: 'TOTAL', unstable: true],
                                [threshold: 8, type: 'TOTAL', failed: false]
                            ]
                        )
                    }
                }
            }
        }
        stage('Performance') {
            agent { label 'Analysis' }
            steps {
                cleanWs()
                bat 'whoami'
                bat 'hostname'
                bat 'echo %WORKSPACE%'
                unstash 'source-code'  // Recupera el código descargado
                
                bat '''
                    SET FLASK_APP=app\\api.py
                    SET PYTHONPATH=%WORKSPACE%
                    start flask run
                    "C:\\Program Files\\apache-jmeter-5.6.3\\bin\\jmeter" -n -t "%WORKSPACE%\\test\\jmeter\\flask.jmx" -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
        stage('Reports') {
            agent { label 'Git' }
            steps {
                cleanWs()
                bat 'whoami'
                bat 'hostname'
                bat 'echo %WORKSPACE%'
                unstash 'test-results'  // Recupera los resultados de las pruebas
                
                junit 'result-unit.xml'
                cobertura coberturaReportFile: 'coverage.xml'
            }
            post {
                always {
                   cleanWs()
                }
            }
        }
    }
}
