pipeline {
    agent any

    environment {
        // Set environment variables
        AZURE_SUBSCRIPTION_ID = 'your_azure_subscription_id'
        VAULT_ADDR = 'your_vault_address'
        VAULT_TOKEN = 'your_vault_token'
        CLIENT_ID = 'your_azure_client_id'
        RESOURCE_GROUP = 'your_azure_resource_group'
        VAULT_PATH = 'secret/azure_keys'
    }

    stages {
        stage('Check Expiry') {
            steps {
                script {
                    // Check the expiration date of the client ID keys
                    def expirationDate = sh(script: 'az ad app show --id your_azure_client_id --query "passwordCredentials[].endDate" --output tsv', returnStdout: true).trim()

                    // Calculate 15 days prior to expiration
                    def renewThreshold = sh(script: 'date -d "$expirationDate - 15 days" "+%Y-%m-%d"', returnStdout: true).trim()

                    // Get the current date
                    def currentDate = sh(script: 'date "+%Y-%m-%d"', returnStdout: true).trim()

                    // Compare dates and proceed to renewal if within the threshold
                    if (currentDate <= renewThreshold) {
                        echo "Renewing Azure Client ID keys..."
                        // Call Ansible playbook to renew keys
                        ansiblePlaybook(
                            playbook: 'your-ansible-playbook.yml',
                            inventory: 'path/to/inventory.ini',
                            credentialsId: 'your_ansible_cred_id'
                        )
                        
                        echo "Updating keys in Vault..."
                        // Call a script or use Vault CLI to update keys in the Vault
                        sh 'vault kv put secret/azure_keys azure_client_id_key=your_new_key_value azure_client_secret_key=your_new_secret_value'   // your-command-to-update-keys-in-vault
                    } else {
                        echo "Client ID keys are not within the renewal threshold."
                    }
                }
            }
        }
    }
}

