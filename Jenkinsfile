def IMG_BUILD_ID;
def WORKSPACE;

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def GIT_COPYIMG_PBK_TAG = "0.2.1";
def GIT_CUSTOMOS_TOML_TAG = "0.1.0";

def GIT_URL_CUSTOMOS_TOML = "https://github.com/bikoizle/iac_lorax-blueprint-customos";
def GIT_URL_COPYIMG = "https://github.com/bikoizle/iac_ansible-playbook-copyimg";

def COPYIMG_PBK_DIR = "copyimg";
def CUSTOMOS_TOML_DIR = "coreos";

def CUSTOMOS_TOML = "customos.toml"; 
def COPYIMG_PBK = "copyimg.yml";

def COMPOSER_BPT = "customos";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS";
def OS_FILE;


node {

    stage("Fetch files"){

     echo "Fetching CustomOS blueprint file"

     checkout([$class: 'GitSCM',
            branches: [[name: "refs/tags/$GIT_CUSTOMOS_TOML_TAG"]],
            userRemoteConfigs: [[
                credentialsId: "$GIT_CREDS_ID",
                url: "$GIT_URL_CUSTOMOS_TOML"]],
            extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$CUSTOMOS_TOML_DIR"]],
        ])

     echo "Fetching copyimg playbook"

     checkout([$class: 'GitSCM', 
            branches: [[name: "refs/tags/$GIT_COPYIMG_PBK_TAG"]],
            userRemoteConfigs: [[
                credentialsId: "$GIT_CREDS_ID",
                url: "$GIT_URL_COPYIMG"]],
            extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$COPYIMG_PBK_DIR"]],
        ])
    }

    stage("Load blueprint"){

     sh "composer-cli blueprints push $CUSTOMOS_TOML_DIR/$CUSTOMOS_TOML"

    }

    stage("Build Image"){
     IMG_BUILD_ID = sh(returnStdout: true, script: "composer-cli compose start $COMPOSER_BPT qcow2 | cut -d ' ' -f 2").trim()
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
           playbook: "$COPYIMG_PBK_DIR/$COPYIMG_PBK",
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
