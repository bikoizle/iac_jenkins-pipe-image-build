def IMG_BUILD_ID;

node {
    stage('Build Image'){
     IMG_BUILD_ID = sh(returnStdout: true, script: "composer-cli compose start test vmdk | cut -d ' ' -f 2").trim()
     echo 'Starting Image building'
     sh "echo 'VM id is: $IMG_BUILD_ID'"
     timeout(time: 1, unit: 'HOURS'){
         waitUntil{
            sh(returnStdout: true, script: "composer-cli compose status | grep $IMG_BUILD_ID").contains("FINISHED")
         }
     }
     sh "composer-cli compose image $IMG_BUILD_ID" 
    }
}
