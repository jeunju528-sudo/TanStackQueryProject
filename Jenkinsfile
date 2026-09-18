pipeline {

    agent any

    environment {
        IMAGE_NAME = "react-app:latest"
        APP_DIR = "~/app"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Create .env') {
            steps {
                withCredentials([
					string(
						credentialsId: 'oracle_url',
						variable: 'DB_URL'
					),
					string(
						credentialsId: 'oracle_name',
						variable: 'DB_USERNAME'
					),
					string(
						credentialsId: 'oracle_pwd',
						variable: 'DB_PASSWORD'
					)
				]){
					sh '''
						echo "SPRING_PROFILES_ACTIVE=prod" > .env
						echo "LOCAL_DB_URL=${DB_URL}" >> .env
						echo "DB_URERNAME=${DB_URERNAME}" >> .env
						echo "DB_PASSWORD=${DB_PASSWORD}" >> .env
						
						chmod 600 .env
					   '''
				}
            }
        }
        
        stage('Gradle Build') {
            steps {
                sh '''
                    chmod +x gradlew
                    ./gradlew clean build -x test
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Rolling Deploy') {
            steps {
                sh '''
                    cd ${APP_DIR}

                    echo "===== 최신 이미지 확인 ====="
                    docker images react-app

                    echo "===== Rolling 배포 ====="

                    docker compose up -d \
                        --no-deps \
                        --scale app=2

                    echo "===== 컨테이너 확인 ====="
                    docker compose ps

                    echo "===== Health Check ====="

                    sleep 10

                    docker compose ps

                    echo "===== Nginx Reload ====="
                    docker exec nginx nginx -s reload

                    echo "===== 배포 완료 ====="
                '''
            }
        }
    }

    post {
        success {
            echo 'Rolling deployment completed successfully.'
        }

        failure {
            echo 'Rolling deployment failed.'
        }
    }
    
}
