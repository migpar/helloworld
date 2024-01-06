pipeline{
    
    agent any
    
    stages{
        
        stage('Get Codes') {
            parallel{
                stage('Get Code windows'){
                    agent{
                        label 'windows'
                    }
                    steps{
                        // obtener el codigo del repo
                        git 'https://github.com/migpar/helloworld.git'
                    }
                }   

                stage('Get Code linux'){
                    agent{
                        label 'linux'
                    }
                    steps{
                        // obtener el codigo del repo
                        git 'https://github.com/migpar/helloworld.git'
                    }
                } 
            }
        }  

        stage('Test Stages') {
            parallel{

                stage('Unit') {
                    agent{
                        label 'localWin'
                    }

                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                             
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                    }
                }

                stage('Static'){
                    agent {
                        label 'localWin'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                            bat '''
                                C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\flake8 --exit-zero --format=pylint app > flake8.out
                            '''
                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates : [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                        }
                    }
                }

                stage('Security Test'){
                    agent {
                        label 'localWin'
                    }
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''}
                                C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}
                            '''
                            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates : [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                        }
                    }
                }

                stage('Coverage'){
                    agent {
                        label 'localWin'
                    }
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                            bat '''
                                coverage run --branch --source=app --omit=app\\__init__.py.app\\api.py -m pytest test\\unit 
                                coverage xml
                            '''
                            cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,30,30', lineCoverageTargets: '100,40,40', failUnstable: false
                        }
                    }
                }

            }
        }

        stage('Rest') {
            agent{
            label 'raspi'
            }
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                    sh '''
                        set PYTHONPATH=%WORKSPACE%
                        env FLASK_APP=${WORKSPACE}/app/api.py python -m flask run &
                        java -jar "${WORKSPACE}/test/wiremock/mappings/wiremockstandalone/wiremock-standalone-3.3.1.jar" --port 9090 --root-dir "${WORKSPACE}/test/wiremock" &
                        sleep 10
                        pytest --junitxml=result-rest.xml test/rest
                    '''
                }
            }
        }

        stage('performance'){
            agent {
                label 'raspi'
            }
            steps{
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                    sh '''
                        set PYTHONPATH=%WORKSPACE%
                        env FLASK_APP=${WORKSPACE}/app/api.py python -m flask run &
                        sleep 10
                        /home/miguel/apache-jmeter-5.6.2/bin/jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl
                    '''
                    perfReport filterRegex: '', showTrendGraphs: true, sourceDataFiles: 'flask.jtl'
                }
            }
        }                

    }
}
