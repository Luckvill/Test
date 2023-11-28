pipeline {
    agent any
    stages {
        stage('Mantenimientos de la database') {
            steps {
                script {
                    // Se descarga la base de datos Employees.db
                    sh 'wget -O Employees.db https://github.com/Luckvill/PROF-2023-Ejercicio4/raw/main/Employees.db'
                    
                    // Se hace una copia de los datos actuales
                    sh 'sqlite3 Employees.db ".dump" > Backup.sql'

                    // Se elimina el esquema actual
                    sh 'rm Employees.db'
                    
                    // Se elimina el esquema actual y se carga el nuevo esquema
                    sh 'sqlite3 Employees.db < sqlite.sql'
                    
                    // Se Restauran los datos respaldados anteriormente
                    sh 'grep -E "INSERT" Backup.sql | sqlite3 Employees.db' 
                }
            }
        }

        stage('Crea el Webhook en caso de que no exista') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'TOKEN_REPO_PROFESOR', variable: 'GITHUB_TOKEN')]) {
                        def existingWebhook = sh(
                            script: 'curl -s -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/repos/GRISE-UPM/PROF-2023-Ejercicio4/hooks',
                            returnStdout: true).trim()
                        def URL = "http://" + sh(script: 'curl -s ifconfig.me', returnStdout: true).trim() + ":8080/ghprbhook/"
                        // Verifica si el webhook ya existe en el repo, si no lo crea
                        if (!existingWebhook.contains("$URL")) {
                        def payload = '{"name": "web", "active": true, "events": ["pull_request", "status"], "config": {"url": "' + URL + '", "content_type": "json", "secret": "1234"}}'
                        sh """
                            curl -X POST \
                            -H "Authorization: token $GITHUB_TOKEN" \
                            -H "Accept: application/vnd.github.v3+json" \
                            -d '${payload}' \
                            https://api.github.com/repos/GRISE-UPM/PROF-2023-Ejercicio4/hooks
                            """
                        } else {
                            echo 'El webhook ya existe.'
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                if (env.ghprbActualCommit != null) {
                    def pullRequestSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo pullRequestSHA
                    echo '1'
                    def status = '{"state": "success", "description": "Pull Request build was successful", "context": "Jenkins"}'
                    withCredentials([string(credentialsId: 'TOKEN_REPO_PROFESOR', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '${status}' \
                        https://api.github.com/repos/GRISE-UPM/PROF-2023-Ejercicio4/statuses/${pullRequestSHA}
                        """
                    }
                } else {
                    def commitSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo commitSHA
                    echo '2'
                    withCredentials([string(credentialsId: 'TOKEN_JENKINS', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '{"state": "success", "description": "Database maintenance successful", "context": "Jenkins"}' \
                        https://api.github.com/repos/Luckvill/PROF-2023-Ejercicio4/statuses/${commitSHA}
                        """
                    }
                }
            }
        }
        failure {
            script {
                if (env.ghprbActualCommit != null) {
                    def pullRequestSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    def status = '{"state": "failure", "description": "Pull Request build failed", "context": "Jenkins"}'
                    withCredentials([string(credentialsId: 'TOKEN_REPO_PROFESOR', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '${status}' \
                        https://api.github.com/repos/GRISE-UPM/PROF-2023-Ejercicio4/statuses/${pullRequestSHA}
                        """
                    }
                } else {
                    def commitSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    withCredentials([string(credentialsId: 'TOKEN_JENKINS', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '{"state": "failure", "description": "Database maintenance failed", "context": "Jenkins"}' \
                        https://api.github.com/repos/Luckvill/PROF-2023-Ejercicio4/statuses/${commitSHA}
                        """
                    }
                }
            }
        }
    }
}
