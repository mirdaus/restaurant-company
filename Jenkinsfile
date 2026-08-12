pipeline {
    agent any 
    
    tools {
        nodejs 'NodeJS'
    }

   
    environment {
        /*
         * Ubuntu Jenkins agent paths.
         */
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"

        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "230026708124"

        ECR_REPOSITORY = "restaurant-company"
        ECR_REGISTRY = "230026708124.dkr.ecr.us-east-1.amazonaws.com"
        ECR_IMAGE = "230026708124.dkr.ecr.us-east-1.amazonaws.com/restaurant-company"

        IMAGE_NAME = "restaurant-company"
        IMAGE_TAG = "${BUILD_NUMBER}"

        SONAR_PROJECT_KEY = "restaurant-company"
        SONAR_PROJECT_NAME = "restaurant-company"
    }

    stages {

        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm

                sh '''
                    set -e

                    echo "======================================"
                    echo "Checkout completed"
                    echo "Workspace: $WORKSPACE"
                    echo "Build number: $BUILD_NUMBER"
                    echo "Image tag: $IMAGE_TAG"
                    echo "======================================"

                    git log -1 --oneline
                    ls -la
                '''
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    sh """
                        set -e

                        echo "===== Node.js ====="
                        node --version

                        echo "===== npm ====="
                        npm --version

                        echo "===== Docker ====="
                        docker --version

                        echo "===== AWS CLI ====="
                        aws --version

                        echo "===== Trivy ====="
                        trivy --version

                        echo "===== SonarScanner ====="
                        ${scannerHome}/bin/sonar-scanner --version

                        echo "===== AWS Identity ====="
                        aws sts get-caller-identity

                        echo "===== Required Files ====="
                        test -f package.json
                        test -f Dockerfile

                        echo "All required tools and files are available."
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    if [ -f package-lock.json ]; then
                        echo "Installing dependencies with npm ci..."
                        npm ci
                    else
                        echo "package-lock.json not found."
                        echo "Installing dependencies with npm install..."
                        npm install
                    fi
                '''
            }
        }

        stage('Parallel Quality Checks') {
            failFast false

            parallel {

                stage('Lint') {
                    steps {
                        sh '''
                            set -e

                            echo "Checking for lint script..."

                            if node -e "
                                const p = require('./package.json');
                                process.exit(
                                    p.scripts && p.scripts.lint ? 0 : 1
                                );
                            "; then
                                echo "Running lint..."
                                npm run lint
                            else
                                echo "No lint script found. Skipping lint."
                            fi
                        '''
                    }
                }

                stage('Unit Tests') {
                    steps {
                        sh '''
                            set -e

                            echo "Checking for test script..."

                            if node -e "
                                const p = require('./package.json');
                                process.exit(
                                    p.scripts && p.scripts.test ? 0 : 1
                                );
                            "; then
                                echo "Running unit tests..."
                                CI=true npm test
                            else
                                echo "No test script found. Skipping tests."
                            fi
                        '''
                    }
                }

                stage('SonarQube Scan') {
                    steps {
                        script {
                            def scannerHome = tool 'SonarScanner'

                            withSonarQubeEnv('SonarQube') {
                                sh """
                                    set -e

                                    echo "Running SonarQube analysis..."

                                    ${scannerHome}/bin/sonar-scanner \
                                      -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                      -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                                      -Dsonar.projectVersion=${BUILD_NUMBER} \
                                      -Dsonar.sources=src \
                                      -Dsonar.sourceEncoding=UTF-8 \
                                      -Dsonar.exclusions=node_modules/**,dist/**,build/**,coverage/**
                                """
                            }
                        }
                    }
                }

                stage('Trivy Filesystem Scan') {
                    steps {
                        sh '''
                            set -e

                            echo "Running Trivy filesystem scan..."

                            trivy fs \
                              --severity HIGH,CRITICAL \
                              --ignore-unfixed \
                              --exit-code 0 \
                              --format table \
                              --output trivy-filesystem-report.txt \
                              .

                            echo "Trivy filesystem scan completed."

                            cat trivy-filesystem-report.txt || true
                        '''
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    set -e

                    echo "Building application..."
                    npm run build

                    if [ -d dist ]; then
                        echo "Build output:"
                        ls -lah dist
                    elif [ -d build ]; then
                        echo "Build output:"
                        ls -lah build
                    else
                        echo "Build command completed."
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "Building Docker image..."
                    echo "Image: $ECR_IMAGE:$IMAGE_TAG"

                    docker build \
                      --pull \
                      --tag $IMAGE_NAME:$IMAGE_TAG \
                      --tag $ECR_IMAGE:$IMAGE_TAG \
                      --tag $ECR_IMAGE:latest \
                      .

                    echo "Docker image built successfully."

                    docker image inspect \
                      $IMAGE_NAME:$IMAGE_TAG \
                      --format='Image ID: {{.Id}}'
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    set -e

                    echo "Scanning Docker image..."

                    trivy image \
                      --severity HIGH,CRITICAL \
                      --ignore-unfixed \
                      --exit-code 0 \
                      --format table \
                      --output trivy-image-report.txt \
                      $IMAGE_NAME:$IMAGE_TAG

                    echo "Trivy image scan completed."

                    cat trivy-image-report.txt || true
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    set -e

                    echo "Logging in to Amazon ECR..."

                    aws ecr get-login-password \
                      --region $AWS_REGION | \
                    docker login \
                      --username AWS \
                      --password-stdin $ECR_REGISTRY

                    echo "ECR login successful."
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    set -e

                    echo "Pushing versioned image..."
                    docker push $ECR_IMAGE:$IMAGE_TAG

                    echo "Pushing latest image..."
                    docker push $ECR_IMAGE:latest

                    echo "Both image tags were pushed successfully."
                '''
            }
        }

        stage('Verify Image in ECR') {
            steps {
                sh '''
                    set -e

                    echo "Verifying image in Amazon ECR..."

                    aws ecr describe-images \
                      --region $AWS_REGION \
                      --repository-name $ECR_REPOSITORY \
                      --image-ids imageTag=$IMAGE_TAG

                    echo "Image verification completed."
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo """
                Image is ready for deployment:

                ${ECR_IMAGE}:${IMAGE_TAG}

                Deployment is not configured in this stage yet.
                """
            }
        }
    }

    post {
        success {
            echo """
            ==============================================
            PIPELINE SUCCEEDED

            Versioned image:
            ${ECR_IMAGE}:${IMAGE_TAG}

            Latest image:
            ${ECR_IMAGE}:latest
            ==============================================
            """
        }

        failure {
            echo """
            ==============================================
            PIPELINE FAILED

            Open Console Output and check the first failed
            stage for the original error.
            ==============================================
            """
        }

        always {
            archiveArtifacts(
                artifacts: 'trivy-*-report.txt',
                allowEmptyArchive: true
            )

            sh '''
                echo "Logging out from Amazon ECR..."
                docker logout $ECR_REGISTRY || true
            '''

            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true
            )
        }
    }
}
