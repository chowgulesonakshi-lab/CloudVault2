
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'codecurious06/cloudvault-backend'
        KUBECONFIG = 'C:\\Users\\Sonakshi\\.kube\\config'
        DOCKER_PATH = 'C:\\Users\\Sonakshi\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin'
        K8S_NAMESPACE = 'cloudvault2'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/chowgulesonakshi-lab/CloudVault2.git'
            }
        }

        stage('Backend Test') {
            steps {
                dir('backend') {
                    bat 'python -m pip install -r requirements.txt'
                    bat 'python -m pytest -v'
                }
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    bat 'npm ci'
                    bat 'npm run build'
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat '''
                    set "PATH=%DOCKER_PATH%;%PATH%"
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% backend
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    bat '''
                        @echo off
                        set "PATH=%DOCKER_PATH%;%PATH%"
                        echo %DH_TOKEN%| docker login --username "%DH_USER%" --password-stdin
                        if errorlevel 1 exit /b 1

                        docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1

                        docker logout
                    '''
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                bat '''
                    set "PATH=%DOCKER_PATH%;%PATH%"
                    set "KUBECONFIG=%KUBECONFIG%"

                    kubectl config use-context docker-desktop

                    kubectl create namespace %K8S_NAMESPACE% --dry-run=client -o yaml | kubectl apply -f -
                    if errorlevel 1 exit /b 1

                    kubectl apply -n %K8S_NAMESPACE% -f k8s/postgres.yaml
                    if errorlevel 1 exit /b 1

                    kubectl apply -n %K8S_NAMESPACE% -f k8s/backend.yaml
                    if errorlevel 1 exit /b 1

                    kubectl apply -n %K8S_NAMESPACE% -f k8s/backend-service.yaml
                    if errorlevel 1 exit /b 1

                    kubectl set image deployment/cloudvault-backend backend=%DOCKER_IMAGE%:%BUILD_NUMBER% -n %K8S_NAMESPACE%
                    if errorlevel 1 exit /b 1

                    kubectl rollout status deployment/cloudvault-backend -n %K8S_NAMESPACE% --timeout=180s
                '''
            }
        }

        stage('Verify') {
            steps {
                bat '''
                    set "PATH=%DOCKER_PATH%;%PATH%"
                    set "KUBECONFIG=%KUBECONFIG%"

                    kubectl get pods -n %K8S_NAMESPACE%
                    kubectl get services -n %K8S_NAMESPACE%
                    kubectl get deployment cloudvault-backend -n %K8S_NAMESPACE%

                    kubectl get deployment cloudvault-backend -n %K8S_NAMESPACE% -o jsonpath="{.spec.template.spec.containers[0].image}"
                '''
            }
        }
    }

    post {
        success {
            echo 'CloudVault2 CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'CloudVault2 pipeline failed. Check the failed stage logs.'
        }
    }
}
