pipeline {
    agent any

    parameters {
        string(
            name: 'BACKUP_CLIENT',
            defaultValue: '',
            description: 'Backup client hostname'
        )
    }

    environment {
        ANSIBLE_PLAYBOOK = "ansible/playbook.yml"
        INVENTORY_FILE   = "ansible/inventory.ini"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Validate Parameter') {
            steps {
                script {
                    if (!params.BACKUP_CLIENT?.trim()) {
                        error "BACKUP_CLIENT parameter is required!"
                    }
                }
            }
        }

        stage('Create Inventory') {
            steps {
                sh """
                echo "[backup_client]" > ${INVENTORY_FILE}
                echo "${BACKUP_CLIENT}" >> ${INVENTORY_FILE}
                """
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sh """
                ansible-playbook -i ${INVENTORY_FILE} ${ANSIBLE_PLAYBOOK}
                """
            }
        }
    }

    post {
        success {
            echo "Backup automation completed successfully for ${BACKUP_CLIENT}"
        }
        failure {
            echo "Backup automation failed for ${BACKUP_CLIENT}"
        }
    }
}