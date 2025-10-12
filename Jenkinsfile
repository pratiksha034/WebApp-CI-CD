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
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Archive Artifact') {
            steps {
                echo '3. Archiving the WAR artifact...'
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "4. Deploying WAR file to Tomcat at: ${params.TOMCAT_URL}"

                // --- FIX 1: Use 'script' block for Groovy logic and variable declarations ---
                script {
                    // Define WAR file details (now inside the script block)
                    def warFile = "WebApp-CI-CD-1.0-SNAPSHOT.war" 
                    def contextPath = "/WebApp-CI-CD" 

                    // --- FIX 2: Correctly map the credential variables to be used in the 'bat' step ---
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'tomcat-deploy-creds',
                            usernameVariable: 'admin',    // Inject username as TOMCAT_USER
                            passwordVariable: 'pra@932214'     // Inject password as TOMCAT_PASS
                        )
                    ]) {
                        // The 'bat' command uses %TOMCAT_USER% and %TOMCAT_PASS% which now match above
                        bat """
                        echo Attempting deployment via curl...
                        curl -u %TOMCAT_USER%:%TOMCAT_PASS% -T target/${warFile} "${params.TOMCAT_URL}/manager/text/deploy?path=${contextPath}&update=true"
                        """
                    }
                }
                // --- End of 'script' block ---
            }
        }
        
        stage('Verification') {
            // Note: The contextPath variable is not accessible outside the 'script' block, 
            // so we hardcode the URL here for simplicity.
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
