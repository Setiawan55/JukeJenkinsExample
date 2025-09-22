pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec' //te
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

    // stage('SonarQube - SAST') {
    //   steps {
    //     sh "mvn clean verify sonar:sonar -Dsonar.projectKey=rnd-application -Dsonar.projectName='rnd-application' -Dsonar.host.url=https://sq-juke.diset.my.id -Dsonar.token=sqp_579469c4b2e440ce6d43968dbf5625934e90d6fd"
    //   }
    // }
    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=rnd-application -Dsonar.projectName='rnd-application' -Dsonar.host.url=https://sq-juke.diset.my.id -Dsonar.token=sqp_579469c4b2e440ce6d43968dbf5625934e90d6fd"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t dyset5/numeric-app-example:""$GIT_COMMIT"" .'
          sh 'docker push dyset5/numeric-app-example:""$GIT_COMMIT""'
        }
      }
    }
  }
}