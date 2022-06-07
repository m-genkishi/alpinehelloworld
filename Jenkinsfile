pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "genkishi-staging"
        PRODUCTION = "genkishi-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t genkishi/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run Container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run -d -p 80:5000 -e PORT=5000 --name $IMAGE_NAME genkishi/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''

                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        curl http://172.18.0.2 | grep -q "Hello world!"
                    '''

                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''

                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_genkishi')
            }
            steps {
                script {
                    sh '''
                        curl https://cli-assets.heroku.com/install.sh | sh
                        heroku container:login
                        heroku create $STAGING || echo "Project already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''

                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_genkishi')
            }
            steps {
                script {
                    sh '''
                        curl https://cli-assets.heroku.com/install.sh | sh
                        heroku container:login
                        heroku create $PRODUCTION || echo "Project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''

                }
            }
        }
    }
}