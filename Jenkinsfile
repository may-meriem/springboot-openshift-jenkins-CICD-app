pipeline {
  agent {
      label 'maven'
  }
   
 triggers {
       pollSCM('*/5 * * * *')
    }
  
 options { disableConcurrentBuilds() }
  
  stages {     
    stage('Test App') {
      steps {
        sh "mvn clean test"
      }
    }
    stage('Quality Check :: Sonarqube & JaCoCo') {
      steps {
        sh "mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=e3df00d9279eb0d36033362fdc43b4476ec9b292"
      }
    }
    stage('Insatll App') {
      steps {
        sh "mvn install"
      }
    }
    withCredentials([sshUserPrivateKey(credentialsId: 'ssh-cred')]){ 
      stage('Create Image Builder') {
    
        when {
          expression {
            openshift.withCluster('okd_cluster' , 'okd_cred') {
              openshift.withProject('cicd-demo'){
                return !openshift.selector("bc", "springbootapp").exists();
              }
            }
          }
        }
        steps {
           script {
            openshift.withCluster('okd_cluster' , 'okd_cred') {
              openshift.withProject('cicd-demo'){
                openshift.newBuild("--name=springbootapp","--image-stream=openjdk18-openshift:1.1", "--binary")
                }
            }
            
           }  
        }
      }    
    }
  } 
}
