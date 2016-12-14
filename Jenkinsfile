node ('docker-build') {
    stage "Checkout Deployment Architecture and Operations Source"
    checkout scm
    // sh "git submodule update --init"
    
    stage "Checkout Developer Source Code"
    dir("${env.DEVPROJROOTDIR}") {
        git url: "${env.DEVPROJROOTURL}"
        sh "git submodule update --init"
        sh "git submodule update --force"
    }
    
    stage "Clear running services"
    // NOTE: this is a temporary workaround for port clashing 
    sh "docker service  ls -q | (xargs docker service rm || echo )"
    
    stage "Build Application Images"
    dir("${env.DEVPROJCOMPOSEDIR}") {
        sh "docker-compose build" // build --pull is failing on some nodes
    }
    
    stage "Upload and Checkout Docker Images from register"
    dir("${env.DEVPROJCOMPOSEDIR}") {
        sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}"
        sh "docker-compose push"
        sh "docker-compose pull"
    }
    
    stage "Create Application Bundle"
    dir("${env.DEVPROJCOMPOSEDIR}") {
        sh "docker-compose bundle -o ${env.JOB_NAME}_app.dab"
    }
    
    stage "Build Infrasture Images"
    sh "docker-compose build" // build --pull is failing on some nodes
    
    stage "Upload Infra Images from register"
    sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}"
    sh "docker-compose push"
    sh "docker-compose pull"
    
    stage "Create Infrastructure Bundle"
    sh "docker-compose bundle -o ${env.JOB_NAME}_infra.dab"
    
    stage "Merge Infrastructure and Application Bundle"
    sh "docker run --rm -v `pwd`:/data mikeagileclouds/dabmerger --out /data/${env.JOB_NAME}.dab /data/${env.JOB_NAME}_infra.dab /data/${env.DEVPROJCOMPOSEDIR}/${env.JOB_NAME}_app.dab"
    
    stage "Upload Application Bundle"
    sh "curl -k -u demouser:73pass76 -X PUT https://52.53.183.20/artifactory/agileclouds/${env.JOB_NAME}.dab -T ${env.JOB_NAME}.dab"
}
    
node ('swarm-deploy') {
    stage "Download Application Bundle"
    sh "curl -k -u demouser:73pass76  https://52.53.183.20/artifactory/agileclouds/${env.JOB_NAME}.dab -o ${env.JOB_NAME}.dab"
    
    stage "Deploy Docker App Bundle"
    sh "docker stack deploy ${env.JOB_NAME}" // deploy create as well as update stack - ?Does note seem to be working?
    
    stage "Configure Service updates for end users - External ports, volumes/networks, access control"
    sh "sh scripts/polyglot-deploy.sh ${env.JOB_NAME}"
        
    stage "Publish Swarm Node and Service details"
    sh "docker node ls"
    sh "docker service ls"
}
