pipeline {
    agent any

    // 👇 This ensures Jenkins always exposes parameters
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    environment {
	ANSIBLE_REMOTE_HOST = 'ansible@192.168.1.200'
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
                sshagent (credentials: ['ansible_ssh_pass']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $ANSIBLE_REMOTE_HOST \
		    "cd /home/ansible/playbooks/jenkins_ansible_pipeline && ansible-playbook -i inventory.ini ping.yml --limit=${TARGET_HOST}
                    echo "Pinging ${TARGET_HOST} from inventory.ini"
<<<<<<< HEAD
=======
                    ansible-playbook ping.yml -i inventory.ini -f 5 --limit=${TARGET_HOST}
>>>>>>> 6eedc02a2c021290a158f58723faec70c90c4a69
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

