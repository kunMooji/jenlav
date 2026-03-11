pipeline {
    agent any

    environment {
        PHP_IMAGE = 'shippingdocker/php-composer:7.4'
        APP_ENV = 'development'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build / Install Dependencies') {
            steps {
                script {
                    docker.image(env.PHP_IMAGE).inside('-u root') {
                        sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image(env.PHP_IMAGE).inside('-u root') {
                        sh './vendor/bin/phpunit'
                    }
                }
            }
        }

        stage('Database Migration') {
            steps {
                script {
                    docker.image(env.PHP_IMAGE).inside('-u root') {
                        sh 'php artisan migrate --force'
                    }
                }
            }
        }

        stage('Cache Clear') {
            steps {
                script {
                    docker.image(env.PHP_IMAGE).inside('-u root') {
                        sh 'php artisan config:clear'
                        sh 'php artisan route:clear'
                        sh 'php artisan cache:clear'
                        sh 'php artisan view:clear'
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh 'echo "Deploying to development server..."'
            }
        }
    }

    post {
        always {
            echo 'Pipeline selesai!'
        }
        success {
            echo 'Build dan deploy berhasil.'
        }
        failure {
            echo 'Build gagal, cek log!'
        }
    }
}