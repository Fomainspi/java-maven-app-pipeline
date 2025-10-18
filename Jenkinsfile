pipeline {
  agent any

  tools {
    jdk 'jdk17'
    maven 'maven3'
  }

  environment {
    SCANNER_HOME = tool 'sonar-scanner'
  }

  stages {
    stage('Git Checkout') {
      steps {
        git credentialsId: 'git-cred', url: 'https://github.com/Fomainspi/java-maven-app-pipeline.git'
      }
    }

    stage('Compile') {
      steps {
        sh "mvn compile"
      }
    }

    stage('Test') {
      steps {
        sh "mvn test"
      }
    }

    stage('File System Scan') {
      steps {
        sh "trivy fs ."
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar') {
          sh """ ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=java-maven-app -Dsonar.projectKey=java-maven-app \
              -Dsonar.sources=src \
              -Dsonar.java.binaries=. """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
        }
      }
    }

    stage('Build Artifact') {
      steps {
        sh "mvn package"
      }
    }

    stage('Publish To Nexus') {
      steps {
        withMaven(
          globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
          sh "mvn deploy"
        }
      }
    }

    stage('Build & Tag Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker_token') {
            sh "docker build -t fomawill/java-maven-app:1.6 ."
          }
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker_token', toolName: 'docker') {
            sh "docker push fomawill/java-maven-app:1.6"
          }
        }
      }
    }
  } // <-- closing brace for 'stages'

} // <-- missing closing brace for 'pipeline'
