pipeline {
    agent any

    stages {
        stage('Get Code') {
            agent { label 'principal' }
            steps {
                echo 'Me traigo el Codigo'
                bat 'whoami'
                bat 'hostname'
                echo "${WORKSPACE}"
                git branch: 'master', url: 'https://github.com/Eritolosa/helloworld.git'
                bat 'dir'
                stash name: 'source', includes: '**/*'
                deleteDir()
            }
        }
        stage('Test') {
            parallel {
                stage('Unit') {
                    agent { label 'linux1' }
                    steps {
                        echo 'Ejecutando pruebas unitarias'
                        bat 'whoami'
                        bat 'hostname'
                        echo "${WORKSPACE}"
                        unstash 'source'
                        bat '''
                            cd test\\unit
                            set PYTHONPATH=..\\..
                            pytest --junitxml=result-unit.xml
                        '''
                        stash name: 'unit-results', includes: 'test/unit/result-unit.xml'
                        deleteDir()
                    }
                }

                stage('Rest') {
                    agent { label 'linux2' }
                    steps {
                        echo 'Ejecutando pruebas rest'
                        bat 'whoami'
                        bat 'hostname'
                        echo "${WORKSPACE}"
                        unstash 'source'
                        bat '''
                            set FLASK_APP=app.api:api_application
                            set FLASK_ENV=development
                            start /B flask run
                            cd test\\wiremock
                            start /B java -jar wiremock-standalone-3.13.0.jar --port 9090 --root-dir .
                            cd ..\\rest
                            set PYTHONPATH=..\\..
                            pytest --junitxml=result-rest.xml
                        '''
                        stash name: 'rest-results', includes: 'test/rest/result-rest.xml'
                        deleteDir()
                    }
                }
            }
        }
        stage('Results') {
            agent { label 'principal' }
            steps {
                echo 'Recopilando resultados'
                bat 'whoami'
                bat 'hostname'
                echo "${WORKSPACE}"
                unstash 'unit-results'
                unstash 'rest-results'
                junit 'test/**/result-*.xml'
                deleteDir()
            }
        }
        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set PYTHONPATH=.
                        coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest test\\unit
                        coverage xml
                    '''
                    recordCoverage(tools:[[parser:'COBERTURA', pattern: 'coverage.xml']],
                                    qualityGates: [
                                        [threshold: 85.0, metric: 'LINE', baseline: 'PROJECT', criticality: 'UNSTABLE'],
                                        [threshold: 95.0, metric: 'LINE', baseline: 'PROJECT', criticality: 'SUCCESS'],
                                        [threshold: 80.0, metric: 'BRANCH', baseline: 'PROJECT', criticality: 'UNSTABLE'],
                                        [threshold: 90.0, metric: 'BRANCH', baseline: 'PROJECT', criticality: 'SUCCESS']
                                    ]
                                    )
                }
            }
        }
}
}