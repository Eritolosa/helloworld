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
                git branch: 'main', url: 'https://github.com/Eritolosa/helloworld.git'
                bat 'dir'
                stash name: 'source', includes: '**/*'
                deleteDir()
            }
        }

        stage('Build') {
            agent { label 'principal' }
            steps {
                echo 'No compilamos nada'
                bat 'whoami'
                bat 'hostname'
                echo "${WORKSPACE}"
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
                            set PYTHONPATH=.
                            pytest --junitxml=result-unit.xml test\\unit
                        '''
                        stash name: 'unit-results', includes: 'result-unit.xml'
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
                            start /B java -jar test\\wiremock\\wiremock-standalone-3.13.0.jar --port 9090 --root-dir test\\wiremock
                            set PYTHONPATH=.
                            pytest --junitxml=result-rest.xml test\\rest
                        '''
                        stash name: 'rest-results', includes: 'result-rest.xml'
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
                junit 'result-*.xml'
                deleteDir()
            }
        }
    }
}
