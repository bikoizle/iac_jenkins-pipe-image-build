def IMG_BUILD_ID;
def WORKSPACE;

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def GIT_COPYIMG_PBK_TAG = "0.2.1";
def GIT_VMBUILD_PBK_TAG = "0.1.1";
def GIT_CUSTOMOS_TOML_TAG = "0.2.0";

def GIT_URL_CUSTOMOS_TOML = "https://github.com/bikoizle/iac_lorax-blueprint-customos.git";
def GIT_URL_COPYIMG = "https://github.com/bikoizle/iac_ansible-playbook-copyimg.git";
def GIT_URL_VMBUILD = "https://github.com/bikoizle/iac_ansible-playbook-vmbuild.git";

def COPYIMG_PBK_DIR = "copyimg";
def VMBUILD_PBK_DIR = "vmbuild";
def CUSTOMOS_TOML_DIR = "customos";

def CUSTOMOS_TOML = "customos.toml"; 
def COPYIMG_PBK = "copyimg.yml";
def VMBUILD_PBK = "vmbuild.yml";

def COMPOSER_BPT = "customos";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS";
def OS_FILE;
def OS_VM_NAME = "customos_test";
def OS_VM_BUILD_TIMEOUT = "200";
def OS_VM_FLAVOUR = "lab.small";
def OS_VM_NET = "private_network";

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

     echo "Fetching vmbuild playbook"

     checkout([$class: 'GitSCM',
            branches: [[name: "refs/tags/$GIT_VMBUILD_PBK_TAG"]],
            userRemoteConfigs: [[
                credentialsId: "$GIT_CREDS_ID",
                url: "$GIT_URL_VMBUILD"]],
            extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$VMBUILD_PBK_DIR"]],
        ])

    }

    stage("Load blueprint"){

     echo "Configuring customos user password"

     withCredentials([string(credentialsId: "customos_pwd", variable: "customos_creds")]) {

      sh "sed -i 's|insecure|$customos_creds|g' $CUSTOMOS_TOML_DIR/$CUSTOMOS_TOML"

     }

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

    stage("Create VM"){

     echo "Creating VM"

     withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {

       ansiblePlaybook(
           playbook: "$VMBUILD_PBK_DIR/$VMBUILD_PBK",
           vaultCredentialsId: "vault_creds",
           extraVars: [
               vault: "$vault_file",
               url: "$OS_URL",
               project: "$OS_PROJECT",
               project_domain: "$OS_PROJECT_DOMAIN",
               user_domain: "$OS_USER_DOMAIN",
               image_name: "$OS_IMAGE_NAME",
               vm_name: "$OS_VM_NAME",
               timeout: "$OS_VM_BUILD_TIMEOUT",
               vm_flavour: "$OS_VM_FLAVOUR",
               vm_net: "$OS_VM_NET"
           ])
      }

    }

}
