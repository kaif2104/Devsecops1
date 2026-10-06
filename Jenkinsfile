pipeline {
    agent any
    tools {
        jdk "jdk"
        maven "maven"
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kaif2104/Devsecops1.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Trivy FS') {
            steps {
                sh "trivy fs . --format table -o fs.html"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqubeServer') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
                          -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests"
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy -DskipTests"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t kaif03/full-stack ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html kaif03/full-stack:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push kaif03/full-stack:latest"
                    }
                }
            }
        }
    }  // Closing stages

    post {
        always {
            script {
                try {
                    def jobName = env.JOB_NAME
                    def buildNumber = env.BUILD_NUMBER
                    def pipelineStatus = currentBuild.result ?: 'SUCCESS'
                    pipelineStatus = pipelineStatus.toUpperCase()
                    
                    def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

                    def body = """
                    <body>
                        <div style="border: 2px solid ${bannerColor}; padding: 10px;">
                            <h3 style="color: ${bannerColor};">
                                Pipeline Status: ${pipelineStatus}
                            </h3>
                            <p>Job: ${jobName}</p>
                            <p>Build Number: ${buildNumber}</p>
                            <p>Status: ${pipelineStatus}</p>
                        </div>
                    </body>
                    """

                    emailext(
                        subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                        body: body,
                        to: 'mksocials21@gmail.com',
                        from: 'jenkins@example.com',
                        replyTo: 'jenkins@example.com',
                        mimeType: 'text/html'
                    )
                } catch (Exception e) {
                    echo "Email notification skipped or failed: ${e.getMessage()}"
                }
            }
        }
    }
}