pipeline {
    agent any

    parameters {
        string(
            name: 'BACKUP_CLIENT',
            description: 'AWS Ubuntu EC2 public IP or hostname for backup'
        )
    }

    environment {
        INVENTORY_FILE = 'inventory.ini'
        ANSIBLE_PLAYBOOK = 'playbook.yml'
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.BACKUP_CLIENT?.trim()) {
                        error "BACKUP_CLIENT parameter cannot be empty"
                    }
                }
            }
        }

        stage('Create Ansible Inventory') {
            steps {
                sh """
                echo "[backup_client]" > ${INVENTORY_FILE}
                echo "${BACKUP_CLIENT}" >> ${INVENTORY_FILE}
                """
            }
        }

        stage('Test SSH Connectivity') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ubuntu-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no \
                        -i $SSH_KEY \
                        ubuntu@${BACKUP_CLIENT} \
                        "echo SSH connection successful"
                    """
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ubuntu-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh """
                    ansible-playbook \
                      -i ${INVENTORY_FILE} \
                      --private-key $SSH_KEY \
                      -u ubuntu \
                      ${ANSIBLE_PLAYBOOK}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Backup pipeline executed successfully for ${BACKUP_CLIENT}"
        }
        failure {
            echo "Backup pipeline failed for ${BACKUP_CLIENT}"
        }
    }
}
