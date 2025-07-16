pipeline {
    agent any

    tools {
    }

    environment {
        SONAR_TOKEN = credentials('mon_token_sonar')
    }

    stages {
        stage('Install dependencies') {
            steps {
                sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
            }
        }

        stage('Prepare Laravel') {
            steps {
                sh '''
                    cp .env.example .env
                    php artisan key:generate
                '''
            }
        }

        stage('Run tests') {
            steps {
                sh './vendor/bin/phpunit --log-junit tests/logs/junit.xml'
            }
            post {
                always {
                    junit 'tests/logs/junit.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarServer') {
                    sh '''
                        vendor/bin/sonar-scanner \
                          -Dsonar.projectKey=laravel-realworld-example-app \
                          -Dsonar.sources=app \
                          -Dsonar.tests=tests \
                          -Dsonar.php.coverage.reportPaths=tests/logs/coverage.xml \
                          -Dsonar.host.url=http://172.17.0.3:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
    }
}
