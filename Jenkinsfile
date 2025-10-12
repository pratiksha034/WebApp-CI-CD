pipeline {
    agent any
    parameters {
        string(name: 'TOMCAT_URL', defaultValue: 'http://localhost:8080', description: 'The base URL for the Tomcat Manager (e.g., http://192.168.1.100:8080)')
    }

    tools {
        jdk 'JDK21' // Replace with your configured JDK name
        maven 'M3'  // Replace with your configured Maven name
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

                // IMPORTANT: Use the actual artifact name from your pom.xml (WebApp-CI-CD-1.0-SNAPSHOT.war)
                def warFile = "WebApp-CI-CD-1.0-SNAPSHOT.war" 
                
                // IMPORTANT: Use the correct path (which should match the finalName in your pom.xml, which is WebApp-CI-CD)
                def contextPath = "/WebApp-CI-CD" 

                // Use 'withCredentials' to inject the secrets as environment variables
                withCredentials([
                    usernamePassword(
                        credentialsId: 'tomcat-deploy-creds', 
                        usernameVariable: 'admin', 
                        passwordVariable: 'pra@932214'
                    )
                ]) {
                    // Using 'bat' command for Windows environment.
                    // Access variables using %VARIABLE_NAME% syntax.
                    // Note: If 'curl' isn't available, you might need to use 'powershell' or install curl/wget.
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
