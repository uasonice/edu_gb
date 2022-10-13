import java.text.SimpleDateFormat
def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())

pipeline {
  agent any
  environment {
    strDockerTag = "${TODAY}_${BUILD_ID}"
    strDockerImage ="uasonice/cicd_gb:${strDockerTag}"
		dockerID = "DockerHub_uasonice"
  }

  stages {
    stage('Checkout') {
      steps { git branch: 'master', url:'https://github.com/uasonice/edu_gb' }
    }
    stage('Build') {
      steps { sh './mvnw clean package' }
    }
    stage('Unit Test') {
      steps { sh './mvnw test' }
      
      post {
        always {
          junit '**/target/surefire-reports/TEST-*.xml'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps{ echo 'SonarQube Analysis' /* */ }
    }
    stage('SonarQube Quality Gate'){
      steps{ echo 'SonarQube Quality Gate' /* */ }
    }
    stage('Docker Image Build') {
      steps {
        script {
          oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
        }
      }
    }
    stage('Docker Image Push') {
      steps {
        script {
          docker.withRegistry('', dockerID) { oDockImage.push() }
        }
      }
    }
    stage('Staging Deploy') {
      steps {
        sshagent(credentials: ['Staging-PrivateKey']) {
          sh "ssh -o StrictHostKeyChecking=no root@192.168.56.144 docker container rm -f guestbookapp"
          sh "ssh -o StrictHostKeyChecking=no root@192.168.56.144 docker container run \
  -d -p 38080:80 \
  --name=guestbookapp \
  -e MYSQL_IP=192.168.56.140 -e MYSQL_PORT=3306 \
  -e MYSQL_DATABASE=guestbook \
  -e MYSQL_USER=root -e MYSQL_PASSWORD=education \
  ${strDockerImage} "
        }
      }
    }
    stage ('JMeter LoadTest') {
      steps {  echo 'JMeter LoadTest'
/*
        sh '~/lab/sw/jmeter/bin/jmeter.sh -j jmeter.save.saveservice.output_format=xml -n -t src/main/jmx/guestbook_loadtest.jmx -l loadtest_result.jtl' 
        perfReport filterRegex: '', showTrendGraphs: true, sourceDataFiles: 'loadtest_result.jtl' 
*/
      } 
    }
  }
  post { 
    always { 
      slackSend(tokenCredentialId: 'slack-token'
        , channel: '#교육', color: 'good'
        , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 끝났습니다XXX. Details: (<${BUILD_URL} | here >)")
    }
    success { 
      slackSend(tokenCredentialId: 'slack-token'
        , channel: '#교육', color: 'good'
        , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 성공적으로 끝났습니다XXX. Details: (<${BUILD_URL} | here >)")
    }
    failure { 
      slackSend(tokenCredentialId: 'slack-token'
        , channel: '#교육', color: 'danger'
        , message: "${JOB_NAME} (${BUILD_NUMBER}) 빌드가 실패하였습니다XXX. Details: (<${BUILD_URL} | here >)")
  }
  }
}

