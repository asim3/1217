pipeline {
    agent any

    environment {
        DOCKER_HUB = credentials('DOCKER_HUB')
        DOCKER_REGISTRY = "registry.test.asimt.sa"
        PROJECT_NAME = "1217"
        IMAGE_NAME = 'father_1217'
        IMAGE_VERSION = "v0.${BUILD_ID}"
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        // timeout on whole pipeline job
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION} .'
            }
        }


        stage('Tests') {
            options {
                timeout(time: 1, unit: 'MINUTES')
            }
            steps {
                sh 'docker container run --rm ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}'
            }
        }


        stage('Release') {
            steps {
                sh 'echo ${DOCKER_HUB_PSW} | docker login -u ${DOCKER_HUB_USR} --password-stdin'
                sh 'docker push     ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}'
                sh 'docker image rm ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}'
                sh 'docker pull     ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}'
            }
        }


        stage('Deploy') {
            steps {
              sh '''
cat <<EOF | docker stack deploy -c - stack-${PROJECT_NAME}
version: "3.8"
services:
    stack-${PROJECT_NAME}:
        image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_VERSION}
        deploy:
            mode: replicated
            replicas: 3
            labels:
                - traefik.enable=true
                - traefik.http.routers.${PROJECT_NAME}-app.rule=Host(\\`${PROJECT_NAME}.test.asimt.sa\\`)
                - traefik.http.routers.${PROJECT_NAME}-app.tls=true
                - traefik.http.services.${PROJECT_NAME}-svc.loadbalancer.server.port=80
            resources:
                reservations:
                    memory: 128M
                limits:
                    memory: 2048M
        networks:
            - main-public
networks:
    main-public:
        external: true
EOF
              '''
            }
        }


        stage('Done') { steps { echo "Done." } }
    }


    post {
        always {
            sh 'docker logout'
        }

        failure {
            mail cc: 'cc@mail.com', from: 'me@mail.com', to: 'to@mail.com', subject: 'My Email Subject', body: """
            My Body 
            Line one
            Line 222
            Report @ ${RUN_DISPLAY_URL}
            """
        }

        success {
            echo "OK from success."
        }
    }
}
