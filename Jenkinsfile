pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS = credentials('azure-service-principal')
        PYTHON_VERSION = '3.11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/python-webapp.git'
            }
        }
        
        stage('Setup Python') {
            steps {
                script {
                    bat 'python -m pip install --upgrade pip'
                    bat 'pip install -r requirements.txt'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    bat 'python -m pytest'
                }
            }
        }
        
        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal('azure-service-principal')]) {
                    bat '''
                        az login --service-principal -u $AZURE_CREDENTIALS_USR -p $AZURE_CREDENTIALS_PSW --tenant $AZURE_CREDENTIALS_TEN
                        az group create --name python-webapp-rg --location eastus
                        az appservice plan create --name python-webapp-plan --resource-group python-webapp-rg --sku B1 --is-linux
                        az webapp create --resource-group python-webapp-rg --plan python-webapp-plan --name python-webapp-service --runtime "PYTHON:3.11"
                        az webapp config set --resource-group python-webapp-rg --name python-webapp-service --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
} 