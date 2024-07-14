pipeline {
 agent { node { label "maven-sonarqube-node" } }
 parameters   {
   choice(name: 'aws_account',choices: ['108367059910'], description: 'aws account hosting image registry')
   choice(name: 'ecr_tag',choices: ['1.0.0','1.1.0','1.2.0'], description: 'Choose the ecr tag version for the build')
       }
tools {
    maven "Maven-3.9.8"
    }
    stages {
      stage('1. Git Checkout') {
        steps {
          git branch: 'main', credentialsId: 'github-repo-PAT', url: 'https://github.com/yanickwanzi/addressbook.git'
        }
      }
      stage('2. Build with maven') { 
        steps{
          sh "mvn clean package"
         }
       }
      stage('3. SonarQube analysis') {
      environment {SONAR_TOKEN = credentials('sonar-project-token')}
      steps {
       script {
         def scannerHome = tool 'SonarQube-Scanner-6.1.0.4477';
         withSonarQubeEnv("sonarqube-integration") {
         sh "${tool("SonarQube-Scanner-6.1.0.4477")}/bin/sonar-scanner  \
           -Dsonar.projectKey=addressbook-webapp \
           -Dsonar.projectName='addressbook-webapp' \
           -Dsonar.host.url=https://sonarqube.king1z.com \
           -Dsonar.token=$SONAR_TOKEN \
           -Dsonar.sources=src/main/java/ \
           -Dsonar.java.binaries=target/classes"
          }
         }
       }
      }
      stage('4. Docker image build') {
         steps{
          sh "aws ecr get-login-password --region us-east-2 | sudo docker login --username AWS --password-stdin ${params.aws_account}.dkr.ecr.us-east-2.amazonaws.com"
          sh "sudo docker build -t addressbook ."
          sh "sudo docker tag addressbook:latest ${params.aws_account}.dkr.ecr.us-east-2.amazonaws.com/addressbook:${params.ecr_tag}"
          sh "sudo docker push ${params.aws_account}.dkr.ecr.us-east-2.amazonaws.com/addressbook:${params.ecr_tag}"
         }
       }


      stage('5. Application deployment in eks') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'k8s-kubeconfig', serverUrl: '') {
          sh "kubectl apply -f manifest"
          }
         }
       }
      stage('6. Monitoring solution deployment in eks') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'k8s-kubeconfig', serverUrl: '') {
          sh "kubectl apply -f monitoring"
          sh "chmod +x -R script"
          sh(""" script/createIRSA-AMPIngest.sh""")
          sh(""" script/createIRSA-AMPQuery.sh""")
          }
         }
       }
      stage ('7. Email Notification') {
         steps{
         mail bcc: 'wanzi1ky@gmail.com', body: '''Build is Over. Check the application using the URL below. 
         https//abook.king1z.com/addressbook-1.0
         Let me know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', cc: 'wanzi1ky@gmail.com', from: '', replyTo: '', subject: 'Application was Successfully Deployed!!', to: 'wanzi1ky@gmail.com'
      }
    }
 }
}



