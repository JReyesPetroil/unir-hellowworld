pipeline {
    agent any
    stages {       
        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        pytest --junitxml=result-unit.xml $WORKSPACE/test/unit
                    '''
                    junit 'result*.xml'
                }
            }
        }
        stage('Rest') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export FLASK_APP=$WORKSPACE/app/api.py
                        flask run &
                        sleep(3)
                        pytest --junitxml=result-rest.xml test/rest
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
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 10, type: 'TOTAL', unstable: true], [threshold: 11, type: 'TOTAL', unstable: false]]
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
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true], [threshold: 2, type: 'TOTAL', unstable: false]]
                }
            }
        }
        stage('Performance'){
            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
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
        }
        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        python3 -m coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                        python3 -m coverage xml
                    '''
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,90'
                }
            }
        }
    }
}