pipeline{
    agent {
        node {
            label 'Master'
        }
    }
    stages{
        stage('Get repo'){
            steps{
                echo 'access repo by private key ssh'
                git branch : 'master',
                    credentialsId : 'github-ssh',
                    url: 'git@github.com:r4kogama/helloworld.git'
            }
        }
        stage('Dependencies'){
            steps{
                echo 'Instalacion de dependencias...'
                bat 'python -m pip install --upgrade pytest pytest-html-plus'
                echo '...generador de informes HTML unificados instalado.'
            }
        }
        stage('Test'){
            parallel{
                stage('Unit_test'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat """
                                set PYTHONPATH=${WORKSPACE}
                                pytest test\\unit --junitxml=report/result-unit.xml --html-output=report/unit
                            """
                        }
                    }
                }
                stage('Service'){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat """
                                set FLASK_APP=app\\api.py
                                start /B flask run
                                ping -n 10 127.0.0.1 > nul
                                start /B java -jar C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\wiremock\\wiremock-standalone-3.13.2.jar --port 9090 --root-dir test\\wiremock
                                set PYTHONPATH=${WORKSPACE}
                                pytest test\\rest --junitxml=report/result-rest.xml --html-output=report/rest
                            """ 
                        }
                    }
                }
            }
        }
        stage('Result'){
            steps{
                junit testResults: 'report/result-*.xml', allowEmptyResults: true
                archiveArtifacts artifacts: 'report/**/*', allowEmptyArchive: true
            }
        }
    }
}