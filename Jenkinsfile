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
                sh 'PYTHONPATH=. nohup python3 app/calc.py &'
                sh 'sleep 5'
            }
        }

        stage('Iniciar Wiremock') {
            steps {
                echo 'Iniciando Wiremock...'
                sh 'ls -la test/wiremock'
                sh 'nohup java -jar test/wiremock/wiremock-standalone-2.27.2.jar --port 8081 --root-dir test/wiremock &'
                sh 'sleep 10'
                echo 'Comprobando si Wiremock responde...'
                sh 'curl -v http://localhost:8081/__admin || echo "Wiremock no respondió"'
            }
        }
        
        stage('Unit') {
            steps {
                sh 'PYTHONPATH=. pytest test/unit --junitxml=result.xml'
            }
        }
    }
}
