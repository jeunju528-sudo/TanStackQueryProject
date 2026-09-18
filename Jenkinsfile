pipeline {

    agent any

    environment {

        // Docker Hub 이미지
        IMAGE_NAME = "jeunju528/react-app:latest"

        // 서버 배포 디렉터리
        APP_DIR = "/home/sist/app"
    }

    stages {

        // =========================================
        // 1. Git Checkout
        // =========================================
        stage('Git Checkout') {

            steps {

                checkout scm
            }
        }


        // =========================================
        // 2. Gradle Build
        // =========================================
        stage('Gradle Build') {

            steps {

                sh '''
                    echo "===== Gradle Build ====="

                    chmod +x gradlew

                    ./gradlew clean build -x test

                    echo "===== JAR 확인 ====="

                    ls -al build/libs
                '''
            }
        }


        // =========================================
        // 3. Docker Build
        // =========================================
        stage('Docker Build') {

            steps {

                sh '''
                    echo "===== Docker Build ====="

                    docker build \
                        -t ${IMAGE_NAME} .

                    echo "===== Docker Image 확인 ====="

                    docker images | grep react-app
                '''
            }
        }


        // =========================================
        // 4. Docker Hub Push
        // =========================================
        stage('Docker Hub Push') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )

                ]) {

                    sh '''
                        echo "===== Docker Hub Login ====="

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "===== Docker Hub Push ====="

                        docker push ${IMAGE_NAME}

                        echo "===== Docker Hub Logout ====="

                        docker logout
                    '''
                }
            }
        }


        // =========================================
        // 5. Create .env
        // =========================================
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

                ]) {

                    sh '''
                    	echo "===== 배포 디렉터리 생성 ====="
                        mkdir -p ${APP_DIR}
                        
                        echo "===== .env 생성 ====="
                        cat > ${APP_DIR}/.env <<EOF
						SPRING_PROFILES_ACTIVE=prod
						DB_URL=${DB_URL}
						DB_USERNAME=${DB_USERNAME}
						DB_PASSWORD=${DB_PASSWORD}
						EOF
						
                        chmod 600 ${APP_DIR}/.env
                        echo "===== .env 생성 완료 ====="
                    '''
                }
            }
        }


        // =========================================
        // 6. Rolling Deploy
        // =========================================
        stage('Rolling Deploy') {

            steps {

                sh '''
                
                	echo "===== docker-compose.yml 파일 복사 ====="
                    cp docker-compose.yml ${APP_DIR}/
                    
                    echo "===== 배포 디렉터리 ====="
                    cd ${APP_DIR}

                    echo "===== 현재 위치 ====="

                    pwd

                    echo "===== 파일 확인 ====="

                    ls -al

                    echo "===== Docker Image Pull ====="

                    docker pull ${IMAGE_NAME}

                    echo "===== Docker Compose 시작 ====="

                    docker compose up -d --scale app=2

                    echo "===== 컨테이너 확인 ====="

                    docker compose ps

                    echo "===== Health Check 대기 ====="

                    sleep 30

                    echo "===== Health Check 결과 ====="

                    docker compose ps

                    echo "===== Nginx Reload ====="

                    docker exec nginx nginx -s reload

                    echo "===== 배포 완료 ====="
                '''
            }
        }
    }


    // =========================================
    // 결과
    // =========================================
    post {

        success {

            echo '======================================'
            echo ' Rolling deployment completed successfully.'
            echo '======================================'
        }

        failure {

            echo '======================================'
            echo ' Rolling deployment failed.'
            echo '======================================'
        }
    }
}