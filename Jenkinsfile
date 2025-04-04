pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'python-webapp-rg-02'
        APP_SERVICE_NAME = 'python-webapp-service-1408003'
        PYTHON_VERSION = '3.10'
        PYTHON_PATH = 'C:\\Users\\window 10\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/aditya-blanko/python-webapp.git'
            }
        }
        
        stage('Build') {
            steps {
                bat '''
                    "%PYTHON_PATH%" --version
                    "%PYTHON_PATH%" -m pip install --upgrade pip
                    "%PYTHON_PATH%" -m pip install -r requirements.txt
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    // Login to Azure
                    bat 'az login --service-principal -u "%AZURE_CLIENT_ID%" -p "%AZURE_CLIENT_SECRET%" --tenant "%AZURE_TENANT_ID%"'
                    
                    // Create resource group and verify
                    bat '''
                        az group create --name %RESOURCE_GROUP% --location eastus
                        az group show --name %RESOURCE_GROUP%
                    '''
                    
                    // Create app service plan
                    bat '''
                        az appservice plan create --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% --sku B1 --is-linux
                        az appservice plan show --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP%
                    '''
                    
                    // Create web app with correct runtime format
                    bat '''
                        az webapp create --resource-group %RESOURCE_GROUP% --plan %APP_SERVICE_NAME%-plan --name %APP_SERVICE_NAME% --runtime "PYTHON|%PYTHON_VERSION%"
                    '''
                    
                    // Configure web app
                    bat 'az webapp config set --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app"'
                    
                    // Create deployment package
                    bat '''
                        mkdir deploy
                        copy app.py deploy\\
                        copy requirements.txt deploy\\
                        powershell Compress-Archive -Path "deploy\\*" -DestinationPath "./deploy.zip" -Force
                    '''
                    
                    // Deploy the application
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./deploy.zip --timeout 1800'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
        always {
            cleanWs()
        }
    }
}
