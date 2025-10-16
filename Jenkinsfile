pipeline {
    agent any

    stages {
        stage('Clone repository') {
            steps {
                echo '=== Clonage du dépôt Git ==='
                checkout scm
            }
        }

        stage('Deploy WordPress') {
            steps {
                echo '=== Lancement du site WordPress ==='
                sh '''
                docker compose down || true
                docker compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo '=== Vérification de la disponibilité du site ==='
                script {
                    sleep 15 // laisse WordPress démarrer
                    def code = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8085", returnStdout: true).trim()
                    if (code != "200") {
                        error "❌ Le site WordPress ne répond pas (code HTTP ${code})"
                    } else {
                        echo "✅ WordPress est accessible sur http://localhost:8085"
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "🚨 Le déploiement a échoué."
        }
        success {
            echo "🎉 Déploiement réussi sur http://localhost:8085"
        }
    }
}
