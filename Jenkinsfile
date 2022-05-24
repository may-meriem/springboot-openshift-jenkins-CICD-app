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
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject(){
              return !openshift.selector("bc", "springbootapp").exists();
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.newBuild("--name=springbootapp","--image-stream=openjdk18-openshift:1.1", "--binary")
            }
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.selector("bc", "springbootapp").startBuild("--from-file=target/spring-example-0.0.1-SNAPSHOT.jar", "--wait")
            }
          } 
        }
      }
    }
    stage('Promote to UAT') {
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.tag("springbootapp:latest", "springbootapp:uat")
            }
          }  
        }
      }
    }
    stage('Create UAT') {
      when {
        expression {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              return !openshift.selector('dc', 'springbootapp-uat').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.newApp("springbootapp:latest", "--name=springbootapp-uat").narrow('svc').expose()
            }
          }
        }
      }
    }
    stage('Promote PROD') {
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.tag("springbootapp:uat", "springbootapp:prod")
            }
          }
        }
      }
    }
    stage('Create PROD') {
      when {
        expression {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              return !openshift.selector('dc', 'springbootapp-prod').exists()
            }
          }  
        }
      }
      steps {
        script {
          openshift.withCluster('okd_cluster', 'okd_cred') {
            openshift.withProject('cicd-demo'){
              openshift.newApp("springbootapp:prod", "--name=springbootapp-prod").narrow('svc').expose()
            }
          }
      }
    }
  }
}
}  
