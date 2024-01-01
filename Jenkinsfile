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
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("OWASP Dependency Check"){
           steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
       
        
        stage("Deploy to Nexus"){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-settings-xml') {
                sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage("Build and Push to Docker Hub"){
               steps{
                   
                echo 'login into docker hub and pushing image....'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     sh "docker build . -t shopping-cart -f docker/Dockerfile"
                     sh "docker tag shopping-cart ${env.dockerHubUser}/shopping-cart:latest"
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push ${env.dockerHubUser}/shopping-cart:latest"


               }
           }
         }

          stage("TRIVY Docker Image Scan"){
            steps{

                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     
                    sh "trivy image ${env.dockerHubUser}/shopping-cart:latest" 

               }
                
            }
        }

         stage("Deploy to Docker Container"){
            steps{
                sh " docker run -d --name shopping-cart -p 8085:8070 nahid0002/shopping-cart:latest "
            }
        }

        stage('Deploy to kubernetes') {
            steps {
                script {
                   
                    withKubeConfig([credentialsId: 'K8s', serverUrl: '']) {
                        sh ('kubectl apply -f deployment.yaml')
                    }
                }
            }
        }

    


        

    }

     post {
            always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'nahidkishore99@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
        }   
}
