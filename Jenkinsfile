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

        stage('Docker Loging'){
            steps {
                withCredentials([usernamePassword(credentialsID: 'Dockerhub-key', passwordVariable: 'password', usernameVariable: 'username')]){
                    sh 'docker login -u $username -p $password'
                }
            }
        }     


        stage('Docker  build and Push'){
            steps {
                sh 'printenv'
                sh 'docker build -t devopseje/numeric-app-devsecops:""$BUILD_NUMBER"" .'
                sh 'docker push devopseje/numeric-app-devsecops:""$BUILD_NUMBER""'
            }
        }     

    }
}