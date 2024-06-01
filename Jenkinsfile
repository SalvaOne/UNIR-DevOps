pipeline {
    agent any
    
    stages {
        stage('Get Code') {
            steps {
                script {
                    //Verificacion de que existe o no un repositorio en este proyecto
                    def repoExists = fileExists("${env.WORKSPACE}/.git")
                    
                    //Si no existe, lo descargamos
                    if (!repoExists) {
                        git 'https://github.com/SalvaOne/UNIR-DevOps.git'
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
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        
                        python3 -m pytest --junitxml=result-unit.xml test/unit
                    '''
                }
            }
            
        }
        stage('Graphic') {
            steps {
                junit 'result-unit.xml'
                echo 'Test terminado y gráfica creada/actualizada'
            }
        }
    }
}
