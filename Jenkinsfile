pipeline {
    agent any

    stages {
         stage('Instalar dependencias') {
           steps {
                sh 'pip3 install flask pytest requests --break-system-packages'
            }
        }
         stage('Echo') {
            steps {
                echo '¡Hola desde Jenkins! Primera etapa OK'
            }
        }

        stage('Clonar repositorio') {
            steps {
                git branch: 'master', url: 'https://github.com/MelissaMelendez15/helloworld.git'
                sh 'ls -la'
            }
        }

        stage('Verificar workspace') {
            steps {
                sh 'echo $WORKSPACE'
            }
        }

        stage('Build') {
            steps {
                echo 'Etapa de build vacia por ahora.'
            }
        }

        stage('Iniciar Flask') {
            steps {
                sh 'PYTHONPATH=. nohup python3 app/api.py &'
                sh 'sleep 5'
                sh 'curl -v http://localhost:5000/ || echo "Flask no respondió"'
            }
        }

        stage('Iniciar Wiremock') {
            steps {
                echo 'Iniciando Wiremock...'
                sh '''
                   mkdir -p test/wiremock
                   if [ ! -f test/wiremock/wiremock-standalone-2.27.2.jar ]; then
                    echo "Descargando Wiremock JAR..."
                    curl -L https://repo1.maven.org/maven2/com/github/tomakehurst/wiremock-standalone/2.27.2/wiremock-standalone-2.27.2.jar \
                       -o test/wiremock/wiremock-standalone-2.27.2.jar
                   fi
                '''
                sh 'nohup java -jar test/wiremock/wiremock-standalone-2.27.2.jar --port 9090 --root-dir test/wiremock &'
                sh 'sleep 10'
                echo 'Comprobando si Wiremock responde...'
                sh 'curl -v http://localhost:9090/__admin || echo "Wiremock no respondió"'
            }
        }

        stage('Tests en paralelo') {
            parallel {
                stage('Unit') {
                    steps {
                        sh 'PYTHONPATH=. pytest test/unit --junitxml=result.xml'
                    }
                }

                stage('Service') {
                    steps {
                       sh 'curl -v http://localhost:5000/ || echo "Flask no disponible antes del test de servicio"'
                       sh 'sleep 5'
                       sh 'PYTHONPATH=. pytest test/rest --junitxml=results-service.xml'
                    }
                }
            }
        }
    }

    post {
        always {
            junit 'results-*.xml'
            junit 'results-unit.xml'
            junit 'results-service.xml'
        }
    }
}
