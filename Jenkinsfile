pipeline {
  agent any

  tools {
    jdk 'jdk17'
    maven 'Maven3'
  }

  environment {
    SCANNER_HOME = tool 'sonar-scanner'
  }

  stages {

    stage('Git Checkout') {
      steps {
        git credentialsId: 'Github-Cred', url: 'https://github.com/Fomainspi/java-maven-app-pipeline.git'
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

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar') {
          sh """
            ${SCANNER_HOME}/bin/sonar-scanner \
              -Dsonar.projectName=java-maven-app \
              -Dsonar.projectKey=java-maven-app \
              -Dsonar.sources=src \
              -Dsonar.java.binaries=.
          """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          waitForQualityGate abortPipeline: true, credentialsId: 'Sonar4Jenkins'
        }
      }
    }

    stage('Build Artifact') {
      steps {
        sh "mvn package -DskipTests"
      }
    }

    stage('Publish To Nexus') {
      steps {
        withMaven(
          globalMavenSettingsConfig: 'Global-Maven-settings',
          mavenSettingsConfig: 'project-settings',   // REQUIRED for Nexus credentials
          jdk: 'jdk17',
          maven: 'Maven3',
          traceability: true
        ) {
          sh "mvn deploy -DskipTests"
        }
      }
    }

    stage('Build & Tag Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'Docker-cred') {
            sh "docker build -t fomawill/java-maven-app:1.7 ."
          }
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'Docker-cred') {
            sh "docker push fomawill/java-maven-app:1.7"
          }
        }
      }
    }

  } // END stages
} // END pipeline
