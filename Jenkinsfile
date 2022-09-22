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


        stage('Docker  build and Push'){
            steps {
                sh 'printenv'
                sh 'docker build -t devopseje/numeric-app-devsecops:""$GIT_COMMIT"" .'
                sh 'docker push  devopseje/numeric-app-devsecops:""$GIT_COMMIT""'
            }
        }     

    }
}