pipeline {

    agent any

    environment {

        // Docker Hub 이미지
        IMAGE_NAME = "jeunju528/react-app:latest"

        // 배포 서버 디렉터리
        APP_DIR = "/home/sist/app"
    }

    stages {

        // =====================================================
        // 1. Git Checkout
        // =====================================================
        stage('Git Checkout') {

            steps {

                echo "===== Git Checkout ====="

                checkout scm
            }
        }
   
         stage('Copy Deploy Files') {

            steps {

                sh '''
                    echo "======================================"
                    echo " Docker Compose / Nginx 파일 복사"
                    echo "======================================"

                    mkdir -p ${APP_DIR}

                    cp docker-compose.yml ${APP_DIR}/docker-compose.yml

                    cp nginx.conf ${APP_DIR}/nginx.conf

                    echo "======================================"
                    echo " 배포 파일 확인"
                    echo "======================================"

                    ls -al ${APP_DIR}
                '''
            }
        }

        // =====================================================
        // 2. Gradle Build
        // =====================================================
        stage('Gradle Build') {

            steps {

                sh '''
                    echo "======================================"
                    echo " Gradle Build"
                    echo "======================================"

                    chmod +x gradlew

                    ./gradlew clean build -x test

                    echo "======================================"
                    echo " JAR 파일 확인"
                    echo "======================================"

                    ls -al build/libs
                '''
            }
        }


        // =====================================================
        // 3. Docker Build
        // =====================================================
        stage('Docker Build') {

            steps {

                sh '''
                    echo "======================================"
                    echo " Docker Build"
                    echo "======================================"

                    docker build \
                        -t ${IMAGE_NAME} .

                    echo "======================================"
                    echo " Docker Image 확인"
                    echo "======================================"

                    docker images | grep react-app
                '''
            }
        }


        // =====================================================
        // 4. Docker Hub Push
        // =====================================================
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
                        echo "======================================"
                        echo " Docker Hub Login"
                        echo "======================================"

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "======================================"
                        echo " Docker Hub Push"
                        echo "======================================"

                        docker push ${IMAGE_NAME}

                        echo "======================================"
                        echo " Docker Hub Logout"
                        echo "======================================"

                        docker logout
                    '''
                }
            }
        }


        // =====================================================
        // 5. .env 생성
        // =====================================================
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
                        echo "======================================"
                        echo " .env 생성"
                        echo "======================================"

                        cat > ${APP_DIR}/.env <<EOF
SPRING_PROFILES_ACTIVE=prod
DB_URL=${DB_URL}
DB_USERNAME=${DB_USERNAME}
DB_PASSWORD=${DB_PASSWORD}
EOF

                        chmod 644 ${APP_DIR}/.env

                        echo ".env 생성 완료"
                    '''
                }
            }
        }

        // =====================================================
        // 6. Docker Compose 배포
        // =====================================================
        stage('Rolling Deploy') {

            steps {

                sh '''
                    echo "======================================"
                    echo " 배포 디렉터리"
                    echo "======================================"

                    cd ${APP_DIR}

                    echo "현재 위치:"
                    pwd

                    echo "======================================"
                    echo " 파일 확인"
                    echo "======================================"

                    ls -al

                    echo "======================================"
                    echo " Docker Compose 설정 확인"
                    echo "======================================"

                    docker compose config

                    echo "======================================"
                    echo " Docker Image Pull"
                    echo "======================================"

                    docker pull ${IMAGE_NAME}

                    echo "======================================"
                    echo " Docker Compose 시작"
                    echo "======================================"

                    docker compose up -d --scale app=2

                    echo "======================================"
                    echo " 컨테이너 확인"
                    echo "======================================"

                    docker compose ps

                    echo "======================================"
                    echo " Health Check 대기"
                    echo "======================================"

                    sleep 30

                    echo "======================================"
                    echo " Health Check 결과"
                    echo "======================================"

                    docker compose ps

                    echo "======================================"
                    echo " Nginx Reload"
                    echo "======================================"

                    docker exec nginx nginx -s reload

                    echo "======================================"
                    echo " 배포 완료"
                    echo "======================================"
                '''
            }
        }
    }


    // =========================================================
    // Pipeline 결과
    // =========================================================
    post {

        success {

            echo '''
========================================
 Jenkins 배포 성공
========================================
'''
        }

        failure {

            echo '''
========================================
 Jenkins 배포 실패
========================================
'''
        }
    }
}