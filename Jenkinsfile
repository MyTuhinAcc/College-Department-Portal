pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "tuhinnew/college-department"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                echo '========================================'
                echo 'CLONING CODE FROM GITHUB'
                echo '========================================'

                git branch: 'main',
                    url: 'https://github.com/MyTuhinAcc/College-Department-Portal.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '========================================'
                echo 'BUILDING DOCKER IMAGE'
                echo '========================================'

                bat """
                    docker build --pull=false -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                    docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
                """
            }
        }

        stage('Push Image') {
            steps {

                echo '========================================'
                echo 'PUSHING IMAGE TO DOCKER HUB'
                echo '========================================'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {

                    bat 'docker login -u %USER% -p %PASS%'

                    bat """
                        docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                        docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                echo '========================================'
                echo 'DEPLOYING TO KUBERNETES'
                echo '========================================'

                withCredentials([
                    file(
                        credentialsId: 'c2a42b70-67d4-4228-a0e4-5aa853a8f9fb',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    bat """
                        set KUBECONFIG=%KUBECONFIG%

                        echo Checking Kubernetes cluster...
                        kubectl config current-context

                        echo Applying Kubernetes configuration...
                        kubectl apply -f deployment.yaml --validate=false

                        echo Updating deployment to new Docker image...
                        kubectl set image deployment/college-department college-department=%DOCKER_IMAGE%:%DOCKER_TAG%

                        echo Waiting for rollout...
                        kubectl rollout status deployment/college-department

                        echo ========================================
                        echo DEPLOYMENT STATUS
                        echo ========================================

                        kubectl get deployment college-department

                        echo ========================================
                        echo POD STATUS
                        echo ========================================

                        kubectl get pods -l app=college-department

                        echo ========================================
                        echo SERVICE STATUS
                        echo ========================================

                        kubectl get service college-department-service

                        echo ========================================
                        echo IMAGE USED
                        echo ========================================

                        kubectl get deployment college-department -o jsonpath="{.spec.template.spec.containers[0].image}"

                        echo.
                    """
                }
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'COLLEGE DEPARTMENT DEPLOYMENT SUCCESSFUL'
            echo '========================================'

            echo "Docker Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo 'Replicas: 3'
            echo 'NodePort: 30081'

            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
        }
    }
}