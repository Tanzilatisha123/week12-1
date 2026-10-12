pipeline {
    agent any

    stages {

       stage('Run Selenium Tests with pytest') {
    steps {
        echo "Running Selenium Tests using pytest"

        // Install Python dependencies with python3
        bat 'python3 -m pip install -r requirements.txt'

        // Start Flask app in background (empty title argument required!)
        bat 'start "" /B python3 app.py'

        // Wait a few seconds for the server to start (use full path to ping)
        bat 'C:\\Windows\\System32\\ping.exe 127.0.0.1 -n 5 > nul'

        // Run tests using pytest with python3
        bat 'python3 -m pytest -v'
    }
}


        stage('Build Docker Image') {
            steps {
                echo "Build Docker Image"
                bat 'docker build -t seleniumdemoapp:v1 .'
            }
        }

        stage('Docker Login') {
            steps {
                // Create this in Jenkins: Manage Jenkins > Credentials > Username with password
                // and give it the ID: dockerhub-creds
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    bat 'echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Push Docker Image to Docker Hub"
                bat 'docker tag seleniumdemoapp:v1 tanzila123/selenium:seleniumtestimage'
                bat 'docker push tanzila123/selenium:seleniumtestimage'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml --validate=false'
                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        always {
            // Stop the background Flask app so port 5000 is free for the next run
            bat(returnStatus: true, script: 'taskkill /F /IM python.exe')
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
