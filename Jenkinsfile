pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build / Install Dependencies') {
            steps {
                sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
            }
        }

        stage('Run Tests') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }

        stage('Database Migration') {
            steps {
                sh 'php artisan migrate --force'
            }
        }

        stage('Cache Clear') {
            steps {
                sh 'php artisan config:clear'
                sh 'php artisan route:clear'
                sh 'php artisan cache:clear'
                sh 'php artisan view:clear'
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