def IMG_BUILD_ID;
def BUILD_STATUS = 0;
def DATE = Calendar.getInstance().getTime().format('YYYYMMdd-hhmmss',TimeZone.getTimeZone('CET'));

def GIT_CREDS_ID = "70c6a9da-bbb3-45b8-8565-d34f227696d9";

def GIT_COPYIMG_PBK_TAG = "0.2.1";
def GIT_VMBUILD_PBK_TAG = "0.1.1";
def GIT_CUSTOMOS_TOML_TAG = "0.6.7";
def GIT_IMGDELETE_PBK_TAG = "0.1.0";
def GIT_VMDELETE_PBK_TAG = "0.1.0";
def GIT_GETVMINFO_PBK_TAG = "0.1.3";

def GIT_URL_CUSTOMOS_TOML = "https://github.com/bikoizle/iac_lorax-blueprint-customos.git";
def GIT_URL_COPYIMG = "https://github.com/bikoizle/iac_ansible-playbook-copyimg.git";
def GIT_URL_VMBUILD = "https://github.com/bikoizle/iac_ansible-playbook-vmbuild.git";
def GIT_URL_IMGDELETE = "https://github.com/bikoizle/iac_ansible-playbook-imgdelete.git";
def GIT_URL_VMDELETE = "https://github.com/bikoizle/iac_ansible-playbook-vmdelete.git";
def GIT_URL_GETVMINFO = "https://github.com/bikoizle/iac_ansible-playbook-getvminfo.git";

def COPYIMG_PBK_DIR = "copyimg";
def VMBUILD_PBK_DIR = "vmbuild";
def CUSTOMOS_TOML_DIR = "customos";
def IMGDELETE_PBK_DIR = "imgdelete";
def VMDELETE_PBK_DIR = "vmdelete";
def GETVMINFO_PBK_DIR = "getvminfo";

def CUSTOMOS_TOML = "customos.toml"; 
def COPYIMG_PBK = "copyimg.yml";
def VMBUILD_PBK = "vmbuild.yml";
def IMGDELETE_PBK = "imgdelete.yml";
def VMDELETE_PBK = "vmdelete.yml";
def GETVMINFO_PBK = "getvminfo.yml";

def COMPOSER_BPT = "customos";

def OS_URL = "http://bender.lan:5000/v3";
def OS_PROJECT = "admin";
def OS_PROJECT_DOMAIN = "Default";
def OS_USER_DOMAIN = "Default";
def OS_IMAGE_NAME = "CustomOS" + "-" + "$DATE";
def OS_FILE;
def OS_VM_NAME = "customos_test" + "-" + "$DATE";
def OS_VM_BUILD_TIMEOUT = "200";
def OS_VM_FLAVOUR = "lab.small";
def OS_VM_NET = "private_network";

def OS_VM_INFO;

node {

     try{

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
    
         echo "Fetching imgdelete playbook"
    
         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_IMGDELETE_PBK_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_IMGDELETE"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$IMGDELETE_PBK_DIR"]],
            ])
    
         echo "Fetching vmdelete playbook"
    
         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_VMDELETE_PBK_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_VMDELETE"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$VMDELETE_PBK_DIR"]],
            ])
    
         echo "Fetching getvminfo playbook"
    
         checkout([$class: 'GitSCM',
                branches: [[name: "refs/tags/$GIT_GETVMINFO_PBK_TAG"]],
                userRemoteConfigs: [[
                    credentialsId: "$GIT_CREDS_ID",
                    url: "$GIT_URL_GETVMINFO"]],
                extensions: [[$class: "RelativeTargetDirectory", relativeTargetDir: "$GETVMINFO_PBK_DIR"]],
            ])
    
        }
    
        stage("Load blueprint"){
    
         echo "Configuring customos user password"
    
         withCredentials([string(credentialsId: "customos_pwd", variable: "customos_creds")]) {
    
          sh "sed -i 's|insecure|$customos_creds|g' $CUSTOMOS_TOML_DIR/$CUSTOMOS_TOML"
    
         }
    
         echo "Injecting builder ssh public key in root user"
    
         withCredentials([string(credentialsId: "ssh_pub_key", variable: "key_file")]) {
    
          builder_key = sh(returnStdout: true, script: "cat $key_file").trim()
          sh "sed -i 's|ROOT_KEY|$builder_key|g' $CUSTOMOS_TOML_DIR/$CUSTOMOS_TOML"
    
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
    
        stage("Get VM info"){
    
         echo "Getting VM info"
    
         withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {
    
           ansiblePlaybook(
               playbook: "$GETVMINFO_PBK_DIR/$GETVMINFO_PBK",
               vaultCredentialsId: "vault_creds",
               extraVars: [
                   vault: "$vault_file",
                   url: "$OS_URL",
                   project: "$OS_PROJECT",
                   project_domain: "$OS_PROJECT_DOMAIN",
                   user_domain: "$OS_USER_DOMAIN",
                   vm_name: "$OS_VM_NAME",
                   timeout: "$OS_VM_BUILD_TIMEOUT",
               ])
          }

         echo "Loading VM info JSON file"

         OS_VM_INFO = readJSON file: "$GETVMINFO_PBK_DIR/output/vminfo.json"

        }
    
        stage("Test VM"){
    
         echo "Removing old VM IP address ssh fingerprint"
    
         vm_ip_address = OS_VM_INFO['accessIPv4'][0]

         sh "ssh-keygen -R $vm_ip_address"
    
         echo "Running flake8 in $CUSTOMOS_TOML_DIR tests"
    
         sh "flake8 $CUSTOMOS_TOML_DIR/tests"
    
         echo "Running $CUSTOMOS_TOML_DIR tests with pytest"
    
         sh "py.test -v --hosts='ssh://root@$vm_ip_address' $CUSTOMOS_TOML_DIR/tests/*.py"
    
        }

    }
    catch(error){
      BUILD_STATUS = 1 
      println error.getMessage()
    }

    stage("Clean up"){

     echo "Deleting VM"

     withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {

       ansiblePlaybook(
           playbook: "$VMDELETE_PBK_DIR/$VMDELETE_PBK",
           vaultCredentialsId: "vault_creds",
           extraVars: [
               vault: "$vault_file",
               url: "$OS_URL",
               project: "$OS_PROJECT",
               project_domain: "$OS_PROJECT_DOMAIN",
               user_domain: "$OS_USER_DOMAIN",
               vm_name: "$OS_VM_NAME",
               timeout: "$OS_VM_BUILD_TIMEOUT",
           ])
      }

      if (BUILD_STATUS == 1){

        echo "Deleting Image"

        withCredentials([string(credentialsId: "vault_path", variable: "vault_file")]) {
   
          ansiblePlaybook(
              playbook: "$IMGDELETE_PBK_DIR/$IMGDELETE_PBK",
              vaultCredentialsId: "vault_creds",
              extraVars: [
                  vault: "$vault_file",
                  url: "$OS_URL",
                  project: "$OS_PROJECT",
                  project_domain: "$OS_PROJECT_DOMAIN",
                  user_domain: "$OS_USER_DOMAIN",
                  image_name: "$OS_IMAGE_NAME",
                  timeout: "$OS_VM_BUILD_TIMEOUT",
              ])
         }

      }

      echo "Deleting lorax composer build"

      sh "composer-cli compose delete $IMG_BUILD_ID"

      echo "Deleting qcow2 file"

      sh "rm -f $OS_FILE"

    }

}
