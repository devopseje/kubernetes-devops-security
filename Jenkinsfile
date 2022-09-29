pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "devopseje/numeric-app-devsecops:${BUILD_NUMBER}"
    applicationURL = "http://devsecops-demo.eastus.cloudapp.azure.com/"
    applicationURI = "/increment/99"
  }


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
            steps{
                withSonarQubeEnv('sonarqube'){
                sh "mvn sonar:sonar \
                 -Dsonar.projectKey=numeric-application2 \
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
                parallel(
                    "Dependency scan": {
                        sh 'mvn dependency-check:check'
                    },
                    "Trivy Scan": {
                        sh 'bash ./trivy-docker-images-scan.sh'
                    },
                     "OPA Conftest": {
                        sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
                     }
                )
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }  

        stage('Docker  build and Push'){
            steps {
                withDockerRegistry([credentialsId: 'Dockerhub-key', url:""]) {              
                sh 'printenv'
                sh 'sudo docker build -t devopseje/numeric-app-devsecops:"$BUILD_NUMBER" .'
                sh 'docker push devopseje/numeric-app-devsecops:"$BUILD_NUMBER"'
                sh 'docker rmi devopseje/numeric-app-devsecops:$BUILD_NUMBER'
                }

            }
        }

        // stage('Vulnerability Scan - kubernetes'){
        //     steps {
        //         sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
        //     }
        // }

        stage('Vulnerability Scan - Kubernetes') {
          steps {
            parallel(
            "OPA Scan": {
                sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
            },
            "Kubesec Scan": {
                sh 'bash kubesec-scan.sh'
            },
            "Trivy scan": {
                sh 'bash trivy-k8s-scan.sh'
            }
            )
        }
        }


            // deploy to kubernete sans verifier si l´application est en status deploy.
   //     stage('Kubernetes Deployment'){
   //         steps{
  //              withKubeConfig([credentialsId: 'kubeconfig']){
  //                  sh "sed -i 's#replace#devopseje/numeric-app-devsecops:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
 //                   sh 'kubectl apply -f k8s_deployment_service.yaml'
 //               }
 //           }
//        } 

   stage('K8s-Deployment -Dev'){
        steps {
            parallel(
                "Deployment to k8s-server": {
                    withKubeConfig([credentialsId: 'kubeconfig']){
                        sh 'bash k8s-deployment.sh'
                    }
                },
                "Rollout status": {
                    withKubeConfig([credentialsId: 'kubeconfig']){
                        sh 'bash k8s-deployment-rollout-status.sh'
                    }
                }
            )
        }
   }

    }
    
}