pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'python-webapp-rg'
        APP_SERVICE_NAME = 'python-webapp-service'
        PYTHON_VERSION = '3.10'
        PYTHON_PATH = 'C:\\Users\\window 10\\AppData\\Local\\Programs\\Python\\Python310\\python.exe' 
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/y/python-webapp.git'
            }
        }
        
        stage('Build') {
            steps {
                // Using full path to Python executable
                bat '''
                    "%PYTHON_PATH%" --version
                    "%PYTHON_PATH%" -m pip install --upgrade pip
                    "%PYTHON_PATH%" -m pip install -r requirements.txt
                    "%PYTHON_PATH%" -m pip install pytest
                    "%PYTHON_PATH%" -m pytest
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    bat "az group create --name $RESOURCE_GROUP --location eastus"
                    bat "az appservice plan create --name ${APP_SERVICE_NAME}-plan --resource-group $RESOURCE_GROUP --sku B1 --is-linux"
                    bat "az webapp create --resource-group $RESOURCE_GROUP --plan ${APP_SERVICE_NAME}-plan --name $APP_SERVICE_NAME --runtime \"PYTHON:${PYTHON_VERSION}\""
                    bat "az webapp config set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --startup-file \"gunicorn --bind=0.0.0.0 --timeout 600 app:app\""
                    
                    // Create deployment package
                    bat "powershell Compress-Archive -Path ./* -DestinationPath ./deploy.zip -Force"
                    
                    // Deploy the package
                    bat "az webapp deployment source config-zip --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src ./deploy.zip"
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
