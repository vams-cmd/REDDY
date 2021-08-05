pipeline {
    
    agent any
    
    tools {
        maven 'MAVEN_HOME'
    }
    
    stages {
        
        stage('Checkout') {
            steps {
            checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'vamsi-git', url: 'https://github.com/vams-cmd/REDDY.git']]]
            }
        }
        
        stage('Build') {
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
        
        stage('Nexus Upload') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'webapp', classifier: '', file: 'webapp/target/webapp.war', type: 'war']], credentialsId: 'bbca81fd-e29d-4636-84a0-d40c994b7257', groupId: 'com.example.maven-project', nexusUrl: '13.127.7.162:9000', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage('Dev Deploy') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'deployer_user', path: '', url: 'http://192.168.70.132:8081')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage('Dev Approval') {
            steps {
                echo "Taking approval from DEV Manager for QA Deployment"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to deploy into QA ?', submitter: 'admin'
                }
            }
        }
        
        stage('QA Deploy') {
            steps {
                echo "deploying to QA Env "
                deploy adapters: [tomcat8(credentialsId: 'deployer_user', path: '', url: 'http://192.168.70.132:8081')], contextPath: null, war: '**/*.war'
            }
        }
    }
}
