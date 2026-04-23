pipeline {
    agent any

    environment {
        ARGOCD_SERVER = "192.168.64.6:31705"
        APP_NAME      = "my-app"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "---- Cloning Public Repo ----"
                git branch: 'main',
                    url: 'https://github.com/vickey-k/test-argo.git'
            }
        }

        stage('Validate Manifests') {
            steps {
                echo "---- Validating YAML files ----"
                sh """
                    kubectl apply --dry-run=client \
                        -f k8s-manifests/apps/my-app/deployment.yaml
                    kubectl apply --dry-run=client \
                        -f k8s-manifests/apps/my-app/service.yaml
                    echo "Manifests are valid!"
                """
            }
        }

        stage('Sync via ArgoCD') {
            steps {
                echo "---- Triggering ArgoCD Sync ----"
                withCredentials([string(
                    credentialsId: 'ARGOCD_TOKEN',
                    variable: 'ARGOCD_AUTH_TOKEN'
                )]) {
                    sh """
                        argocd app sync ${APP_NAME} \
                            --server ${ARGOCD_SERVER} \
                            --auth-token \$ARGOCD_AUTH_TOKEN \
                            --insecure

                        argocd app wait ${APP_NAME} \
                            --server ${ARGOCD_SERVER} \
                            --auth-token \$ARGOCD_AUTH_TOKEN \
                            --insecure \
                            --health \
                            --timeout 120
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "---- Checking Pods ----"
                sh """
                    kubectl get pods -n default -l app=my-app
                    kubectl rollout status deployment/my-app -n default
                    echo "Deployment verified!"
                """
            }
        }
    }

    post {
        success {
            echo "SUCCESS — my-app is live!"
        }
        failure {
            echo "FAILED — Check ArgoCD UI or kubectl logs"
        }
    }
}
EOF

# Push to GitHub
git add Jenkinsfile
git commit -m "fix: change bat to sh for Linux Jenkins"
git push origin main
