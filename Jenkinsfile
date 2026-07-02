pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', credentialsId: 'git-token', url: 'https://github.com/gajuk/Capstone-DotNET-Mongo-CI.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'dotnet build'
            }
        }
        stage('Unit Testing') {
            steps {
                echo 'dotnet test'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=noteApp \
                            -Dsonar.projectKey=noteApp '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
            }
        }
        
        stage('Build Image & Tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t gaju8734/noteapp:$IMAGE_TAG ."
                    }
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push gaju8734/noteapp:$IMAGE_TAG"
                    }
                }
            }
        }
        stage('Update Manifest File CD Repo') {
            steps {
                script {
                    cleanWs()
                    withCredentials([usernamePassword(credentialsId: 'git-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                            # Clone the CD Repo
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/gajuk/Capstone-DotNET-Mongo-CD.git
                            
                            # Update the tag in manifest
                            cd Capstone-DotNET-Mongo-CD
                            sed -i "s|gaju8734/noteapp:.*|gaju8734/noteapp:${IMAGE_TAG}|" Manifest/manifest.yaml
                            
                            # Confirm Changes
                            echo "Updated manifest file contents:"
                            cat Manifest/manifest.yaml
                            
                            # Commit and push the changes
                            git config user.name "Jenkins"
                            git config user.email "jenkins@example.com"
                            git add Manifest/manifest.yaml
                            git commit -m "Update image tag to ${IMAGE_TAG}"
                            git push origin master
                        '''
                    }
                    
                }
            }
        }
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'gajanankotpelliwar@gmail.com',
                from: 'gajanankotpelliwar@gmail.com',
                replyTo: 'gajanankotpelliwar@gmail.com',
                mimeType: 'text/html',
               
            )
        }
    }
}
}
