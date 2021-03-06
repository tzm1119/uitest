def getHost() {
  def remote = [:]
  remote.name = 'server-dev'
  remote.host = "${env.REMOTE_HOST}"
  remote.user = "${env.CREDS_DEV_SERVER_USR}"
  remote.password = "${env.CREDS_DEV_SERVER_PSW}"
  remote.port = 22
  remote.allowAnyHosts = true
  return remote
}

pipeline {
    agent {
        label 'vm-slave' 
    }

    environment {
      REMOTE_HOST = '52.237.75.205'
      LOCAL_HOST = '52.163.121.34'
      DOCKER_REPO_URL = 'docker.pkg.github.com/boat-house-group4/boat-house'
      CREDS_GITHUB_REGISTRY = credentials('creds-github-registry')
      CREDS_DEV_SERVER = credentials('creds-dev-server')
      def server=''
    }
    stages {
        stage('before-build'){
            steps {
                sh "printenv"
            }
        }

        stage('build test image'){
            steps {
                sh "docker build -f Dockerfile -t ${DOCKER_REPO_URL}/uitest:${env.BRANCH_NAME}-${env.BUILD_ID} -t ${DOCKER_REPO_URL}/uitest:latest ."
                sh "docker login docker.pkg.github.com -u ${CREDS_GITHUB_REGISTRY_USR} -p ${CREDS_GITHUB_REGISTRY_PSW}"
                sh "docker push ${DOCKER_REPO_URL}/uitest:latest"
            }
        }

        stage('run ui test in container'){
            steps {
                script {
                    // 远程执行
                    // server = getHost()
                    // echo "copy docker-compose-hub.yml file to remote server...." 
                    // sshCommand remote: server, command: "mkdir -p uitest"
                    // sshCommand remote: server, command: "mkdir -p uitest/report"        
                    // sshPut remote: server, from: 'docker-compose-hub.yml', into: 'uitest'
                    
                    // echo "stopping previous docker containers...."       
                    // sshCommand remote: server, command: "docker login docker.pkg.github.com -u ${CREDS_GITHUB_REGISTRY_USR} -p ${CREDS_GITHUB_REGISTRY_PSW}"
                    // sshCommand remote: server, command: "docker-compose -f ./uitest/docker-compose-hub.yml -p uitest-hub down"
                    
                    // echo "pulling newest docker images..."
                    // sshCommand remote: server, command: "docker-compose -f ./uitest/docker-compose-hub.yml -p uitest-hub pull"
                    
                    // echo "restarting new docker containers...."
                    // sshCommand remote: server, command: "docker-compose -f ./uitest/docker-compose-hub.yml -p uitest-hub up -d"
                    // echo "hub successfully started!"
                    
                    // echo "start run ui test container ...."
                    // sshCommand remote: server, command: "docker run -v ~/uitest/report:/app/TestResults ${DOCKER_REPO_URL}/uitest:latest"
                    
                    // echo "finished uitest start upload report ...."
                    // sshCommand remote: server, command: "scp -r localadmin@${REMOTE_HOST}:~/uitest/report ~/uitest/report"
      
                    sh "mkdir -p uitest/report"
                    sh "docker-compose -f docker-compose-hub.yml -p uitest-hub down"
                    sh "docker-compose -f docker-compose-hub.yml -p uitest-hub pull"
                    sh "docker-compose -f docker-compose-hub.yml -p uitest-hub up -d"
                    sh "docker run -v \$(pwd)/uitest/report:/app/TestResults ${DOCKER_REPO_URL}/uitest:latest"
                    mstest testResultsFile:"**/*.trx", keepLongStdio: true
                    }
            }
        }
    }
 }