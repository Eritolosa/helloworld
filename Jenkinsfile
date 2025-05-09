pipeline {
    agent any

    stages {
        stage('Get Code') {
            agent { label 'principal' }
            steps {
                echo 'Me traigo el Codigo'
                git branch: 'main', url: 'https://github.com/Eritolosa/helloworld.git'
                bat 'dir'
                echo "${WORKSPACE}"
                stash name: 'source', includes: '**/*'
            }
        }
        stage('Build') {
            agent { label 'principal' }
                steps {
                    echo 'No compilamos nada'
                }
            }
        
        stage('Test'){
            parallel{
                stage('Unit') {
                    agent { label 'linux1' }
                    steps {
                        echo 'Ejecutando pruebas unitarias'
                        unstash 'source'
                        bat '''
                        cd helloworld-master\\helloworld-master
                        set PYTHONPATH=.
                        pytest --junitxml=result-unit.xml test\\unit
                        '''
                        stash name: 'unit-results', includes: 'helloworld-master/helloworld-master/result-unit.xml'
                    }
                }
                stage('Rest') {
                    agent { label 'linux2' }
                    steps {
                        echo 'Ejecutando pruebas rest'
                        unstash 'source'
                        bat '''
                            cd helloworld-master\\helloworld-master
                            set FLASK_APP=app.api:api_application
                            set FLASK_ENV=development
                            start /B flask run
                            start /B java -jar C:\\Users\\tolos\\OneDrive\\Escritorio\\Devops\\REPOS\\helloworld-master\\helloworld-master\\test\\wiremock\\wiremock-standalone-3.13.0.jar --port 9090 --root-dir C:\\Users\\tolos\\OneDrive\\Escritorio\\Devops\\REPOS\\helloworld-master\\helloworld-master\\test\\wiremock
                            set PYTHONPATH=.
                            pytest --junitxml=result-rest.xml test\\unit
                        '''
                        stash name: 'rest-results', includes: 'helloworld-master/helloworld-master/result-rest.xml'
                    }
                }
            }
        }
        
        stage ('Results'){
            agent { label 'principal' }
            steps {
                unstash 'unit-results'
                unstash 'rest-results'
                junit 'helloworld-master/helloworld-master/result-*.xml'
            }
        }
}
}