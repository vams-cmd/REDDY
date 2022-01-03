pipeline {
  agent any
 
  tools {
  maven 'Maven3'
  }
  stages {
    stage ('Checkout') {
      steps {
      git branch: 'main', credentialsId: 'cd6f0f77-d845-4a9c-bc44-26b9c58b60d5', url: 'https://github.com/vams-cmd/REDDY.git'
      }
    }  
    stage ('Build') {
      steps {
      sh 'mvn clean install -f webapp/pom.xml'
      }
    }
    stage('Code Quality') {
        steps {
            withSonarQubeEnv('SonarQube') {
            sh 'mvn sonar:sonar -f webapp/pom.xml'
            }    
        }
    }
    stage ('Dev Deployment') {
      steps {
      sshPublisher(publishers: [sshPublisherDesc(configName: 'Dockerhost', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /home/dockeradmin; docker stop project_image_container; docker rm project_image_container; docker rmi project_image; docker build -t project_image .; docker run -d --name project_image_container -p 8080:8080 project_image; ', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      }
  post {
      always {
          emailext body: 'approval request with testing status', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
        }
      }
    }
    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
  }
}
