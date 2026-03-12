pipeline {
    agent any

    environment {
        COMPOSER_IMAGE = 'composer:2'
        PHP_IMAGE = 'php:8.2-cli'
        PROD_HOST = '192.168.172.39'
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
                    docker.image(COMPOSER_IMAGE).inside('-u root --entrypoint=') {
                        sh '''
                            composer install --no-interaction --prefer-dist --optimize-autoloader
                            cp .env.example .env
                            php artisan key:generate
                        '''
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image(COMPOSER_IMAGE).inside('-u root --entrypoint=') {
                        sh 'php artisan test'
                    }
                }
            }
        }

        stage('Database Migration') {
            steps {
                script {
                    docker.image('laravelphp/vapor:php82').inside('-u root --entrypoint= --network jenkins-net') {
                        sh '''
                            sed -i "s/DB_HOST=.*/DB_HOST=mysql-jenkins/" .env
                            sed -i "s/DB_DATABASE=.*/DB_DATABASE=jenlav/" .env
                            sed -i "s/DB_USERNAME=.*/DB_USERNAME=jenlav/" .env
                            sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=secret/" .env
                            php artisan migrate --force
                        '''
                    }
                }
            }
        }

        stage('Cache Clear') {
            steps {
                script {
                    docker.image('laravelphp/vapor:php82').inside('-u root --entrypoint= --network jenkins-net') {
                        sh '''
                            sed -i "s/DB_HOST=.*/DB_HOST=mysql-jenkins/" .env
                            sed -i "s/DB_DATABASE=.*/DB_DATABASE=jenlav/" .env
                            sed -i "s/DB_USERNAME=.*/DB_USERNAME=jenlav/" .env
                            sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=secret/" .env
                            php artisan cache:clear
                            php artisan config:clear
                        '''
                    }
                }
            }
        }

stage('Deploy to Production') {
    steps {
        script {
            sshagent(credentials: ['ssh-prod']) {
                docker.image('alpine').inside("-u root --entrypoint= -v ${SSH_AUTH_SOCK}:/ssh-agent -e SSH_AUTH_SOCK=/ssh-agent") {
                    sh '''
                        apk add --no-cache rsync openssh-client
                        mkdir -p /root/.ssh
                        ssh-keyscan -H ''' + PROD_HOST + ''' >> /root/.ssh/known_hosts
                        rsync -rav --delete ./ ubuntu@''' + PROD_HOST + ''':/var/www/html/ --exclude=.env --exclude=storage --exclude=.git
                    '''
                }
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