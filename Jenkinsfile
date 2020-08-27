node('master') {
  stage('Poll') {
    checkout scm
     echo "Poll"
  }
  stage('Build & Unit test') {
    withMaven(maven: 'M3') {
        sh 'mvn clean test-compile surefire:test';
    }
    junit '**/target/surefire-reports/TEST-*.xml'
    archive 'target/*.jar'
    echo "Build & Unit test"
    stash name: "binary", includes: "target/oven-0.0.1.war.src/pt/PLAN_A.jmx"
  }
  stage('Build') {
    withMaven(maven: 'M3') {
        sh 'mvn clean package'
    }
    archive 'target/*.jar'
    stash name: "binary", includes: "target/oven-0.0.1.jar.src/pt/PLAN_A.jmx"
  }
}


node('docker_pt') {
   stage ('Start Tomcat'){
     echo "Start Tomcat"
     sh '''cd /home/jenkins/tomcat/bin
     ./startup.sh''';
   }
   stage ('Deploy'){
     echo "Start Deploy"
     unstash 'binary'
     sh 'cp target/oven-0.0.1.jar /home/jenkins/tomcat/webapps/';
   }
   stage ('Performance Testing'){
     sh '''cd /opt/jmeter/bin/
     ./jmeter.sh -n -t $WORKSPACE/src/pt/PLAN_A.jmx -l $WORKSPACE/test_report.jtl''';
     step([$class: 'ArtifactArchiver',artifacts: '**/*.jtl'])
   }
 }




