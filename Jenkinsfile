pipeline {
  agent any
  tools {
    maven 'M3'
  }

  environment {
    HARBOR_HOST = '@IP:8082'
  }

  stages {
    stage('Clean Maven Repository') {
      steps {
        echo 'Cleaning local Maven repository...'
        sh 'rm -rf ~/.m2/repository/*'
      }
    }

    stage('Build') {
      steps {
        echo 'Building with Maven'
        dir('app') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        echo 'Running SonarQube analysis'
        dir('app') {
          withSonarQubeEnv('SonarQube') {
            sh 'mvn sonar:sonar -Dsonar.projectKey=CI-CD-Project'
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        waitForQualityGate abortPipeline: true
      }
    }

    stage('Deploy Nexus') {
      steps {
        echo 'Deploying to Nexus snapshots'
        dir('app') {
          sh 'mvn deploy -DskipTests'
        }
      }
    }

    stage('Docker Build') {
      steps {
        echo 'Building Docker image'
        sh 'docker build -t reservia-app:v1 app'
      }
    }

    stage('Push Harbor') {
      steps {
        echo 'Tag and push to Harbor'
        sh 'docker tag reservia-app:v1 ${HARBOR_HOST}/reservia/reservia-app:v1'
        sh 'docker push ${HARBOR_HOST}/reservia/reservia-app:v1'
      }
    }
  }
  post {
    always {
      echo 'Pipeline finished'
    }
  }
}