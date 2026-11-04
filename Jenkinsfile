pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-website'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        CONTAINER_NAME = 'my-website-ci'
    }

    stages {
        stage('Build image') {
            steps {
                sh 'docker build --tag "$IMAGE_NAME:$IMAGE_TAG" .'
            }
        }

        stage('Smoke test') {
            steps {
                sh '''
                    set -eu
                    docker rm --force "$CONTAINER_NAME" >/dev/null 2>&1 || true
                    docker run --detach --publish 18080:80 --name "$CONTAINER_NAME" "$IMAGE_NAME:$IMAGE_TAG"
                    trap 'docker rm --force "$CONTAINER_NAME" >/dev/null 2>&1 || true' EXIT
                    for attempt in 1 2 3 4 5; do
                        if curl --fail --silent --show-error http://127.0.0.1:18080/ >/dev/null; then
                            exit 0
                        fi
                        sleep 1
                    done
                    echo "Container did not serve the website on port 18080." >&2
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm --force "$CONTAINER_NAME" >/dev/null 2>&1 || true'
            sh 'docker image rm "$IMAGE_NAME:$IMAGE_TAG" >/dev/null 2>&1 || true'
        }
    }
}
