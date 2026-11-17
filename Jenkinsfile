
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'afnan2004/repo2'
        SONAR_SERVER_NAME     = 'sonarqube_env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/Afnan3311/repo2.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo "passed"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"
                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=topology-key \
                            -Dsonar.projectName='topology-project' \
                            -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Build & Post Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                            docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest -f Dockerfile .

                            docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {
                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    withCredentials([usernamePassword(
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        usernameVariable: 'GH_USER',
                        passwordVariable: 'GH_TOKEN'
                    )]) {
                        sh """
                            git remote set-url origin https://\${GH_USER}:\${GH_TOKEN}@github.com/Afnan3311/repo2.git

                            git pull --rebase origin ${MANIFEST_BRANCH}

                            sed -i '/name: maker/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' manifests/deployment.yaml

                            git config user.name "Afnan3311"
                            git config user.email "mohdafnan9617@gmail.com"

                            git commit -am "chore(manifest): update repo2 image tag to ${env.BUILD_NUMBER} [skip ci]"

                            git push origin ${MANIFEST_BRANCH}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Manifest repo updated with new image tag.'
        }
        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
