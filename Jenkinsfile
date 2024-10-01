def openshiftName = 'demo-trade-api'
def projectName = 'dummy-trade-filler'
def version = "0.0.${currentBuild.number}"
def dockerImageTag = "${projectName}:${version}"

pipeline {
  agent any
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-17-amazon-corretto.x86_64/'
    PATH = "${JAVA_HOME}/bin:${env.PATH}"
  }
  stages {
    stage('Check java and gradle version') {
      steps {
        sh 'java --version'
        sh 'chmod a+x ./gradlew'
        sh './gradlew --version'
      }
    }
    stage('Test') {
      steps {
        sh './gradlew -Dorg.gradle.java.home=${JAVA_HOME} test'
      }
    }

    stage('Build') {
      steps {
        sh './gradlew -Dorg.gradle.java.home=${JAVA_HOME} build'
      }
    }

    stage('Build Container') {
      steps {
        sh "docker build -t ${dockerImageTag} ."
      }
    }

    stage('Deploy Container To Openshift') {
      steps {
        sh "oc project ${openshiftName} || oc new-project ${openshiftName}"
        sh "oc get service mongo || oc new-app mongo"
        sh "oc delete all --selector app=${projectName} || echo 'Unable to delete all previous openshift resources'"
        sh "oc new-app ${dockerImageTag} -l version=${version} -e DB_HOST=mongo"
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
      archiveArtifacts 'build/reports/**/*'
    }
  }
}
