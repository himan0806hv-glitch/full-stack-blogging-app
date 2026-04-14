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
                git branch: 'main', url: 'https://github.com/himan0806hv-glitch/full-stack-blogging-app.git'
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
        stage('Copy Dependencies') {
    steps {
        sh 'mvn dependency:copy-dependencies -DoutputDirectory=target/dependency'
    }
}
        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonarqubeserver') {
            sh '''$SCANNER_HOME/bin/sonar-scanner \
                  -Dsonar.projectName=Blogging-app \
                  -Dsonar.projectKey=Blogging-app \
                  -Dsonar.java.binaries=target/classes \
                  -Dsonar.java.libraries=target/dependency/*.jar \
                  -Dsonar.exclusions=target/**,fs.html'''
        }
    }
}
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                sh "docker build -t himan0812/blogging-app:latest ."
                }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html himan0812/blogging-app:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh "docker push himan0812/blogging-app:latest"
                }
                }
            }
        }
        stage('K8s Deploy') {
            steps {
               withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://16264535E6F5946E107B618B1CD20BBA.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f deployment-service.yml"
                    sleep 20
                }
            }
        }
        stage('Verify Deployment') {
            steps {
               withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://16264535E6F5946E107B618B1CD20BBA.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl get pods"
                    sh "kubectl get service"
                }
            }
        }
        
    }  // Closing stages
}  // Closing pipeline
post {
    always {
        script {
            // Get job name, build number, and pipeline status
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            pipelineStatus = pipelineStatus.toUpperCase()
            
            // Set the banner color based on the status
            def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

            // HTML body for the email
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

            // Send email notification
            emailext(
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                body: body,
                to: 'himan0806.hv.com',
                from: 'himanshu0806.hv.com',
                replyTo: 'himanshu0806.hv.com',
                mimeType: 'text/html'
            )
        }
    }
}

