pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'afnan2004/aws-dashboard'

        SONAR_SERVER_NAME     = 'sonarqube_env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'

        GIT_CREDENTIALS_ID    = 'github-cred'

        // Replace with your new manifest repository URL
        MANIFEST_REPO         = 'https://github.com/Afnan3311/repo2.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                //checkout scm
                echo "passed"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=aws-dashboard-key \
                            -Dsonar.projectName=aws-dashboard-project \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${DOCKER_CREDENTIALS_ID}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login \
                            -u "\$DOCKER_USER" --password-stdin

                            docker build \
                            -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -t ${DOCKER_IMAGE}:latest \
                            -f Dockerfile .

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest

                            docker logout
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    // Checkout manifest repository
                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GIT_CREDENTIALS_ID}",
                            usernameVariable: 'GH_USER',
                            passwordVariable: 'GH_TOKEN'
                        )
                    ]) {
                        sh '''
                            # Update deployment image tag
                            IMAGE_TAG="${DOCKER_IMAGE}:${BUILD_NUMBER}"

                            sed -i -E \
                            "s|(^[[:space:]]*image:).*|\\1${IMAGE_TAG}|" \
                            manifests/deployment.yml

                            # Configure Git
                            git config user.name "Afnan331"
                            git config user.email "mohdafnan9617@gmail.com"

                            # Commit updated manifest
                            git add manifests/deployment.yml

                            if ! git diff --cached --quiet; then
                                git commit -m "Update AWS Dashboard image to ${BUILD_NUMBER} [skip ci]"

                                # Push changes to manifest repository
                                PUSH_URL="https://${GH_USER}:${GH_TOKEN}@${MANIFEST_REPO#https://}"

                                git push "$PUSH_URL" HEAD:${MANIFEST_BRANCH}
                            else
                                echo "No manifest changes detected"
                            fi
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! AWS Dashboard image pushed and manifest repository updated.'
        }

        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

