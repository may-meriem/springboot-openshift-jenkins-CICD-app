pipeline {
  agent {
      label 'maven'
  }
   
  stages {     
     stage('Clone Repo') {
      steps {
        git url : 'https://github.com/may-meriem/springboot-openshift-jenkins-CICD-app.git' 
      }
    }
    stage('Test App') {
      steps {
        sh "mvn clean test"
      }
    }
    
    stage('Insatll App') {
      steps {
        sh "mvn install"
      }
    }
    
   stage('Build Image') {
     when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "springbootapp").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=springbootapp","--image-stream=openjdk18-openshift:1.1", "--binary")
          }
        }
      }
    }
    stage('Deploy Image') {
      steps {
        script {
          openshift.withCluster('okd_cluster' , 'okd_cred') {
              openshift.selector("bc", "springbootapp").startBuild("--from-file=target/spring-example-0.0.1-SNAPSHOT.jar", "--wait")
            
          } 
        }
      }
    }
    
}
}
