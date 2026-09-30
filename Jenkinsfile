pipeline {
    agent {
        label 'docker-builder'
    }

    environment {
        PROJECT_ID = 'fall-cloud-computing-2026'
        REGION = 'us-east1'
        REPOSITORY = 'jenkins-docker'
        IMAGE = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/portfolio-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Build and Push Image') {
            steps {

                container('gcloud-builder') {
                    sh '''
                        echo "Creating Artifact Registry authentication..."

                        ACCESS_TOKEN=$(gcloud auth print-access-token)

                        AUTH=$(printf 'oauth2accesstoken:%s' "$ACCESS_TOKEN" | base64 -w 0)

                        mkdir -p /kaniko/.docker

                        cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "https://us-east1-docker.pkg.dev": {
      "auth": "$AUTH"
    }
  }
}
EOF

                        echo "Artifact Registry authentication configured."
                    '''
                }

                container('kaniko') {
                    sh '''
                        echo "Building and pushing portfolio image..."

                        /kaniko/executor \
                          --context "${WORKSPACE}" \
                          --dockerfile "${WORKSPACE}/Dockerfile" \
                          --destination "${IMAGE}"
                    '''
                }
            }
        }

        stage('Deploy to GKE') {
            steps {

                container('gcloud-builder') {
                    sh '''
                        echo "Deploying portfolio to GKE..."

                        sed "s|IMAGE_PLACEHOLDER|${IMAGE}|g" \
                          k8s/deployment.yaml | kubectl apply -f -

                        kubectl apply -f k8s/service.yaml

                        echo "Waiting for deployment..."

                        kubectl rollout status \
                          deployment/portfolio-app \
                          -n portfolio \
                          --timeout=180s
                    '''
                }
            }
        }
    }
}
