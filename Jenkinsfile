pipeline {
    agent any

    environment {
        APP_IMAGE = 'php:7.4-cli'    // image PHP untuk composer
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    docker.image(APP_IMAGE).inside('-u root') {
                        sh '''
                            composer install --no-interaction --prefer-dist --optimize-autoloader
                        '''
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image(APP_IMAGE).inside('-u root') {
                        sh 'php artisan test'
                    }
                }
            }
        }

        stage('Database Migration') {
            steps {
                script {
                    docker.image(APP_IMAGE).inside('-u root') {
                        sh 'php artisan migrate --force'
                    }
                }
            }
        }

        stage('Cache Clear') {
            steps {
                script {
                    docker.image(APP_IMAGE).inside('-u root') {
                        sh 'php artisan cache:clear'
                        sh 'php artisan config:clear'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline selesai!'
        }
        success {
            echo 'Build berhasil!'
        }
        failure {
            echo 'Build gagal, cek log!'
        }
    }
}