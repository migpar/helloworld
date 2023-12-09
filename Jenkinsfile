pipeline{
    
    agent any
    
    stages{
        
        stage('Get Code'){
              steps{
                  // obtener el codigo del repo
                  // git 'https://github.com/migpar/helloworld.git'
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
                    steps {
                        bat '''
                            set FLASK_APP="%WORKSPACE%\\app\\api.py"
                            set FLASK_ENV=development
                            start /B flask run
                            start java -jar "C:\\Users\\miguel\\Desktop\\workspace\\helloworld\\test\\wiremock\\mappings\\wiremockstandalone\\wiremock-standalone-3.3.1.jar" --port 9090 --root-dir "C:\\Users\\miguel\\Desktop\\workspace\\helloworld\\test\\wiremock"
                            set PYTHONPATH=%WORKSPACE%
                            C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\pytest --junitxml=result-rest.xml test\\rest
                        '''
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


    