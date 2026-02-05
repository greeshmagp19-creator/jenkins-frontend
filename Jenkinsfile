pipeline {
    agent any
 
    environment {
        AWS_REGION = "eu-north-1"
        S3_BUCKET = "demo12091923"
        CLOUDFRONT_DISTRIBUTION_ID = "E202ALDHVTXK81"
    }
 
    options {
        timestamps()
    }
 
    stages {
 
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token-creds',
                    url: 'https://github.com/greeshmagp19-creator/jenkins-frontend.git'
            }
        }
 
        stage('Check Node & NPM') {
            steps {
                sh '''
                  node -v
                  npm -v
                '''
            }
        }
 
        stage('Install Dependencies') {
            steps {
                sh '''
                  rm -rf node_modules package-lock.json
                  npm cache clean --force
                  npm install
                '''
            }
        }
 
        stage('Build Frontend') {
            steps {
                sh '''
                  npm run build
                '''
            }
        }
 
        stage('Verify Build Output') {
            steps {
                sh 'ls -la build/ '
            }
        }
 
        stage('Deploy to S3') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                      aws s3 sync dist/ s3://${S3_BUCKET}/ --delete
                    '''
                }
            }
        }
 
        stage('Invalidate CloudFront Cache') {
            when {
                expression { env.CLOUDFRONT_DISTRIBUTION_ID != "E202ALDHVTXK81" }
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''
                      aws cloudfront create-invalidation \
                        --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \
                        --paths "/*"
                    '''
                }
            }
        }
    }
 
    post {
        success {
            echo "✅ Frontend deployed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}