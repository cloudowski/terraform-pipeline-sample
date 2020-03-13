pipeline {
  agent none

  stages {
    
    stage('Plan') {
      agent { kubernetes { yamlFile "ci/pods.yaml" } }

      steps {
        container('terraform') {
          ansiColor('vga') {
            sh """
            export TF_IN_AUTOMATION=1
            #export TF_CLI_ARGS="-no-color"

            terraform init
            terraform plan -out tf.plan
            """
            stash name: 'tf', includes: '.terraform/,tf.plan'
          }
        }
      }
    }

    stage('Deploy') {
      agent { kubernetes { yamlFile "ci/pods.yaml" } }


      input {
          message "Apply plan?"
      }

      steps {
          milestone(10)
          container('terraform') {
              unstash name: 'tf'
              ansiColor('vga') {
                sh """
                    export TF_IN_AUTOMATION=1
                    #export TF_CLI_ARGS="-no-color"
                    terraform apply tf.plan
                """
              }
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
