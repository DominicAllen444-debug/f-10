pipeline {
    agent any

    environment {
        NAMESPACE = 'default'
        GITHUB_REPO = 'https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git'
        KUBECONFIG_SOURCE = 'C:\\Users\\YOUR-WINDOWS-USER\\.kube\\config'
        KUBECONFIG_DEST = 'C:\\Windows\\System32\\config\\systemprofile\\.kube\\config'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPO}"
            }
        }

        stage('Refresh Kubernetes Connection') {
            steps {
                bat '''
                    for %%D in ("%KUBECONFIG_DEST%") do if not exist "%%~dpD" mkdir "%%~dpD"
                    copy /Y "%KUBECONFIG_SOURCE%" "%KUBECONFIG_DEST%"
                    if errorlevel 1 exit /b 1

                    kubectl get nodes
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Deploy Application with Readiness Probe') {
            steps {
                bat '''
                    if exist "k8s\\deployment.yaml" (
                        set "DEPLOY_FILE=k8s\\deployment.yaml"
                    ) else if exist "deployment.yaml" (
                        set "DEPLOY_FILE=deployment.yaml"
                    ) else (
                        echo ERROR: deployment.yaml not found in repository root or k8s folder.
                        dir
                        exit /b 1
                    )

                    kubectl apply -n %NAMESPACE% -f "%DEPLOY_FILE%"
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Wait for Application Readiness') {
            steps {
                bat '''
                    kubectl rollout status deployment/readiness-demo --namespace=%NAMESPACE% --timeout=180s
                    if errorlevel 1 (
                        echo Deployment did not become ready. Showing diagnostic details:
                        kubectl get pods -n %NAMESPACE% -o wide
                        kubectl describe deployment readiness-demo -n %NAMESPACE%
                        kubectl get events -n %NAMESPACE% --sort-by=.metadata.creationTimestamp
                        exit /b 1
                    )
                '''
            }
        }

        stage('Verify Readiness Probe') {
            steps {
                bat '''
                    kubectl get deployment readiness-demo -n %NAMESPACE% -o wide
                    kubectl get pods -n %NAMESPACE% -o wide
                    kubectl describe deployment readiness-demo -n %NAMESPACE%
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed and readiness probe verified successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output and Kubernetes events.'
        }
    }
}
