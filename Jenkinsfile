pipeline {
    
    agent any
    
    environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
        username = 'parasjain01'
        registry = 'paras22/nagp-devops-assign-1'
    }

    stages {
        stage('Checkout' ){
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/captainp99/assign-devops-1.git']]])
            }
        }
        
        stage('Restore Packages') {
            steps {
                bat "dotnet restore"
            }
        }
        
        stage('Start SonarQube Analysis') {
            steps {
                echo "Start SonarQube Analysis"
                withSonarQubeEnv("Test_Sonar"){
                bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-${username} /n:nagp-devops-assign-1 /v:1.0 -d:sonar.cs.opencover.reportsPaths='test-project/TestResults/*/coverage.opencover.xml' -d:sonar.cs.xunit.reportsPaths='test-project/TestResults/TestFileReport.xml'"
                }
            }
        }
        
        stage('Code build') {
            steps {
                //Cleans the output of the project
                echo "Clean Previous Build"
                bat "dotnet clean nagp-devops-assign-1\\nagp-devops-assign-1.csproj"
                
                //Builds the project and all its dependencies
                echo "Code Build"
                bat 'dotnet build nagp-devops-assign-1\\nagp-devops-assign-1.csproj -c Release -o "nagp-devops-assign-1/app/build"'
            }
        }
        
        stage('Test Case Execution') {
            steps {
                echo "Execute Automated Unit Test"
                bat "dotnet test test-project\\test-project.csproj --settings coverlet.runsettings -l:trx;LogFileName=TestFileReport.xml"
            }
        }
        
        stage('Stop SonarQube Analysis') {
            steps {
                echo "Stop SonarQube Analysis"
                withSonarQubeEnv("Test_Sonar"){
                bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
                }
            }
        }
        
        stage('Creating docker image') {
            steps{
                echo "Creating docker image"
                bat "docker build -t i-${username}-master ."
             }
        }
        
        stage('Docker deployment') {
            steps{
                echo "Deploy on docker"
                bat "docker run --name c-${username}-master -d -p 7100:80 i-${username}-master"
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
    
    post{
        always {
            echo "Test Report Generation Step"
            xunit([MSTest(deleteOutputFiles: true, failIfNotNew: true, pattern: 'test-project\\TestResults\\TestFileReport.xml', skipNoTestFiles: true)])
        }
    }
}
