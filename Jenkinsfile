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

    stage('Mutation Tests - PIT') {
    steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
    }
    post {
        always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
    }
    }

    stage('SonarQube Analysis') {
            steps {
                    sh """

                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=devsecops2 \
                        -Dsonar.projectName='devsecops2' \
                        -Dsonar.host.url=http://51.103.114.71:9000 \
                        -Dsonar.token=sqp_a892c7b8fac03013812cdf8506c6f3a799a93025
                    """
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
            sh "sed -i 's|replace|megs17/numeric-app:${GIT_COMMIT}|g' k8s_deployment_service.yaml"

            sh "kubectl apply -f k8s_deployment_service.yaml"
        }
    }
}



    

    


    }
}
