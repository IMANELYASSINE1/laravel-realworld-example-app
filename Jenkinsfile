pipeline {
    agent any  // Utilise un agent standard au lieu de Docker

    environment {
        SONAR_TOKEN = credentials('mon_token_sonar')
        COMPOSER_ALLOW_SUPERUSER = 1
    }

    stages {
        stage('Vérifier les prérequis') {
            steps {
                script {
                    // Vérifie que PHP est installé
                    def phpInstalled = sh(script: 'command -v php || true', returnStdout: true).trim()
                    if (!phpInstalled) {
                        error 'PHP n\'est pas installé sur cet agent Jenkins. Veuillez installer PHP et les extensions requises.'
                    }
                    
                    // Installe Composer s'il n'existe pas
                    def composerInstalled = sh(script: 'command -v composer || true', returnStdout: true).trim()
                    if (!composerInstalled) {
                        sh '''
                            curl -sS https://getcomposer.org/installer -o composer-setup.php
                            php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                            rm composer-setup.php
                        '''
                    }
                }
            }
        }

        stage('Installer les dépendances') {
            steps {
                sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
            }
        }

        stage('Préparer Laravel') {
            steps {
                sh '''
                    cp .env.example .env
                    php artisan key:generate
                '''
            }
        }

        stage('Exécuter les tests') {
            steps {
                sh 'mkdir -p tests/logs'
                sh './vendor/bin/phpunit --log-junit tests/logs/junit.xml --coverage-clover tests/logs/coverage.xml'
            }
            post {
                always {
                    junit 'tests/logs/junit.xml'
                }
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarServer') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=laravel-realworld-example-app \
                          -Dsonar.sources=app \
                          -Dsonar.tests=tests \
                          -Dsonar.php.coverage.reportPaths=tests/logs/coverage.xml \
                          -Dsonar.host.url=http://172.17.0.3:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
