pipeline {
    agent any
    
    stages {       
        stage('Build') {
            steps {
                echo 'NO HAY NADA QUE COMPILAR'
                sh 'ls'
            }
        }
        
        stage('wiremock') {
            steps {
                sh 'mkdir -p downloads'
                fileOperations([
                    fileDownloadOperation(userName: '', password: '', proxyHost: '', proxyPort: '', targetFileName: 'wiremock.jar', url: 'https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.10.0/wiremock-standalone-3.10.0.jar', targetLocation: 'downloads/',)
                ])
                sh '''
                    java -jar downloads/wiremock.jar --port 9090 --root-dir test/wiremock &
                '''
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            sh '''
                                export PYTHONPATH=$WORKSPACE
                                pytest --junitxml=result-unit.xml $WORKSPACE/test/unit
                            '''
                        }
                    }
                }
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            sh '''
                                export FLASK_APP=$WORKSPACE/app/api.py
                                flask run &
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Result'){
            steps {
                junit 'result*.xml'
            }
        }
    }
}
