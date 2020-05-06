def eng_account = "895523100917"

def project = [:]
project.config      = 'hmpps-env-configs'
project.d_man_dep   = 'delius-manual-deployments'

def environments = [
  'delius-core-sandpit',
  'delius-core-dev',
  'delius-auto-test',
  'delius-test',
  'delius-perf',
  'delius-stage',
  'delius-mis-dev',
  'delius-mis-test',
  'delius-po-test1',
  'delius-training',
  'delius-training-test',
  'delius-perf',
  'delius-pre-prod',
  'delius-prod'
]


def prepare_env() {
    sh '''
        docker pull mojdigitalstudio/hmpps-ansible-builder:latest
    '''
}

def run_ansible(environment_name) {

    sshagent (credentials: ['hmpps_integration_test-key']) {
        sh """
        #!/usr/env/bin bash
        set +x
        docker run --rm \
        -v `pwd`:/home/tools/data \
        -v ~/.ssh:/home/tools/.ssh \
        -v $SSH_AUTH_SOCK:/ssh-agent \
        -e SSH_AUTH_SOCK=/ssh-agent \
        mojdigitalstudio/hmpps-ansible-builder \
            bash -c \"export ANSIBLE_CONFIG=/home/tools/data/hmpps-env-configs/ansible/ansible.cfg && \
                  ansible-playbook -u hmpps_integration_test \
                  -i "/home/tools/data/hmpps-env-configs/${environment_name}/ansible" \
                  ./delius-manual-deployments/operations/oracle_parameters/set_oracle_parameters.yml \
                  --extra-vars "target_host=oracle_databases" \
                  -v \"
        set -x
        """
    }
}

pipeline {

  agent { label "jenkins_slave" }

  parameters {
    choice(name: 'environment_name', choices: environments, description: 'Select environment')
   }

  stages {
    stage('Setup') {
      steps {
        dir( project.config ) {
          git url: 'git@github.com:ministryofjustice/' + project.config, branch: env.GIT_BRANCH.split(/\//)[1], credentialsId: 'f44bc5f1-30bd-4ab9-ad61-cc32caf1562a'
        }
        dir( project.d_man_dep ) {
          git url: 'git@github.com:ministryofjustice/' + project.d_man_dep,  branch: env.GIT_BRANCH.split(/\//)[1], credentialsId: 'f44bc5f1-30bd-4ab9-ad61-cc32caf1562a'
        }
        prepare_env()
      }
    }

    stage('Set Custom Oracle Parameters') {
      steps {
        run_ansible(environment_name)
      }
    }
  }

  post {
    always {
      deleteDir()
      }
    failure {
      slackSend(message: "Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'danger')
    }
  }
}
