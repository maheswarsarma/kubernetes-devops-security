pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker_cred", url: ""]) {
          sh 'printenv'
          sh 'docker build -t maheswarsarma/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push maheswarsarma/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'sa-jenkins', namespace: '', restrictKubeConfigAccess: false, serverUrl: 'http://localhost:8001') {
                               sh "sed -i 's#replace#maheswarsarma/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                                sh "kubectl apply -f k8s_deployment_service.yaml"
 
                }

        }
      }
    }
    
  }

