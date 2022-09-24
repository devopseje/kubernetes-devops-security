pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true" // skip test
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }

         stage('Unit Tests - Junit and Jacoco') {
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

        stage('Mutation Tests - PIT'){
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }

        stage('Deploy to sonarqube'){
            step{         
           Sh '''mvn clean verify sonar:sonar \
            -Dsonar.projectKey=numeric-application \
            -Dsonar.host.url=http://devsecops-ejemaster.eastus.cloudapp.azure.com:9000 \
            -Dsonar.login=sqp_88d8e67ac3344178ffd467661f636063842715a4 '''
             }
        }     
        stage('Docker  build and Push'){
            steps {
                withDockerRegistry([credentialsId: 'Dockerhub-key', url:""]) {              
                sh 'printenv'
                sh 'docker build -t devopseje/numeric-app-devsecops:""$BUILD_NUMBER"" .'
                sh 'docker push devopseje/numeric-app-devsecops:""$BUILD_NUMBER""'
                sh 'docker rmi devopseje/numeric-app-devsecops:"$BUILD_NUMBER"  --force'
                }

            }
        }

        stage('Kubernetes Deployment'){
            steps{
                withKubeConfig([credentialsId: 'kubeconfig']){
                    sh "sed -i 's#replace#devopseje/numeric-app-devsecops:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
                    sh 'kubectl apply -f k8s_deployment_service.yaml'
                }
            }
        }    

    }
}