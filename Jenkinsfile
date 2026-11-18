
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'midterm-api'
        CONTAINER_NAME = 'midterm-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install --no-cache-dir -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                    GIT_COMMIT="$GIT_COMMIT" .venv/bin/python train.py
                '''
            }
        }

        stage('Test Training Output') {
            steps {
                sh '''
                    .venv/bin/python -c "
import json, joblib
m = json.load(open('artifacts/metrics.json'))
required = ['student_id', 'accuracy', 'training_rows', 'model_version', 'git_sha']
assert all(k in m for k in required), 'Missing metrics'
assert m['student_id'] == 'FA23-BDS-018'
joblib.load('artifacts/model.joblib')
print('Training output checks passed')
"
                '''
            }
        }

        stage('Build Docker Image') {
    steps {
        sh '''
            docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} iberahimshoaib/midterm-api:latest
        '''
    }
}

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p 8000:8000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 5
                    curl --fail http://localhost:8000/health
                '''
            }
        }
    }

    post {
        success {
            echo 'Model trained and application deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
