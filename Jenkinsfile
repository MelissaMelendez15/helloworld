pipeline {
    agent any

    stages {
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
                sh 'rm -rf $WORKSPACE/*'
            }
        }

        stage('Build') {
            steps {
                echo 'Etapa de build vacia por ahora.'
            }
        }
    }
}
