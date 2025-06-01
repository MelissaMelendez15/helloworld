pipeline {
    agent any

    stages {
         stage('Instalar dependencias') {
           steps {
                sh 'pip3 install flask pytest requests --break-system-packages'
            }
        }

        stage('Verificar workspace') {
            steps {
                sh 'echo $WORKSPACE'
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

        stage('Service') {
            steps {
                sh 'curl -v http://localhost:5000/ || echo "Flask no disponible antes del test de servicio"'
                sh 'sleep 5'
                sh 'PYTHONPATH=. pytest test/rest --junitxml=results-service.xml'
            }
        }

        stage('Static') {
            steps {
                sh'flake8 --exit-zero --format=pylint app > flake8.out'

                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')],
                qualityGates: [
                    [threshold: 8, type: 'TOTAL', unstable: true],
                    [threshold: 10, type: 'TOTAL', failBuild: true]
                ]
            }
        }

        stage('Security') {
            steps {
                sh '''
                    bandit -r app -f custom -o bandit.out \
                    --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}" || true
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                qualityGates: [
                    [threshold: 2, type: 'TOTAL', unstable: true],
                    [threshold: 4, type: 'TOTAL', failBuild: true]
                ]
            }
        }

        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                       PYTHONPATH=. coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                       coverage xml -o coverage.xml
                    '''
                }
                cobertura coberturaReportFile: 'coverage.xml', 
                          lineCoverageTargets: '95,85',
                          conditionalCoverageTargets: '90,80'
            }

        }

        stage('Performance') {
            steps {
                echo 'Inciando prueba de rendimiento con JMeter...'
                sh 'jmeter -n -t test/jmeter/flask.jmx -f -l test/jmeter/flask.jtl'
                perfReport sourceDataFiles: 'test/jmeter/flask.jtl'
            }
        }

    }

    post {
        always {
            junit 'results-*.xml'
            cleanWs()
        }
    }
}
