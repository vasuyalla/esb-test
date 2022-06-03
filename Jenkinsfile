pipeline {
    agent any
    tools {
      maven 'Maven 3.6.2'
    }
    environment {
      // Configure in global tool setting of jenkins slave or master
        SLACK_UPLOAD_FILE_TOKEN = "dev-slack-app-token"
        CMDLINE = "clean initialize package"
        AWS_SECRET_ACCESS_KEY = "$AWS_SECRET_ACCESS_KEY_DEV"
        AWS_ACCESS_KEY_ID = "$AWS_ACCESS_KEY_ID_DEV"
        ECR_URL = "998808488458.dkr.ecr.us-east-2.amazonaws.com/v01/esb_operationassetsinventory"
        ECR_ENV_DEV = "dev"
        POD_PATTERN = "esb-operationassetsinventory-"
        K8S_NAMESPACE = "tibco-dev"
        DOCKER_IMAGE = "esb_operationassetsinventory"
    }
    stages {
      stage('Cleanup')
      {
        steps {
          echo "Cleaning it up..."
          echo currentBuild.projectName
        }
      }
      stage('Docker Build')
      {
        steps {
          echo "Compile & Build..."
          sh 'mvn -f Opr_Inv_OperationAssetsInventory.application.parent/pom.xml clean initialize package docker:build'
        }
      }
      stage('Docker Push')
      {
        steps {
          echo "Deploying..."
          sh '''
            export THISTAG=`date "+%y%m%d%H%M"`
            aws ecr get-login-password --region us-east-2 | docker login -u AWS --password-stdin $ECR_URL

            docker tag $DOCKER_IMAGE:latest $ECR_URL:$THISTAG
            docker push $ECR_URL:$THISTAG

            docker tag $DOCKER_IMAGE:latest $ECR_URL:$ECR_ENV_DEV
            docker push $ECR_URL:$ECR_ENV_DEV
          '''
        }
      }
      stage('Container Restart')
      {
        steps {
          echo "Restarting container"
          sh '''
            export PODNAME=`kubectl -n $K8S_NAMESPACE get pods | grep $POD_PATTERN | awk '{print $1}'`
            kubectl -n $K8S_NAMESPACE delete pod $PODNAME
          '''
          }
      }
      stage('QA Test')
      {
        steps {
          echo "QA Test..."
        }
      }
    }
    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25'))
    }
}
      
