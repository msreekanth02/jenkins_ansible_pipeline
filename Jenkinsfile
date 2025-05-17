pipeline {
    agent any

    // 👇 This ensures Jenkins always exposes parameters
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
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_FORCE_COLOR = 'True'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Ansible (if missing)') {
            steps {
                sh '''
                if ! command -v ansible-playbook >/dev/null 2>&1; then
                    echo "Installing Ansible..."
                    sudo yum update && sudo yum install -y ansible
                fi
                '''
            }
        }

        stage('Run Ansible Ping') {
            steps {
                sshagent (credentials: ['ansible_ssh_user']) {
                    sh '''
                    echo "Pinging ${TARGET_HOST} from inventory.ini"
                    ansible-playbook -i inventory.ini ping.yml --limit=${TARGET_HOST}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Ansible ping task completed for ${TARGET_HOST}"
        }
    }
}

