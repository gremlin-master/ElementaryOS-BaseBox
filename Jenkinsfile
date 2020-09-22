node ("master") {

    // We expect the elementaryos iso file to be placed in the userContent directory
    // which is available at: http://jenkins.zit.local:8080/userContent

    /*
     * It isn't possible to run the ElementaryOS installer with a preseed file.
     * Because of this we need to run the graphical installer and therefore we need xvfb: https://plugins.jenkins.io/xvfb/
    */
    wrap([$class: 'Xvfb']) {
        stage('checkout') {
            checkout scm
        }

        // Use packer from a docker container. See: https://hub.docker.com/r/hashicorp/packer/
        stage('Validate Packer File') {
            // Original Command:
            // packer validate -var "elementaryos_iso_path= ${JENKINS_HOME}/userContent/elementaryos-5.1-stable.20200706.iso" elementaryos-base-linux.json
            sh "docker run \
            --mount type=bind,source=${WORKSPACE},target=/mnt/ElementaryOS-BaseBox \
            hashicorp/packer:latest validate \
            -var \"elementaryos_iso_path=${JENKINS_HOME}/userContent/elementaryos-5.1-stable.20200706.iso\" \
            /mnt/ElementaryOS-BaseBox/elementaryos-base-linux.json"
        }

        stage('Build ElementaryOS BaseBox') {
            // Original Command:
            // packer build -var "elementaryos_iso_path= ${JENKINS_HOME}/userContent/elementaryos-5.1-stable.20200706.iso" -force elementaryos-base-linux.json
            /*
            --mount type=bind,source=${WORKSPACE},target=/mnt/ElementaryOS-BaseBox \
            hashicorp/packer:latest build \
            -var \"elementaryos_iso_path=${JENKINS_HOME}/userContent/elementaryos-5.1-stable.20200706.iso\" \
            /mnt/ElementaryOS-BaseBox/elementaryos-base-linux.json"
            */
        }
    }
}

