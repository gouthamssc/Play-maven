pipeline {
  agent {
    kubernetes {
        cloud 'Local Kubernetes cluster'
        label 'default-pod'
      }
  }  
  
  tools {
        oc 'oc-3.11'
    }
  
   environment {
      // key = credentials('oc_login_token')
       docker=credentials('docker-reg')
       SCALA_PROJECT_NAME='maas-producer-maven'
    }
    parameters{
      string(name:'PROJECT_NAME',defaultValue:'maas')
      //string(name:'OPENSHIFT_ROUTE',defaultValue:'fms-sandbox2.containers.ciocloudservices.ibm.com')
      string(name:'INSTANCE_TYPE',defaultValue:'tdd')
      choice(name: 'URL', choices: ['https://c107-e.us-south.containers.cloud.ibm.com:31542','https://c100-e.us-south.containers.cloud.ibm.com:30409'], description: 'OC_LOGIN_URL')
      choice(name: 'OPENSHIFT_ROUTE', choices: ['fms-cloud-tdd.containers.ciocloudservices.ibm.com','fms-sandbox2.containers.ciocloudservices.ibm.com'], description: 'OC Route')
      credentials credentialType: 'org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl', defaultValue: 'oc_login_token_tdd', description: '', name: 'key', required: true
    }
  stages {
    
    stage('compile') {
      steps {
     // slackSend(color: '#2056e8', message: " ${currentBuild.fullDisplayName} build started (<${env.BUILD_URL}|Open>)", channel: 'gsi-ica-jenkins-build',tokenCredentialId: "jenkins-slack-plugin")
        
        sh 'mvn compile play2:run'
      }
    }
    
    stage('test') {
      steps {
        sh 'mvn test play2:run'
      }
    }
    
    stage('dist') {
      steps {
         sh 'mvn install play2:dist'
      }
    }  
    stage('dockerbuild') {
      steps {
        sh "docker login txo-gsi-ica-docker-local.artifactory.swg-devops.com -u $docker_USR -p $docker_PSW"
        sh "docker build -t txo-gsi-ica-docker-local.artifactory.swg-devops.com/$SCALA_PROJECT_NAME:0.1 ."
        sh "docker tag txo-gsi-ica-docker-local.artifactory.swg-devops.com/$SCALA_PROJECT_NAME:0.1 txo-gsi-ica-docker-local.artifactory.swg-devops.com/$SCALA_PROJECT_NAME:latest"
        sh "docker push txo-gsi-ica-docker-local.artifactory.swg-devops.com/$SCALA_PROJECT_NAME:0.1" 
        sh "docker push txo-gsi-ica-docker-local.artifactory.swg-devops.com/$SCALA_PROJECT_NAME:latest"      
      }
    }  
    stage('openshift login') {
      environment{
          key=credentials('key')} 
      steps {
          sh 'oc login $URL --token=$key'
          sh 'oc project $PROJECT_NAME'
          sh 'oc process -f deployment.yaml -p PROJECT_NAME=$PROJECT_NAME -p INSTANCE_TYPE=$INSTANCE_TYPE -p OPENSHIFT_ROUTE=$OPENSHIFT_ROUTE -p SCALA_PROJECT_NAME=$SCALA_PROJECT_NAME | oc apply -f -'
          sh 'oc rollout latest dc/$SCALA_PROJECT_NAME-$PROJECT_NAME'
        }
    }
  }

}
