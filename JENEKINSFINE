pipeline {
    agent any

    environment {
        REGISTRY = "yourregistry.azurecr.io"
        IMAGE_NAME = "myapp"
        AZURE_CREDENTIALS = "azure-sp"   // Jenkins stored credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your/repo.git'
            }
        }

        stage('Login to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS}",
                    subscriptionIdVariable: 'AZ_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZ_CLIENT_ID',
                    clientSecretVariable: 'AZ_CLIENT_SECRET',
                    tenantIdVariable: 'AZ_TENANT_ID')]) {
                        sh '''
                        az login --service-principal -u $AZ_CLIENT_ID -p $AZ_CLIENT_SECRET --tenant $AZ_TENANT_ID
                        az acr login --name ${REGISTRY}
                        '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push Image to ACR') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Deploy to Azure WebApp') {
            steps {
                sh '''
                az webapp create --resource-group MyResourceGroup --plan MyAppServicePlan \
                    --name my-jenkins-app --deployment-container-image-name $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }
}
