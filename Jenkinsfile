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
        
        
        stage('build') {
            steps {
                echo 'Eyy, esto es python, No hay que compilar nada!!'
                echo WORKSPACE
                bat 'dir'
            }
        }

        stage('Test Stages') {
            parallel{

                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                    }
                }

                stage('Rest') {
                    agent{
                    label 'linux'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE' ){
                            sh '''
                                env FLASK_APP=${WORKSPACE}/app/api.py python -m flask run &
                                java -jar "${WORKSPACE}/test/wiremock/mappings/wiremockstandalone/wiremock-standalone-3.3.1.jar" --port 9090 --root-dir "${WORKSPACE}/test/wiremock" &
                                sleep 15
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }

            }
        }


        stage('Results'){
            steps {
                junit 'result*.xml'
            }
        }
    }
}