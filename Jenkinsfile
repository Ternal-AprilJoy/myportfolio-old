pipeline {
    agent any

    environment {
        STORAGE_ACCOUNT = 'apriljoystore2026'
        FILE_SHARE      = 'webcontent'
       ACI_URL = 'http://apriljoynginx2026.centralindia.azurecontainer.io'

        WS1 = '10.0.11.56'
        WS2 = '10.0.12.108'

        SSH_KEY = '/var/lib/jenkins/.ssh/aws.pem'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

stage('Deploy Staging') {
    steps {
        sh '''
        rm -rf deploy
        mkdir deploy

        cp index.html deploy/ 2>/dev/null || true
        cp *.css deploy/ 2>/dev/null || true
        cp *.js deploy/ 2>/dev/null || true
        cp *.ttf deploy/ 2>/dev/null || true

        cp -r images deploy/ 2>/dev/null || true

        AZ_KEY=$(cat /var/lib/jenkins/secrets/azure.key)

        /usr/bin/az storage file upload-batch \
          --account-name "$STORAGE_ACCOUNT" \
          --account-key "$AZ_KEY" \
          --destination "$FILE_SHARE" \
          --source deploy
        '''
    }
}

        stage('Validate Staging') {
            steps {
                sh '''
                echo "Checking ACI availability..."

                curl -f "$ACI_URL"

                echo "Checking page content..."

                curl -s "$ACI_URL" | grep -i "April"

                echo "Staging validation successful"
                '''
            }
        }

        stage('Deploy WS1') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no \
                -i $SSH_KEY \
                index.html ubuntu@$WS1:/tmp/index.html

                ssh -o StrictHostKeyChecking=no \
                -i $SSH_KEY \
                ubuntu@$WS1 \
                "sudo cp /tmp/index.html /var/www/html/index.html"

                echo "WS1 deployment complete"
                '''
            }
        }

        stage('Deploy WS2') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no \
                -i $SSH_KEY \
                index.html ubuntu@$WS2:/tmp/index.html

                ssh -o StrictHostKeyChecking=no \
                -i $SSH_KEY \
                ubuntu@$WS2 \
                "sudo cp /tmp/index.html /var/www/html/index.html"

                echo "WS2 deployment complete"
                '''
            }
        }

    }

    post {
        success {
            echo 'Deployment completed successfully'
        }

        failure {
            echo 'Pipeline stopped. Production deployment blocked.'
        }
    }
}
