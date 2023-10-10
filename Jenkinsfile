pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }


    stages {
        stage("Clean Workspace"){
            steps{
                cleanWs()
            }
        }


        stage("Git Checkout"){
            steps{
               git branch: 'main', url: 'https://github.com/nahidkishore/Shopping-Cart.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile -DskipTests=true"
            }
        }
          stage("Test Cases"){
            steps{
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart '''
                }
            }
        }
        
        stage("SonarQube Quality Gate"){
       
             steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
         }

     }
     
     stage("Build Artifact"){
            steps{
                sh "mvn clean install -DskipTests=true"
            }
        }
        stage("OWASP Dependency Check"){
           steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        

    }
}
