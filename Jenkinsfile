pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'//test 
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

        stage('Sonarqube-SAST')
        {
          steps{
               withSonarQubeEnv('SonarQube'){
                  sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://35.223.254.7:9000"
          }

          timeout(time: 2, unit: 'MINUTES') {
            script{
               waitForQualityGate(webhookSecretId: '5e1a7039c903d05563078d35863f75f551f069ad') abortPipeline: true //
            }
                    
           }           
          } 
       
        }

     stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t tejaspathak01/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push tejaspathak01/numeric-app:""$GIT_COMMIT""'

      

          }
        }
      }
     stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#tejaspathak01/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
     }
    }
}    