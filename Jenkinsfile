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
        
        stage('Test'){
            parallel{
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
                        set PYTHONPATH=.
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
                            cd test\\rest
                            set FLASK_APP=app.api:api_application
                            set FLASK_ENV=development
                            start /B flask run
                            cd ..\\wiremock
                            start /B java -jar wiremock-standalone-3.13.0.jar --port 9090 --root-dir .
                            cd ..\\rest
                            set PYTHONPATH=.
                            pytest --junitxml=test/rest/result-rest.xml
                            '''
                        stash name: 'rest-results', includes: 'test/rest/result-rest.xml'
                        deleteDir()
                    }
                }
            }
        }
        
        stage ('Results'){
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
}
}