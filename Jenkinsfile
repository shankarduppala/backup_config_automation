pipeline {
    agent any

    parameters {
        string(
            name: 'BACKUP_CLIENT',
            description: 'AWS Ubuntu EC2 public IP or hostname'
        )
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        INVENTORY_FILE = 'inventory.ini'
        ANSIBLE_PLAYBOOK = 'playbook.yaml'
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

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Create Ansible Inventory') {
            steps {
                sh '''
                cd ansible
                echo "[backup_client]" > inventory.ini
                echo "${BACKUP_CLIENT}" >> inventory.ini
                cat inventory.ini
                '''
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
                    sh '''
                    ssh -o StrictHostKeyChecking=no \
                        -i ${SSH_KEY} \
                        ubuntu@${BACKUP_CLIENT} \
                        "echo SSH connection successful"
                    '''
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
                    sh '''
                    cd ansible
                    ansible-playbook \
                      -i inventory.ini \
                      --private-key ${SSH_KEY} \
                      -u ubuntu \
                      playbook.yaml
                    '''
                }
            }
        }
    }

    post {
    success {
        emailext(
            subject: "SUCCESS: Backup Pipeline for ${BACKUP_CLIENT}",
            body: """
            <h3>Backup completed successfully</h3>
            <p><b>Client:</b> ${BACKUP_CLIENT}</p>
            <p><b>Job:</b> ${env.JOB_NAME}</p>
            <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
            <p><b>URL:</b> <a href='${env.BUILD_URL}'>Build Link</a></p>
            """,
            to: "shankarduppala320@gmail.com"
        )
    }

    failure {
        emailext(
            subject: "FAILED: Backup Pipeline for ${BACKUP_CLIENT}",
            body: """
            <h3>Backup failed ❌</h3>
            <p><b>Client:</b> ${BACKUP_CLIENT}</p>
            <p><b>Check logs:</b> <a href='${env.BUILD_URL}'>Build Link</a></p>
            """,
            to: "yshankarduppala320@gmail.com"
        )
    }
}

    
}
