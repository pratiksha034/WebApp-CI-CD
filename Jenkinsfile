// File: Jenkinsfile
pipeline {
    agent any
    
    // Define the Tomcat Manager URL as a build parameter
    parameters {
        string(name: 'TOMCAT_URL', defaultValue: 'http://localhost:8080', 
               description: 'The base URL for the Tomcat Manager (e.g., http://192.168.1.100:8080)')
    }

    // Ensure Maven and JDK tools are configured in Jenkins
    tools {
        jdk 'JDK21' // Make sure this name matches your Jenkins tool configuration
        maven 'M3'// Make sure this name matches your Jenkins tool configuration
    }

    stages {
        stage('Checkout') {
            steps {
                echo '1. Checking out code...'
                checkout scm
            }
        }

        stage('Build & Package WAR') {
            steps {
                echo '2. Compiling and packaging into WAR file...'
                // Build the project, skipping tests for speed
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Archive Artifact') {
            steps {
                echo '3. Archiving the WAR artifact...'
                // Archive the built WAR file for later use/download
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "4. Deploying WAR file to Tomcat at: ${params.TOMCAT_URL}"

                // Define WAR file details based on your pom.xml:
                def warFile = "WebApp-CI-CD-1.0-SNAPSHOT.war" 
                def contextPath = "/WebApp-CI-CD" 

                // Use 'withCredentials' to securely inject the Tomcat Manager secrets
                withCredentials([
                    usernamePassword(
                        credentialsId: 'tomcat-deploy-creds', 
                        usernameVariable: 'admin',    // Inject username as TOMCAT_USER
                        passwordVariable: 'pra@932214'     // Inject password as TOMCAT_PASS
                    )
                ]) {
                    // Use 'bat' command (for Windows) to execute curl for deployment.
                    // Variables are accessed using %VARIABLE_NAME%
                    bat """
                    echo Attempting deployment via curl...
                    curl -u %TOMCAT_USER%:%TOMCAT_PASS% -T target/${warFile} "${params.TOMCAT_URL}/manager/text/deploy?path=${contextPath}&update=true"
                    """
                }
            }
        }
        
        stage('Verification') {
            steps {
                echo "5. Deployment complete."
                echo "Check application at: ${params.TOMCAT_URL}${contextPath}"
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        failure {
            echo 'Deployment FAILED! Review console logs for Tomcat Manager error.'
        }
    }
}
