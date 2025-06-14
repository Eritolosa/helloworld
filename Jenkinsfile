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
                            set PYTHONPATH=.
                            pytest --junitxml=test\\unit\\result-unit.xml test\\unit
                        '''
                        stash name: 'unit-results', includes: 'test/unit/result-unit.xml'
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
                junit 'test/**/result-*.xml'
                deleteDir()
            }
        }
        stage('Staticas') {
            steps {
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
                '''
                recordIssues (
                    tools: [flake8(name:'flake8', pattern: 'flake8.out')], 
                    qualityGates: [
                        [threshold: 10, type: 'TOTAL', unstable: false],
                        [threshold: 8, type: 'TOTAL', unstable: true]
                    ]
                )
            }
        }

        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}:[{test_id}] {msg}"
                '''
                recordIssues (
                    tools: [pyLint(name:'Bandit', pattern: 'bandit.out')], 
                    qualityGates: [
                    [threshold: 2, type: 'TOTAL', unstable: true],
                    [threshold: 4, type: 'TOTAL', unstable: false]
                ]
                )
                }
            }
        }

        stage('Performance') {
            steps {
                bat '"C:\\Users\\tolos\\OneDrive\\Escritorio\\Devops\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -n -t test\\jmeter\\flask.jmx -f -l flask.jtl'
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
        /*stage('Coverage') {
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
        }*/
    }
}