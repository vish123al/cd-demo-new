def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT-${env.BUILD_NUMBER}"
    def gitCommit = readFile("GIT_COMMIT-${env.BUILD_NUMBER}").trim()
    sh "rm -f GIT_COMMIT-${env.BUILD_NUMBER}"
    return gitCommit
}

def gitEmail() {
    sh "git --no-pager show -s --format='%ae' ${gitCommit()} > GIT_EMAIL-${env.BUILD_NUMBER}"
    def gitEmail = readFile("GIT_EMAIL-${env.BUILD_NUMBER}").trim()
    sh "rm -f GIT_EMAIL-${env.BUILD_NUMBER}"
    return gitEmail
}

def ipAddress() {
    sh "docker inspect test-container-${env.BUILD_NUMBER} | jq -r '.[0].NetworkSettings.IPAddress' > IP_ADDRESS-${env.BUILD_NUMBER}"
    def ipAddress = readFile("IP_ADDRESS-${env.BUILD_NUMBER}").trim()
    sh "rm -f IP_ADDRESS-${env.BUILD_NUMBER}"
    return ipAddress
}


node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm


    // Build Docker image
    stage 'Build'
    sh "docker build -t vishaldenge/cd-demo-app:${gitCommit()} ."


    // Test Docker image
    stage 'Test'
    //sh "docker run -d --name=test-container-${env.BUILD_NUMBER} mesos/cd-demo-app:${gitCommit()}"
   // sh "docker run mesosphere/linkchecker linkchecker --no-warnings http://${ipAddress()}:4000/"


    // Log in and push image to Docker Hub
    stage 'Publish'
    
    //withCredentials(
      //  [[
        //    $class: 'UsernamePasswordMultiBinding',
          //  credentialsId: 'docker-hub-credentials',
            //passwordVariable: 'DOCKER_HUB_PASSWORD',
            //usernameVariable: 'DOCKER_HUB_USERNAME'
        //]]
    //) {
      //  sh "docker login -u '${env.DOCKER_HUB_USERNAME}' -p '${env.DOCKER_HUB_PASSWORD}' -e demo@mesosphere.com"
        sh "docker login -u 'vishaldenge' -p 'v!sh@l123'"
        sh "docker push vishaldenge/cd-demo-app:${gitCommit()}"
    //}


    // Deploy
    stage 'Deploy'

    marathon(
        url: 'http://172.29.133.15:8080',
        forceUpdate: false,
      //  credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appid: 'cd-demo-app',
        docker: "vishaldenge/cd-demo-app:${gitCommit()}".toString(),
      //  labels: ['lastChangedBy': "${gitEmail()}".toString()]
    )


    // Clean up
    stage 'Clean'
   // sh "docker kill test-container-${env.BUILD_NUMBER}"
    //sh "docker rm test-container-${env.BUILD_NUMBER}"
}
