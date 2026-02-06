pipeline{
    agent any
    
    environment{
        AWS_REGION = "eu-north-1"
        S3_BUCKET = "demo12091923"
        CLOUDFRONT_DISTRIBUTION_ID = "E202ALDHVTXK81"
    }
    
    options {
        timestamps()
    }
    
    stages{
        
        stage('checkout code') {
            steps {
                git branch: 'main',
                credentialsId: 'github-token-creds',
                url: 'https://github.com/greeshmagp19-creator/jenkins-frontend.git'
            }
        }
        
        stage('check node & NPM') {
            steps{
                sh '''
                  node -v
                  npm -v
                '''
            }
        }
        
        stage ('build') {
            steps {
                sh '''
                   sh 'npm run build'
                 '''
            }
        }
        stage ('verify build output') {
            steps {
                 sh 'ls -la dist'
            }
        }
        stage('deploy to s3'){
            
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh 'aws s3 sync dist/ s3://${S3_BUCKET}/ --delete'
                }
                
            }
        }
        stage('invalidate cloudfront'){
            when {
                expression { env.CLOUDFRONT_DISTRIBUTION_ID != "E202ALDHVTXK81" }
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh '''aws cloudfront create-invalidation \\
                        --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \\
                        --paths "/*"'''
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