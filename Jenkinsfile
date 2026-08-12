pipeline {
    agent any  {
     
    }

    environment {
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:${env.PATH}"

        IMAGE_NAME = "restaurant-company"
        IMAGE_TAG = "latest"

        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "484908302072"
        ECR_REPOSITORY = "restaurant-company"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== Environment ====="
                    echo "PATH=$PATH"

                    echo ""
                    echo "===== Node ====="
                    node -v

                    echo ""
                    echo "===== NPM ====="
                    npm -v

                    echo ""
                    echo "===== Docker ====="
                    docker --version

                    echo ""
                    echo "===== AWS CLI ====="
                    aws --version

                    echo ""
                    echo "===== Trivy ====="
                    trivy --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {

                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {

                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=restaurant-company \
                        -Dsonar.projectName=restaurant-company \
                        -Dsonar.sources=src \
                        -Dsonar.sourceEncoding=UTF-8
                        """

                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --format table \
                    --output trivy-report.txt \
                    --no-progress \
                    ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr'
                ]]) {

                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag \
                    ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

    }

    post {

        success {
            echo "========================================"
            echo "Pipeline completed successfully!"
            echo "SonarQube scan completed."
            echo "Trivy scan completed."
            echo "Docker image pushed to Amazon ECR."
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "Pipeline failed."
            echo "========================================"
        }

        always {
            archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
            cleanWs()
        }
    }
}
[7:44 PM, 8/11/2026] Aisalkyn: which git
[8:16 PM, 8/11/2026] Aisalkyn: brew install trivy
[8:25 PM, 8/11/2026] Aisalkyn: docker exec -u root -it practical_wright bash
[8:26 PM, 8/11/2026] Aisalkyn: which node
which npm
which docker
which aws
which trivy
[8:27 PM, 8/11/2026] Aisalkyn: apt-get update
apt-get install -y nodejs npm
[8:43 PM, 8/11/2026] Aisalkyn: node -v
npm -v
[8:46 PM, 8/11/2026] Aisalkyn: docker inspect practical_wright --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
[8:50 PM, 8/11/2026] Aisalkyn: brew install jenkins-lts
[8:50 PM, 8/11/2026] Aisalkyn: brew services start jenkins-lts
[8:53 PM, 8/11/2026] Aisalkyn: pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:${env.PATH}"

        IMAGE_NAME = "restaurant-company"
        IMAGE_TAG = "latest"

        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "230026708124"
        ECR_REPOSITORY = "restaurant-company"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code from GitHub..."
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== Git ====="
                    git --version

                    echo "===== Node ====="
                    node -v

                    echo "===== NPM ====="
                    npm -v

                    echo "===== Docker ====="
                    docker --version

                    echo "===== AWS CLI ====="
                    aws --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing dependencies..."
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                echo "Building application..."
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag \
                    ${IMAGE_NAME}:${IMAGE_TAG} \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                    docker push \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "========================================"
            echo "PIPELINE SUCCESSFUL"
            echo "Application built successfully."
            echo "Docker image pushed to AWS ECR."
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "PIPELINE FAILED"
            echo "Check the Jenkins Console Output."
            echo "========================================"
        }
    }
}
