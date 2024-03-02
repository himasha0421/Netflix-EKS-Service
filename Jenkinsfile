pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Folder Check'){
            steps {
                sh '''
                    ls -a
                '''
            }
        }

        // sonarqube static code analysis
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix \
                    -Dsonar.sources=. \
                    -Dsonar.scm.disabled=True'''
                }
            }
        }

        // code quality gate (pass if only quality standards meets)
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-secret'
                }
            }
        }
        // install dependancies
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        // ------- Security Clearence Stages

        // OWASP dependency checker
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OWASP-Checker'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        // Trivy File system scanner
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        // Docker build and push to docker hub
        stage("Docker Build & Push"){
            steps{
                script{
                    withCredentials([string( credentialsId: 'tmdb-token', variable: 'TMDB_AUTH_TOKEN' )]) {
                        withDockerRegistry(credentialsId: 'docker-login', toolName: 'docker'){   
                            sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_AUTH_TOKEN} -t netflix ."
                            sh "docker tag netflix himashaj96/netflix:latest "
                            sh "docker push himashaj96/netflix:latest "
                        }
                    }
                }
            }
        }

        // Image Scan
        stage("TRIVY"){
            steps{
                sh "trivy image himashaj96/netflix:latest > trivyimage.txt" 
            }
        }
    }

    // email notification step
    post{
        always {
            emailext attachLog: true,
                    subject: "${currentBuild.result}",
                    body: "Project: ${env.JOB_NAME}<br/>" +
                            "Build Number: ${env.BUILD_NUMBER}<br/>" +
                            "URL: ${env.BUILD_URL}<br/>",
                    to: "himashamora15@gmail.com",
                    attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}