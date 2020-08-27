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
  }
  stage('Static Code Analysis') {
    withMaven(maven: 'M3') {
        sh 'mvn clean verify sonar:sonar -Dsonar.host.url=http://172.30.15.214:9000 -Dsonar.projectName=oven -Dsonar.projectKey=oven -Dsonar.projectVersion=$BUILD_NUMBER'
    }
    echo "Static Code Analysis"
  }
  stage('Integration Test') {
    withMaven(maven: 'M3') {
        sh 'mvn clean test-compile failsafe:integration-test';
    }
    junit '**/target/failsafe-reports/TEST-*.xml'
    archive 'target/*.jar'
    echo "Integration Test"
  }
 node('docker_pt') {
   stage ('Start Tomcat'){
     sh '''cd /home/jenkins/tomcat/bin
     ./startup.sh''';
   }
   stage ('Deploy'){
     unstash 'binary'
     sh 'cp target/hello-0.0.1.war /home/jenkins/tomcat/webapps/';
   }
   stage ('Performance Testing'){
     sh '''cd /opt/jmeter/bin/
     ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl''';
     step([$class: 'ArtifactArchiver',artifacts: '**/*.jtl'])
   }
}



