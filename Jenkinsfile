 pipeline {
     agent any
     triggers {
         cron('H H(16-17) * * *')
     }
     tools {
         jdk 'graalvm'
     }
     options {
         timestamps()
         //ansiColor("xterm")
         buildDiscarder(logRotator(numToKeepStr: '50'))
     }
     stages {
         stage('Checkout') {
             steps {
                 git branch: 'main', url: 'https://github.com/Frederik88/quarkus.git'
             }
         }
         stage('Build') {
             steps {
                 sh "echo $JAVA_HOME"
                 sh "java --version"
                 sh 'mvn -V clean install -DskipTests -DskipITs -DskipDocs'
             }
         }
         stage('Unit Test') {
             options {
                 timeout(time: 4, unit: 'HOURS')
             }
             steps {
                 sh 'mvn clean verify -DskipITs=true'
             }
             post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
             }
         }
         stage('Test Native') {
             options {
                 timeout(time: 7, unit: 'HOURS')
             }
             steps {
                 sh " mvn -fn clean verify -DskipDocs -pl '!integration-tests/gradle,!integration-tests/devtools' -Dnative"
             }
             post {
                 always {
                     junit '**/target/*-reports/TEST*.xml'
                 }
             }
         }
         stage('Reports') {
             parallel {
                 stage('Disk usage') {
                     steps {
                         sh 'du -cskh */*'
                     }
                 }
                 stage('Archive artifacts') {
                     steps {
                         archiveArtifacts artifacts: '**/target/*-reports/TEST*.xml', fingerprint: false
                     }
                 }
             }
         }
     }
     post {
         always {
             cleanWs()
         }
     }
 }