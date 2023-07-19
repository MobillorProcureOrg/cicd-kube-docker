  pipeline{
    agent any
    tools {
      maven "maven-3.9.2"
      jdk "oracleJdk8"
    }
    environment {
      registry = "mobilloradmin/vproapp"
      ARTVERSION = "${env.BUILD_ID}"
      registryCredential = "dockerhub"
    }
    stages {
      stage('Build'){
        steps{
          sh 'mvn clean install -DskipTests'
        }
        post {
          success{
            echo 'Now Archiving it....'
            archiveArtifacts artifacts: '**/target/*.war'
          }
        }
      }
      stage('Unit Tests'){
        steps{
          sh 'mvn test'
        }
      }
      stage('Checkstyle Analysis'){
        steps{
          sh 'mvn checkstyle:checkstyle'
        }
      }
      stage('Sonar Analysis'){
        environment {
          def scannerHome = tool 'sonar4.6'
        }
        steps {
              withSonarQubeEnv('sonar-pro') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
      }
    }
    stage('Quality Gate'){
      steps{
        timeout(time:1,unit:'HOURS'){
          waitForQualityGate abortPipeline:true
        }
      }
    }
    stage('Build App Image'){
      steps{
        script {
          dockerImage=docker.build registry + ":V$BUILD_NUMBER"
        }
      }
    }
    stage('Upload Image'){
      steps {
        script {
          docker.withRegistry('',registryCredential)
          dockerImage.push("V$BUILD_NUMBER")
          dockerImage.push("latest")
        }
      }
    }
    stage('Remove Unused docker image'){
      steps {
        sh "docker rmi $registry:V$BUILD_NUMBER"
      }
    }
    stage('Kubernetes Deploy') {
      agent {label 'KOPS'}
      steps {
        sh "helm upgrate --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
      }

    }
  }
  }