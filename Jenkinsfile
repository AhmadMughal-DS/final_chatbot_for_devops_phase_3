pipeline {
    // Use any available agent instead of requiring a specific label
    agent any
    
    environment {
        PROJECT_NAME = 'devops_chatbot_pipeline'
        DOCKER_COMPOSE_FILE = 'docker-compose'
        GITHUB_REPO = 'https://github.com/AhmadMughal-DS/final_chatbot_for_devops_phase_3'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace before we start
                cleanWs()
                
                // Fetch code from GitHub repository
                echo 'Fetching code from GitHub repository'
                sh    """ git clone "${GITHUB_REPO}" """
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building Docker containers'
                // Check Docker and Docker Compose installation
                sh 'docker --version'
                sh 'docker-compose --version || docker compose --version'
                
                // Build the Docker images - no sudo
                sh '''
                    # Use docker compose directly without fallback to sudo
                    docker-compose -p ${PROJECT_NAME} -build --no-cache
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application with Docker Compose'
                
                // Stop any existing containers with the same project name
                sh '''
                    docker-compose -p ${PROJECT_NAME} -f ${DOCKER_COMPOSE_FILE} down || \
                    echo "No existing containers to stop"
                '''
                
                // Start the containers in detached mode
                sh '''
                    docker-compose -p ${PROJECT_NAME} -f ${DOCKER_COMPOSE_FILE} up -d
                    echo "Deployment complete"
                '''
                
                // Verify that the containers are running
                sh 'docker-compose -p ${PROJECT_NAME} -f ${DOCKER_COMPOSE_FILE} ps'
            }
        }
        
        stage('Verify') {
            steps {
                echo 'Verifying the deployment'
                // Wait for application to be ready
                sh 'sleep 50'
                
                // Check if the container is running
                sh 'docker ps | grep devops_chatbot || echo "Container not found"'
                
                // Show logs for debugging
                sh 'docker logs devops_chatbot_backend || echo "Could not get container logs"'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running Frontend Chat Test'
                
                // Navigate to the cloned repository directory
                dir('final_chatbot_for_devops_phase_3') {
                    // Install Chrome and dependencies for Selenium
                    sh '''
                        # Update package list
                        apt-get update
                        
                        # Install Chrome dependencies
                        apt-get install -y wget gnupg2 software-properties-common
                        
                        # Add Google Chrome repository
                        wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add -
                        echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
                        
                        # Install Google Chrome
                        apt-get update
                        apt-get install -y google-chrome-stable
                        
                        # Install Python dependencies (assuming Python3 and pip3 are already installed)
                        pip3 install -r requirements.txt
                        
                        # Install additional dependencies for headless Chrome
                        apt-get install -y xvfb
                    '''
                    
                    // Wait for the application to be fully ready
                    sh 'sleep 30'
                    
                    // Run the frontend chat test
                    sh '''
                        echo "Running test_frontend_chat.py..."
                        
                        # Set display for headless Chrome (if needed)
                        export DISPLAY=:99
                        Xvfb :99 -screen 0 1024x768x24 &
                        
                        # Wait for Xvfb to start
                        sleep 5
                        
                        # Run the existing frontend chat test
                        python3 tests/test_frontend_chat.py
                        
                        echo "Frontend Chat Test completed!"
                    '''
                }
            }
        }
        
        stage('Auto-Stop After 5 Minutes') {
            steps {
                echo 'Setting up automatic container shutdown after 5 minutes'
                sh '''
                    echo "Containers will be stopped after 5 minutes..."
                    (sleep 300 && docker-compose -p ${PROJECT_NAME} -f ${DOCKER_COMPOSE_FILE} down) &
                    echo "Auto-stop scheduled!"
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace'
            
            // Show final container status
            sh '''
                echo "Final container status:"
                docker ps | grep devops_chatbot || echo "No chatbot containers running"
            '''
            
            deleteDir() // Clean workspace after build
        }
        success {
            echo '🎉 CI Pipeline completed successfully!'
            echo '✅ All stages passed including frontend tests'
            echo '🚀 Application is deployed and tested'
        }
        failure {
            echo '❌ CI Pipeline failed!'
            echo '🔍 Check the logs above for details'
            
            // Show application logs for debugging
            sh '''
                echo "Application logs for debugging:"
                docker logs devops_chatbot_backend || echo "Could not get backend logs"
            '''
            
            // You can add notification steps here (email, Slack, etc.)
        }
    }
}
