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
          [name: '*/master']
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

    stage('Start SonarQube Analysis') {
      steps {
        echo "Start SonarQube Analysis"
        withSonarQubeEnv("Test_Sonar") {
          bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-${username} /n:nagp-devops-assign-1 /v:1.0 -d:sonar.cs.opencover.reportsPaths='test-project/TestResults/*/coverage.opencover.xml' -d:sonar.cs.xunit.reportsPaths='test-project/TestResults/TestFileReport.xml'"
        }
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

    stage('Test Case Execution') {
      steps {
        echo "Execute Unit Test"
        bat "dotnet test test-project\\test-project.csproj --settings coverlet.runsettings -l:trx;LogFileName=TestFileReport.xml"
      }
    }

    stage('Stop SonarQube Analysis') {
      steps {
        echo "Stop SonarQube Analysis"
        withSonarQubeEnv("Test_Sonar") {
          bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
        }
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
            bat "docker tag i-${username}-master ${registry}:$BUILD_NUMBER"
            withDockerRegistry(credentialsId: 'DockerHub', url: '') {
              bat "docker push ${registry}:$BUILD_NUMBER"
            }

          }
        }
      }
    }

    stage('Deploy Docker Image') {
      steps {
        echo "Deploy on docker"
        bat "docker run --name c-${username}-${BRANCH_NAME} -d -p 7200:80 i-${username}-${BRANCH_NAME}"
      }
    }
    
    stage('kubernetes deployment') {
            steps {
                echo "kubernetes deployment"
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
