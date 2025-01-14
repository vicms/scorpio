#! /usr/bin/env groovy

pipeline {

  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        sh 'mvn clean package'
        // Add steps here
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
  openshift.withProject("cloudquestarch-dev") {
  
    def buildConfigExists = openshift.selector("bc", "codelikethewind").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=codelikethewind", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
    } 
    
    openshift.selector("bc", "codelikethewind").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("cloudquestarch-dev") { 
    def deployment = openshift.selector("dc", "codelikethewind") 
    
    if(!deployment.exists()){ 
      openshift.newApp('codelikethewind', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "codelikethewind").related('pods').untilEach(1) { 
        return (it.object().status.phase == "Running") 
      } 
    } 
  } 
}
        }
      }
    }
  }
} 
