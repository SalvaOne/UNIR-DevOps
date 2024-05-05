pipeline {
    agent any
    
    stages {
        stage('Clean Ws') {
            steps {
                //Limpieza del ws antes de descargar
                echo 'Se va a proceder a limpiar el Ws, gracias por su paciencia'
                deleteDir()
            }
        }

        stage('Get Code') {
            steps {
                script {
                    //Verificacion de que existe o no un repositorio en este proyecto
                    def repoExists = fileExists("${env.WORKSPACE}/.git")
                    
                    //Si no existe, lo descargamos desde la rama develop, que contiene los datos actualizados
                    if (!repoExists) {
                        git branch: 'develop', url: 'https://github.com/SalvaOne/UNIR-DevOps'
                        stash (name: 'repo')
                    } else {
                        //Si ya existe
                        echo "El repositorio ya existe en la ubicacion ${env.WORKSPACE}"
                    }
                }
                
            }
        }
        
         stage('Test Unit') {
            steps {
                //Capturamos el error, y permitimos seguir al pipeline
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    echo 'Se va a proceder a realizar un test unitario, por favor espere'
                    unstash 'repo'
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        
                        python3 -m pytest --junitxml=result-unit1.xml test/unit
                    '''
                    stash includes: 'result-unit1.xml', name: 'results1'
                }
            }
            
        }
        
        stage('Test Rest') {
            agent { label 'wiremock' }
            steps {
                //Capturamos el error, y permitimos seguir al pipeline
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    unstash 'repo'
                    echo 'Se va a proceder a realizar un test con wiremock para funciones de la calculadora, por favor espere'
                    sh '''
                        export FLASK_APP=$WORKSPACE/app/api.py
                        flask run &
                        sleep 6
                        java -jar /usr/wiremock/wiremock-standalone-3.5.4.jar --port 9090 --root-dir /var/lib/jenkins/workspace/DevOps Unir 2024/Prueba_Unitaria/test/wiremock &
                        sleep 13
                    '''
                    sh "pwd"
                    sh 'python3 -m pytest --junitxml=result-unit.xml test/rest'
                    stash includes: 'result-unit.xml', name: 'results'
                    sh 'sleep 2'
                }
            }
            
        }
        stage('Graphic') {
            steps {
                unstash 'results'
                unstash 'results1'
                junit 'result-unit.xml'
                junit 'result-unit1.xml'
                echo 'Test terminado y gráficas creadas/actualizadas'
            }
        }
    }
}
