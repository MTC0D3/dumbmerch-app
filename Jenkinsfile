pipeline {
    agent any

    environment {
        BUILD_SERVER   = 'taofiks@103.175.221.143'
        DEPLOY_SERVER  = 'taofiks@103.175.221.143' // gunakan BUILD_SERVER jika sama
        SSH_PORT       = '22'
        BRANCH         = 'main'
        REPO_URL       = 'git@github.com:MTC0D3/dumbmerch-app.git'
        DIRECTORY      = '/home/tofiks/build/staging'
        REGISTRY_URL   = 'registry.taofik.studentdumbways.my.id'
        IMAGE_NAME     = 'dumbmerch-fe-staging'
    }

    stages {
        stage('Clean up Remote Workspace') {
            steps {
                script {
                    sshagent(credentials: ['keyssh']) {
                        sh """
                            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${BUILD_SERVER} << 'EOF'
                            rm -rf ${DIRECTORY}
                            mkdir -p ${DIRECTORY}/frontend
                            echo "Cleaned and recreated build directory."

                            docker rmi -f \$(docker images ${REGISTRY_URL}/${IMAGE_NAME} -q) || true
                            echo "Deleted old Docker images."
                            exit
                            EOF
                        """
                    }
                }
            }
        }

        stage('Pull Code and Build Docker Image') {
            steps {
                script {
                    sshagent(credentials: ['keyssh']) {
                        sh """
                            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${BUILD_SERVER} << 'EOF'
                            cd ${DIRECTORY}/frontend

                            git clone -b ${BRANCH} ${REPO_URL} .
                            echo "Pulled code from branch: ${BRANCH}"

                            version=\$(cat version)
                            echo "Using version: \${version}"

                            docker build -t "${REGISTRY_URL}/${IMAGE_NAME}:\${version}" .
                            docker tag "${REGISTRY_URL}/${IMAGE_NAME}:\${version}" "${REGISTRY_URL}/${IMAGE_NAME}:latest"

                            echo "Built image: \${version} and tagged as latest."
                            exit
                            EOF
                        """
                    }
                }
            }
        }

        stage('Test Docker Container (Local)') {
            steps {
                script {
                    sshagent(credentials: ['keyssh']) {
                        sh """
                            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${BUILD_SERVER} << 'EOF'
                            cd ${DIRECTORY}/frontend

                            version=\$(cat version)

                            docker run -d --name testcode-fe -p 3009:80 "${REGISTRY_URL}/${IMAGE_NAME}:\${version}"
                            echo "Started test container."

                            sleep 5

                            if wget --spider http://localhost:3009 >/dev/null 2>&1; then
                                echo "Website is up!"
                            else
                                echo "Website is down!"
                                docker rm -f testcode-fe
                                exit 1
                            fi

                            docker rm -f testcode-fe
                            echo "Test complete."
                            exit
                            EOF
                        """
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    sshagent(credentials: ['keyssh']) {
                        sh """
                            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${DEPLOY_SERVER} << 'EOF'
                            cd ${DIRECTORY}/frontend

                            docker stop frontend || true
                            docker rm -f frontend || true

                            docker run --pull always -d --name frontend -p 3000:80 "${REGISTRY_URL}/${IMAGE_NAME}:latest"
                            echo "Deployed frontend container on port 3000."
                            exit
                            EOF
                        """
                    }
                }
            }
        }
    }
}
