pipeline {
   agent any
   stages {
      stage('Repo') {
	 steps {
             sh '''
                repo init -u https://github.com/fwruck/trustme_main.git -b ${BRANCH_NAME} -m ${BRANCH_NAME}.xml
                repo sync -j8
             '''

         }
      }
      stage('Build') {
         agent { dockerfile {
            dir 'trustme/build/yocto/docker'
            args '--entrypoint=\'\''
            reuseNode true
         } }
         steps {
            sh '''
               export LC_ALL=en_US.UTF-8
               export LANG=en_US.UTF-8
               export LANGUAGE=en_US.UTF-8
               echo branch name from Jenkins: ${BRANCH_NAME}
               . init_ws.sh out-yocto

               bitbake trustx-cml-initramfs multiconfig:container:trustx-core

               bitbake trustx-cml
            '''
         }
      }
   }

   post {
      always {
         archiveArtifacts artifacts: 'out-yocto/tmp/deploy/images/**/trustme_image/trustmeimage.img', fingerprint: true
      }
  }
}
