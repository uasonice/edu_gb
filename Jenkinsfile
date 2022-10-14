// execute 099
import java.text.SimpleDateFormat
def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())

pipeline { agent any
  environment {
    strDockerTag = "${TODAY}_${BUILD_ID}"
    strDockerImage ="uasonice/cicd_gb:${strDockerTag}"
		dockerID = "DockerHub_uasonice"
		gitUrl = "https://github.com/${myID}/edu_gb"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'master', url:gitUrl
        dir('/root/sub-workspace/guestbook-config'){
          git branch: 'master', url:"https://github.com/${myID}/guestbook-config"
        }
      }
    }
    stage('Build') {
      steps { sh './mvnw clean package' }
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

    stage('Config-Repo PUSH') {
      environment {
        GITHUB_ACCESS_TOKEN = credentials('github-access-token')
      }
      steps {
        dir('/root/sub-workspace/guestbook-config'){
          sh '''
            sed -i "s/cicd_guestbook:.*/cicd_guestbook:${strDockerTag}/g" guestbook/guestbook_deploy.yaml
            git add guestbook/guestbook_deploy.yaml
            git commit -m "[UPDATE] guestbook image tag - ${strDockerImage} (by jenkins)"
            git push "https://uasonice:${GITHUB_ACCESS_TOKEN}@github.com/${myID}/guestbook-config.git"
          '''
        }
      }
    }

    stage('ArgoCD Sync') {
      environment {
        ARGOCD_API_TOKEN = credentials('argocd-api-token')
      }
      steps {
        sh '''
          TOKEN="${ARGOCD_API_TOKEN}"
          PAYLOAD='{"prune": true}'
          curl -v -k -XPOST \
            -H "Authorization: Bearer ${TOKEN}" \
            https://192.168.56.200/api/v1/applications/guestbook/sync
        '''
      }
    }
  }
}

