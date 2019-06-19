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

               if [ ! -z "$(git branch --list ${BRANCH_NAME})" ]; then
                  git checkout -b ${BRANCH_NAME}
               else
                  git fetch origin/${BRANCH_NAME}
                  git reset --hard origin/${BRANCHNAME}
               fi

               cd ${WORKSPACE}/out-yocto
               echo "BRANCH = \\\"${BRANCH_NAME}\\\"" > cmld_git.bbappend.jenkins
               cat cmld_git.bbappend >> cmld_git.bbappend.jenkins
               rm cmld_git.bbappend
               cp cmld_git.bbappend.jenkins cmld_git.bbappend

               bitbake trustx-cml-initramfs multiconfig:container:trustx-core

               bitbake trustx-cml
            '''
         }
      }
      stage('Deploy') {
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
               . init_ws.sh out-yocto
               rm cmld_git.bbappend
               cp cmld_git.bbappend.jenkins cmld_git.bbappend

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
