pipeline {
    agent any
    stages {
        stage('GetCode') {
            steps {
                git branch: 'develop', url: 'https://github.com/JReyesPetroil/unir-hellowworld.git'
            }
        }
        
        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        python3 -m pytest --junitxml=result-unit.xml $WORKSPACE/test/unit
                    '''
                    junit 'result*.xml'
                }
            }
        }
        stage('Rest') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        export FLASK_APP=$WORKSPACE/app/api.py
                        java -jar /home/jenkins/downloads/wiremock-standalone-3.10.0.jar --port 9090 --root-dir $WORKSPACE/test/wiremock &
                        python3 -m flask run &
                        sleep 10

                        python3 -m pytest test/rest --junitxml=result-rest.xml
                    '''
                }
            }
        }
        
        stage('Static') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        python3 -m flake8 --exit-zero --format=pylint app >flake8.out
                    '''
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 15, type: 'TOTAL', unstable: true], [threshold: 16, type: 'TOTAL', unstable: false]]
                }
            }
        }
        
        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        python3 -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}: {line}: [{test_id}] {msg}"
                    '''
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                }
            }
        }
        stage('Performance'){
            steps {
                sh '''
                    export FLASK_APP=app/api.py
                    flask run -p 5000 &
                    sleep 5
                    jmeter -n -t test/jmeter/flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        python3 -m coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                        python3 -m coverage xml
                    '''
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,95'
                }
            }
        }
    }
}