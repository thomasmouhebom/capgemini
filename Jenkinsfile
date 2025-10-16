pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "wp-test"
        WP_URL = "http://localhost:8085"
    }

    stages {

        stage('Clone repository') {
            steps {
                echo "=== Clonage du dépôt Git ==="
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/thomasmouhebom/capgemini.git']]
                ])
            }
        }

        stage('Deploy WordPress') {
            steps {
                echo "=== Lancement du site WordPress ==="
                sh '''
                    echo "🧹 Nettoyage d'anciens conteneurs..."
                    docker compose down || true
                    echo "🚀 Démarrage des conteneurs WordPress et MariaDB..."
                    docker compose up -d
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "=== Vérification de la disponibilité du site ==="
                script {
                    sleep 15  // on laisse le temps aux conteneurs de démarrer
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' ${WP_URL}", returnStdout: true).trim()
                    if (response == '200' || response == '302') {
                        echo "✅ Le site WordPress est accessible (code HTTP ${response})"
                    } else {
                        error("❌ Le site WordPress ne répond pas (code HTTP ${response})")
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Déploiement réussi ! Site disponible sur ${WP_URL}"
        }
        failure {
            echo "🚨 Le déploiement a échoué... "
        }
    }
}
