pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
    
    stage('Unit Tests - JUnit and Jacoco') {
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
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'docker build -t megs17/numeric-app:"$GIT_COMMIT" .'
            sh 'docker push megs17/numeric-app:"$GIT_COMMIT"'
        }
    }
     }

    stage('Kubernetes Deployment - DEV') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
            // Update image directly in the Deployment
            sh "kubectl set image deployment/devsecops devsecops-container=megs17/numeric-app:${GIT_COMMIT}"

            // (Optional) Wait for rollout to finish
            sh "kubectl rollout status deployment/<deployment-name>"
        }
    }
}


    

    


    }
}
