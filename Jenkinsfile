pipeline {
  agent none

  stages {
    
    stage('Plan') {
      agent { kubernetes { yamlFile "ci/pods.yaml" } }

      steps {
        container('terraform') {
            sh """
            export TF_IN_AUTOMATION=1
            export TF_CLI_ARGS="-no-color"

            terraform init
            terraform plan -out tf.plan
            """
            stash name: 'tf', includes: '.terraform/,tf.plan'
        }
      }
    }


    // stage('Approve before apply') {
    //   agent none
    //   options {
    //     timeout(time: 1, unit: 'DAYS') 
    //   }
    //   steps {
    //     input message: "Deploy to stage?"
    //   }
    //   when {
    //     beforeAgent true
    //     beforeInput true
    //     branch 'master'
    //   }
    // }

    stage('Deploy') {
      agent { kubernetes { yamlFile "ci/pods.yaml" } }

      milestone(10)

      input {
          message "Apply plan?"
      }

      steps {
          container('terraform') {
              unstash name: 'tf'
              sh """
                export TF_IN_AUTOMATION=1
                export TF_CLI_ARGS="-no-color"
                terraform apply tf.plan
              """
          }
      }
      when {
        beforeAgent true
        beforeInput true
        branch 'master'
      }
    }

  }
}
