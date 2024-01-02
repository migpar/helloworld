pipeline{
    
    agent any
    
    stages{
        
       
        stage('Get Code'){

            steps{
                // obtener el codigo del repo
                git 'https://github.com/migpar/helloworld.git'
            }
        }

        stage('Static'){
      
            steps {
                bat '''
                    C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\flake8 --exit-zero --format=pylint app > flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates : [[threshold: 10, type: 'TOTAL', unstable: true], [threshold: 11, type: 'TOTAL', unstable: false]]
            }
        }  


        stage('Unit') {

            steps {           
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                     C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\pytest --junitxml=result-unit.xml test\\unit
                '''
                junit 'result*.xml'
            
            }
        }

        stage('Cobertura'){

            steps{
                bat '''
                    C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\coverage run --branch --source=app --omit=app\\__init__.py.app\\api.py -m pytest test\\unit 
                    C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\coverage xml
                '''
                cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,80,90', lineCoverageTargets: '100,80,95', failUnstable: false
            }
        }

        stage('Security'){
            steps{
                bat '''
                    C:\\Users\\miguel\\AppData\\Local\\Programs\\Python\\Python312\\Scripts\\bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates : [[threshold: 1, type: 'TOTAL', unstable: true], [threshold: 2, type: 'TOTAL', unstable: false]]

            }
        }
    }
}