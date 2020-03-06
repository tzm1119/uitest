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
    stages {
        stage('build test image'){
            steps {
                sh "docker build -f ./Dockerfile -t ${DOCKER_REPO_URL}/uitest:${env.BRANCH_NAME}-${env.BUILD_ID} -t ${DOCKER_REPO_URL}/uitest:latest ."
                sh "docker login docker.pkg.github.com -u ${CREDS_GITHUB_REGISTRY_USR} -p ${CREDS_GITHUB_REGISTRY_PSW}"
                sh "docker push ${DOCKER_REPO_URL}/uitest:latest"
            }
        }
    }

    stages {
        stage('run ui test in container'){
            steps {
                script {
                    server = getHost()
                    echo "copy docker-compose-hub file to remote server...." 
                    sshCommand remote: server, command: "mkdir -p uitest"
                    sshCommand remote: server, command: "mkdir -p uitest/report"        
                    sshPut remote: server, from: 'docker-compose-hub', into: './uitest'
                    
                    echo "stopping previous docker containers...."       
                    sshCommand remote: server, command: "docker login docker.pkg.github.com -u ${CREDS_GITHUB_REGISTRY_USR} -p ${CREDS_GITHUB_REGISTRY_PSW}"
                    sshCommand remote: server, command: "docker-compose -f docker-compose-hub -p uitest-hub down"
                    
                    echo "pulling newest docker images..."
                    sshCommand remote: server, command: "docker-compose -f docker-compose-hub -p uitest-hub pull"
                    
                    echo "restarting new docker containers...."
                    sshCommand remote: server, command: "docker-compose -f docker-compose-hub -p uitest-hub up -d"
                    echo "hub successfully started!"
                    
                    echo "start run ui test container ...."
                    sshCommand remote: server, command: "docker run -v ./uitest/report:/app/TestResults uitest"
                    
                    echo "finished uitest start upload report ...."
                    mstest testResultsFile:"./uitest/**/*.trx", keepLongStdio: true
                    }
            }
        }
    }
 }