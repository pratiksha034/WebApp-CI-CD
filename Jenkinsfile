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
        maven 'M3'  // Make sure this name matches your Jenkins tool configuration
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

                // --- FIX: All Groovy variable declarations and logic must be in a script block ---
                script {
                    // FIX 1: Corrected WAR file name based on Maven output
                    def warFile = "WebApp-CI-CD.war" 
                    def contextPath = "/WebApp-CI-CD" 

                    // Use 'withCredentials' to securely inject the Tomcat Manager secrets
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'tomcat-deploy-creds',
                            // FIX 2: Variables must match what the 'bat' command uses
                            usernameVariable: 'TOMCAT_USER',    
                            passwordVariable: 'TOMCAT_PASS'     
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
                // --- End of script block ---
            }
        }
        
        stage('Verification') {
            // Context path is now hardcoded in the echo for simplicity, as it's outside the script block scope
            steps {
                echo "5. Deployment complete."
                echo "Check application at: ${params.TOMCAT_URL}/WebApp-CI-CD"
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
