pipeline {
    agent any
    environment {
        SONARQUBE_URL = 'http://16.16.128.85:9000/'  // Update with your SonarQube server IP
        SONARQUBE_SECRET_FILE = credentials('jenkins_access')  // Secret file in Jenkins credentials
        GITHUB_ACCESS_TOKEN = credentials('github_access')  // GitHub token in Jenkins credentials
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Using GitHub access token for authentication
                    withCredentials([string(credentialsId: 'github-access-token', variable: 'GITHUB_TOKEN')]) {
                        sh 'git config --global url."https://${GITHUB_TOKEN}@github.com".insteadOf "https://github.com"'
                        git branch: 'main', url: 'https://github.com/abindani/git_batchdemo.git'
                    }
                }
            }
        }

        stage('Set up JDK') {
            steps {
                sh 'sudo apt update && sudo apt install -y openjdk-17-jdk'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'sonarqube-secret-file', variable: 'SONARQUBE_FILE')]) {
                        // Run SonarQube analysis using the secret file
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=myproject \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                            -Dsonar.login=$(cat ${SONARQUBE_FILE})
                        '''
                    }
                }
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy Application with Nginx') {
            steps {
                script {
                    sh '''
                        # Stop existing application (if running)
                        sudo pkill -f "java -jar /home/ubuntu/app.war" || true
                        
                        # Copy WAR file to deployment location
                        sudo cp target/*.war /home/ubuntu/app.war
                        
                        # Start new application
                        nohup java -jar /home/ubuntu/app.war > /dev/null 2>&1 &
                        
                        # Configure Nginx (if not already configured)
                        echo '
                        server {
                            listen 80;
                            server_name localhost;
                            
                            location / {
                                proxy_pass http://localhost:8080;
                                proxy_set_header Host $host;
                                proxy_set_header X-Real-IP $remote_addr;
                                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                            }
                        }' | sudo tee /etc/nginx/sites-available/default

                        sudo systemctl restart nginx
                    '''
                }
            }
        }
    }
}
