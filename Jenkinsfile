def IMG_BUILD_ID;
def COPYIMG_DIR;
def WORKSPACE;

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS";
def OS_FILE;


node {

    stage("Fetch files"){

     git_tag = "0.2.1"
     COPYIMG_DIR = "copyimg"

     echo "Fetching copyimg playbook"

     checkout([$class: 'GitSCM', 
            branches: [[name: "refs/tags/$git_tag"]],
            userRemoteConfigs: [[
                credentialsId: "$GIT_CREDS_ID", 
                url: "https://github.com/bikoizle/iac_ansible-playbook-copyimg"]],
            extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$COPYIMG_DIR"]],
        ])
    }

    stage("Build Image"){
     IMG_BUILD_ID = sh(returnStdout: true, script: "composer-cli compose start test qcow2 | cut -d ' ' -f 2").trim()
     echo "Starting Image building"
     sh "echo 'VM id is: $IMG_BUILD_ID'"
     timeout(time: 1, unit: 'HOURS'){
         waitUntil{
            sh(returnStdout: true, script: "composer-cli compose status | grep $IMG_BUILD_ID").contains("FINISHED")
         }
     }
     sh "composer-cli compose image $IMG_BUILD_ID" 
    }

    stage("Copy image"){

     OS_FILE = "${env.WORKSPACE}" + "/" + "$IMG_BUILD_ID-disk.qcow2"

     echo "Copying image"

     withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {

       ansiblePlaybook(
           playbook: "$COPYIMG_DIR/copyimg.yml",
           vaultCredentialsId: "vault_creds",
           extraVars: [
               vault: "$vault_file",
               url: "$OS_URL",
               project: "$OS_PROJECT",
               project_domain: "$OS_PROJECT_DOMAIN",
               user_domain: "$OS_USER_DOMAIN",
               image_name: "$OS_IMAGE_NAME",
               file: "$OS_FILE"
           ])
      }

    }

}
