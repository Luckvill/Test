pipeline {
    agent any

    stages {
        stage('Crea el Webhooks en caso de que no exista') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'Borrar', variable: 'GITHUB_TOKEN')]) {
                        def existingWebhook = sh(
                            script: 'curl -s -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/repos/Luckvill/Test/hooks',
                            returnStdout: true).trim()
                        def URL = "http://" + sh(script: 'curl -s ifconfig.me', returnStdout: true).trim() + ":8080/ghprbhook/"
                        // Verifica si el webhook ya existe en el repo, si no lo crea
                        if (!existingWebhook.contains("$URL")) {
                        def payload = '{"name": "web", "active": true, "events": ["pull_request"], "config": {"url": "' + URL + '", "content_type": "json", "secret": "69420"}}'
                        sh """
                            curl -X POST \
                            -H "Authorization: token $GITHUB_TOKEN" \
                            -H "Accept: application/vnd.github.v3+json" \
                            -d '${payload}' \
                            https://api.github.com/repos/Luckvill/Test/hooks
                            """
                        } else {
                            echo 'El webhook ya existe.'
                        }
                    }
                }
            }
        }
        stage('Database Maintenance') {
            steps {
                script {
                    // Descargar la base de datos
                    sh 'wget -O Employees.db https://github.com/rubenixter/PROF-2023-Ejercicio4/raw/main/Employees.db'

                    // Hacer una copia de los datos actuales
                    sh 'sqlite3 Employees.db ".dump" > backup.sql'
                
                    // Cargar el nuevo esquema borrando la db
                    sh 'rm Employees.db'
                    // sh 'sqlite3 Employees.db < sqlite.sql'

                    // Restaurar los datos respaldados anteriormente
                    sh 'sqlite3 Employees.db < backup.sql'
                }
            }
        }
    }

    post {
        success {
            // Publicar un estado de éxito en GitHub
            script {
                currentBuild.result = 'SUCCESS'
                echo 'Database maintenance successful!'
               
            }
        }
        failure {
            // Publicar un estado de fallo en GitHub
            script {
                currentBuild.result = 'FAILURE'
                echo 'Database maintenance failed!'
                
            }
        }
    }
}
