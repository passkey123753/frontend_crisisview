pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/passkey123753/frontend_crisisview.git'
            }
        }
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh '''
                    docker run --rm --network crisisview_default \
                        -v $(pwd):/usr/src \
                        -w /usr/src \
                        sonarsource/sonar-scanner-cli:4.6 \
                        -Dsonar.projectKey=frontend_crisisview \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**,.next/**,coverage/** \
                        -Dsonar.host.url=http://crisis_sonarqube:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t crisis_frontend .'
            }
        }
        stage('Deliver') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag crisis_frontend $DOCKER_USER/crisis_frontend:latest
                        docker push $DOCKER_USER/crisis_frontend:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    docker stop crisis_frontend || true
                    docker rm crisis_frontend || true
                    docker run -d --name crisis_frontend --network crisisview_default \
                        -p 3000:3000 crisis_frontend
                '''
            }
        }
    }
}
