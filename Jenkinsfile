pipeline {
    agent any

    tools {
        nodejs 'node21'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/awais684/3-tier-devops-project.git'
            }
        }

        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t awais684/camp:latest ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html adijaiswal/camp:latest"
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push awais684/camp:latest"
                    }
                }
            }
        }

        stage('Docker Deploy To Dev') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 awais684/camp:latest"
                    }
                }
            }
        }

    }

    post {

        success {
            mail(
                to: 'prodteam@yourcompany.com',
                subject: "✅ Dev Pipeline Passed — Ready for Prod Deploy #${env.BUILD_NUMBER}",
                body: """\
Hi Prod Team,
...
				""".stripIndent()
            )
        }

        failure {
            mail(
                to: 'devteam@yourcompany.com',
                subject: "❌ Dev Pipeline FAILED — Build #${env.BUILD_NUMBER}",
                body: """\
Hi Dev Team,
...
				""".stripIndent()
            )
        }

    }  

}