pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'TARGET_HOST',
            choices: ['all', 'ansible_controller', 'ansible_node1', 'ansible_node2', 'ansible_node3', 'ansible_node4'],
            description: 'Select the host or group to target with Ansible ping'
        )
        choice(
            name: 'PLAYBOOK_TO_RUN',
            choices: ['ping.yml', 'gather_system_info.yml', 'custom_playbook.yml'], // Add more playbooks here
            description: 'Select the playbook to run'
        )
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_FORCE_COLOR = 'True'
        REMOTE_USER = 'ansible'
        REMOTE_HOST = '192.168.1.200' // <-- Replace with your Ansible controller IP
        REMOTE_DIR = '/home/ansible/jenkins_ansible_pipeline'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Copy Code to Ansible Controller') {
            steps {
                sshagent (credentials: ['ssh_ansible_key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'mkdir -p ${REMOTE_DIR}'
                        rsync -avz --delete ./ ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }

        stage('Run Ansible Ping Remotely') {
            steps {
                sshagent (credentials: ['ssh_ansible_key']) {
                    sh """
                        ssh ${REMOTE_USER}@${REMOTE_HOST} '
                            cd ${REMOTE_DIR}/playbooks &&
                            echo "Running ${PLAYBOOK_TO_RUN} for ${TARGET_HOST}" &&
                            ansible-playbook -i ../inventory.ini ${PLAYBOOK_TO_RUN} --limit=${TARGET_HOST} -f 5
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Playbook '${PLAYBOOK_TO_RUN}' execution completed for '${TARGET_HOST}'"
        }
    }
}

