echo "Building ${env.BRANCH_NAME}"

def compose_version = 'v0.12.4'
node {
  // We use credentials-binding plugin
  withCredentials([string(credentialsId: 'RANCHER_SECRET', variable: 'TOKEN')]){

    stage('Preprare'){
      checkout scm

      sh "wget https://github.com/rancher/rancher-compose/releases/download/v0.12.4/rancher-compose-linux-amd64-${compose_version}.tar.gz -O - | tar -zx"
      sh "mv rancher-compose-${compose_version}/rancher-compose . && rm -rf rancher-compose-${compose_version}"
    }

    stage('Deploy'){
      sh "./rancher-compose --project-name ${env.JOB_NAME}                         \
      --url ${RANCHER_URL}  \
      --access-key ${RANCHER_KEY} \
      --secret-key ${TOKEN}                                                   \
      --verbose up -d --force-upgrade --pull --confirm-upgrade"
    }
  }
}



