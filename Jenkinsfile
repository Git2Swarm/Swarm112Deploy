node ('swarm') {
    stage ( "Checkout Deployment Architecture and Operations Source") {
        checkout scm
        // sh "git submodule update --init"
    }
    
    stage ( "Checkout Developer Source Code") {
        dir("${env.DEVPROJROOTDIR}") {
            git url: "${env.DEVPROJROOTURL}"
            sh "git submodule update --init"
            sh "git submodule update --force"
        }
    }
    
    stage ( "Clear running services") {
        // NOTE: this is a temporary workaround for port clashing 
        sh "docker service  ls -q | (xargs docker service rm || echo )"
    }
    
    stage ( "Build Application Images") {
        dir("${env.DEVPROJCOMPOSEDIR}") {
            sh "docker-compose build" // build --pull is failing on some nodes
        }
    }
    
    stage ( "Upload and Checkout Docker Images from register") {
        dir("${env.DEVPROJCOMPOSEDIR}") {
            sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}"
            sh "docker-compose push"
            sh "docker-compose pull"
        }
    }
    
    stage ( "Create Application Bundle") {
        dir("${env.DEVPROJCOMPOSEDIR}") {
            sh "docker-compose bundle -o ${env.JOB_NAME}_app.dab"
        }
    }
    
    stage ( "Build Infrasture Images") {
        sh "docker-compose build" // build --pull is failing on some nodes
    }
    
    stage ( "Upload Infra Images from register") {
        sh "docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}"
        sh "docker-compose push"
        sh "docker-compose pull"
    }
    
    stage ( "Create Infrastructure Bundle") {
        sh "docker-compose bundle -o ${env.JOB_NAME}_infra.dab"
    }
    
    stage ( "Merge Infrastructure and Application Bundle") {
        sh "docker run --rm -v `pwd`:/data mikeagileclouds/dabmerger --out /data/${env.JOB_NAME}.dab /data/${env.JOB_NAME}_infra.dab /data/${env.DEVPROJCOMPOSEDIR}/${env.JOB_NAME}_app.dab"
    }
    
    stage ( "Upload Application Bundle") {
        echo 'Place holder for DAB file push'
        // sh "curl -u admin:73admin79 -X PUT http://169.55.59.106:8080/artifactory/ext-release-local/${env.JOB_NAME}.dab -T ${env.JOB_NAME}.dab"
    }
    
    stage ( "Download Application Bundle") {
        echo 'Place holder for DAB file push'
        // sh "curl -u admin:73admin79 -X PUT http://169.55.59.106:8080/artifactory/ext-release-local/${env.JOB_NAME}.dab -o ${env.JOB_NAME}.dab"
    }
    
    stage ( "Deploy Docker App Bundle") {
        sh "docker stack deploy ${env.JOB_NAME}" // deploy create as well as update stack - ?Does note seem to be working?
    }
    
    stage ( "Configure Service updates for end users - External ports, volumes/networks, access control") {
        sh "sh scripts/polyglot-deploy.sh ${env.JOB_NAME}"
    }
        
    stage ( "Publish Swarm Node and Service details") {
        sh "docker node ls"
        sh "docker service ls"
    }
}
