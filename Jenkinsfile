pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID = 'c8ce3edc-0522-48a3-b7e4-afe8e3d731d9'
        AZURE_TENANT_ID = 'bac4b78b-fcc2-4614-a32b-b69330b1af9f'
        CONTAINER_REGISTRY = 'goodacr.azurecr.io'
        RESOURCE_GROUP = 'AKS'
        REPO = 'kwujio/front'
        IMAGE_NAME = 'kwujio/front:latest'
        TAG_VERSION = "v1.0"
        TAG = "${TAG_VERSION}${env.BUILD_ID}"
        NAMESPACE = 'front'
        GIT_CREDENTIALS_ID = 'jenkins-git-access'
        GIT_REPO_URL = 'https://github.com/rlozi99/front-end'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image to ACR..') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'acr-credential-id', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_USERNAME')]) {
                        // Log in to ACR
                        sh "az acr login --name $CONTAINER_REGISTRY --username $ACR_USERNAME --password $ACR_PASSWORD"

                        // Build and push Docker image to ACR
                        // 변경: 이미지 이름을 $CONTAINER_REGISTRY/$IMAGE_NAME으로 수정
                        sh "docker build -t $CONTAINER_REGISTRY/$REPO:$TAG ."
                        sh "docker push $CONTAINER_REGISTRY/$REPO:$TAG"
                    }
                }
            }
        }

        stage('Update Kubernetes Configuration') {
            steps {
                script {
                    sh """
                    cd k8s
                    kustomize edit set image ${CONTAINER_REGISTRY}/${REPO}=${CONTAINER_REGISTRY}/${REPO}:${TAG}
                    """
                }
            }
        }

        stage('Commit and Push Changes to Git') {
            steps {
                script {
                    // 사용자 정보 설정
                    sh "git config user.email 'rlozi1999@gmail.com'"
                    sh "git config user.name 'rlozi99'"

                    // 변경사항 커밋
                    sh "git add k8s/*"
                    sh(script: "git commit -m 'Update image to ${TAG}' || echo 'No changes to commit'", returnStatus: true)

                    // 현재 브랜치 확인 및 main으로 체크아웃
                    def currentBranch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    if (currentBranch != "main") {
                        sh "git checkout main"
                    }

                    withCredentials([usernamePassword(credentialsId: 'jenkins-git-access', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        // 원격 저장소에서 최신 변경사항 가져오기
                        sh "git pull --rebase origin main"

                        // 원격 저장소 URL 구성
                        def remote = "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rlozi99/front-end.git"
                        
                        // 원격 저장소에 푸시
                        sh "git push ${remote} main"
                    }
                }
            }
        }
    }
}
