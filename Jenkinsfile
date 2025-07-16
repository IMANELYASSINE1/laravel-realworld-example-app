pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('mon_token_sonar')
    }

    stages {
        stage('Check Composer Installation') {
            steps {
                script {
                    // Check if Composer is installed globally
                    def composerInstalled = sh(script: 'command -v composer || true', returnStdout: true).trim()
                    
                    if (!composerInstalled) {
                        // Install Composer locally if not found
                        sh 'curl -sS https://getcomposer.org/installer | php'
                        env.COMPOSER_CMD = 'php composer.phar'
                    } else {
                        env.COMPOSER_CMD = 'composer'
                    }
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh '${COMPOSER_CMD} install --no-interaction --prefer-dist --optimize-autoloader'
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
                sh 'mkdir -p tests/logs'  // Ensure directory exists
                sh './vendor/bin/phpunit --log-junit tests/logs/junit.xml --coverage-clover tests/logs/coverage.xml'
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
                    sh """
                        vendor/bin/sonar-scanner \
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

        stage('Deliver') {
            steps {
                script {
                    if (fileExists('./jenkins/scripts/deliver.sh')) {
                        sh 'chmod +x ./jenkins/scripts/deliver.sh'
                        sh './jenkins/scripts/deliver.sh'
                    } else {
                        echo 'Warning: deliver.sh script not found. Delivery step skipped.'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after build
        }
    }
}
