# AWS-secrets-manager
This is the automatically 


#!/bin/bash
# Clear existing .env file (if any)
> .env
echo "Cleared old .env file."
# Fetch the list of all secrets from AWS Secrets Manager
echo "Fetching list of secrets from AWS Secrets Manager..."
SECRET_LIST=$(aws secretsmanager list-secrets --query 'SecretList[*].Name' --output text)
if [ -z "$SECRET_LIST" ]; then
    echo "No secrets found in AWS Secrets Manager. Exiting."
    exit 1
fi
# Loop through each secret and fetch its values
for SECRET in $SECRET_LIST; do
    echo "Fetching secrets from $SECRET..."
    # Fetch secrets from AWS Secrets Manager
    if ! aws secretsmanager get-secret-value --secret-id "$SECRET" --query SecretString --output text > /tmp/secrets.json; then
        echo "Failed to fetch secrets from $SECRET. Skipping."
        continue
    fi
    # Parse the secrets and append them to the .env file
    echo "Parsing secrets from $SECRET..."
    if ! jq -r 'to_entries | map("\(.key)=\(.value|tostring)") | .[]' /tmp/secrets.json >> .env; then
        echo "Failed to parse secrets from $SECRET. Exiting."
        exit 1
    fi
    echo "Successfully fetched and parsed secrets from $SECRET."
done
# Clean up temporary JSON file
rm /tmp/secrets.json
echo "Temporary files cleaned up."
# Restart the Docker Compose application
echo "Restarting Docker Compose..."
docker-compose down && docker-compose up -d
echo "Application restarted with updated secrets."
