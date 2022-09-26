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

         }

        stage('Mutation Tests - PIT'){
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }

        }

        stage('Deploy to sonarqube'){
            steps{
                withSonarQubeEnv('sonarqube'){
                sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application \
                 -Dsonar.host.url=http://devsecops-ejemaster.eastus.cloudapp.azure.com:9000  "
             }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        } 

        stage('Vulnerability Scan - for  Docker') {
            steps {
                    sh 'mvn dependency-check:check'
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
     post { 
        always { 
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
    }
}