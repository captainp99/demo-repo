pipeline {

  agent any

  environment {
    scannerHome = tool name: 'sonar_scanner_dotnet'
    username = 'parasjain01'
    registry = 'paras22/nagp-devops-assign-1'
    gitHubUrl = 'https://github.com/captainp99/app_parasjain01'
  }

  //Git 
  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: env.BRANCH_NAME]
        ], extensions: [], userRemoteConfigs: [
          [url: env.gitHubUrl]
        ]])
      }
    }

    // Restore Nuget Packeges
    stage('Restore Packages') {
      steps {
        echo "Restore Nuget Packages"
        bat "dotnet restore"
      }
    }

    stage('Code build') {
      steps {
        //Cleans the output of the project
        echo "Clean Previous Build"
        bat "dotnet clean"

        //Builds the project and all its dependencies
        echo "Code Build"
        bat 'dotnet build --configuration Release"'
      }
    }
      
    
    stage ('Release Artifiact') {
        steps {
            echo 'Publishing Artifacts with release config'
            bat 'dotnet publish --configuration Release'
        }
    }

    stage('Creating docker image') {
      steps {
        echo "Creating docker image"
        bat "docker build -t i-${username}-${BRANCH_NAME} ."
      }
    }

    stage('Containers') {
      parallel {
        stage('Pre-Container Check') {
          steps {
            echo "Pre Container check - Check if container already exist"
            script {
              checkContainerExist = bat(script: "docker ps -qa -f name=c-${username}-${BRANCH_NAME}", returnStdout: true).trim().readLines().drop(1).join('')
              if (checkContainerExist) {
                echo ":: Found container - c-${username}-${BRANCH_NAME}"
                checkContainerRunning = bat(script: "docker ps -q -f name=c-${username}-${BRANCH_NAME}", returnStdout: true).trim().readLines().drop(1).join('')
                if (checkContainerRunning) {
                  echo ":: Stopping running container - c-${username}-${BRANCH_NAME}"
                  bat "docker stop c-${username}-${BRANCH_NAME}";
                }
                echo ":: Removing stopped container - ${username}-${BRANCH_NAME}"
                bat "docker rm c-${username}-${BRANCH_NAME}";
              }
            }
          }
        }
        stage('Push image to docker hub') {

          steps {

            echo "Pushing image to docker hub"
            bat "docker tag i-${username}-${BRANCH_NAME} ${registry}:$BUILD_NUMBER"
            bat "docker tag i-${username}-${BRANCH_NAME} ${registry}:latest"
            withDockerRegistry(credentialsId: 'DockerHub', url: '') {
              bat "docker push ${registry}:$BUILD_NUMBER"
              bat "docker push ${registry}:latest"
            }

          }
        }
      }
    }

    stage('Deploy Docker Image') {
      steps {
        echo "Deploy on docker"
        bat "docker run --name c-${username}-${BRANCH_NAME} -d -p 7300:80 i-${username}-${BRANCH_NAME}"
      }
    }
    
    stage('kubernetes deployment') {
            steps {
                echo "kubernetes deployment"
                // bat "gcloud container clusters get-credentials nagp-dotnet-1 --zone us-central1-c --project optimistic-yeti-321307"
               bat "kubectl apply -f deployment.yaml"
             
            }
        }
  }
    
  post {
    always {
      echo "Test Report Generation Step"
      xunit([MSTest(deleteOutputFiles: true, failIfNotNew: true, pattern: 'test-project\\TestResults\\TestFileReport.xml', skipNoTestFiles: true)])
    }
  }
}
