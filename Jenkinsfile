pipeline {
    agent {
        docker {
            image 'php:8.2-cli'  // Utilise une image Docker avec PHP pré-installé
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SONAR_TOKEN = credentials('mon_token_sonar')
        COMPOSER_ALLOW_SUPERUSER = 1
    }

    stages {
        stage('Installer Composer') {
            steps {
                sh '''
                    curl -sS https://getcomposer.org/installer -o composer-setup.php
                    php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                    composer --version
                '''
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
