pipeline {
    agent {
        docker {
            image 'my-docker-image' // Replace with your Jenkins container image
            args '-u root'       // Allows running QEMU
        }
    }
    environment {
        IMAGE_NAME = "obmc-phosphor-image-evb-ast2600-20240906065230.static.mtd"
        QEMU_SCRIPT = "qemu.sh"
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                apt-get update
                apt-get install -y wget qemu python3-distutils gcc g++ make file gawk diffstat bzip2 cpio chrpath zstd lz4
                wget https://jenkins.openbmc.org/job/latest-qemu-x86/lastSuccessfulBuild/artifact/qemu/build/qemu-system-arm
                chmod +x qemu-system-arm
                '''
            }
        }
        stage('Prepare Image') {
            steps {
                sh '''
                # Ensure the OpenBMC build has been completed on the host
                mkdir -p /workspace/qemu
                cp /host/path/to/build/evb-ast2600/tmp/deploy/images/evb-ast2600/${IMAGE_NAME} /workspace/qemu/
                '''
            }
        }
        stage('Run QEMU') {
            steps {
                script {
                    writeFile file: "${env.QEMU_SCRIPT}", text: '''
                    ./qemu-system-arm -m 512 -M ast2600-evb -nographic \
                    -drive file=./${IMAGE_NAME},format=raw,if=mtd \
                    -net nic \
                    -net user,hostfwd=:127.0.0.1:2222-10.0.2.15:22,hostfwd=:127.0.0.1:2443-10.0.2.15:443,hostfwd=udp:127.0.0.1:2623-10.0.2.15:623,hostname=qemu
                    '''
                }
                sh '''
                chmod +x ${QEMU_SCRIPT}
                ./${QEMU_SCRIPT}
                '''
            }
        }
    }
    post {
        always {
            sh 'rm -rf /workspace/qemu'
        }
    }
}
