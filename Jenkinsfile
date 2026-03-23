node {
    // working on shreyas server
    // 🔹 Define variables
    def APP_NAME = "salary-api"
    def REPORT_DIR = "trivy-reports"
    

    try {
    
        stage('Clean Workspace') {
            deleteDir()
        }
        stage('Setup Maven') {
        script {
            def mvnHome = tool name: 'Maven3', type: 'maven'
            env.PATH = "${mvnHome}/bin:${env.PATH}"
            }
        }
        stage('Checkout Code') {
            git url: 'https://github.com/mukeshdevelp/ot-microservice-sarthi.git', branch: 'backend'
        }

        stage('Set Java 17 Environment') {
            script {
                env.JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
                env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
            }
        }

        stage('Verify Java') {
            sh '''
                echo "JAVA_HOME=$JAVA_HOME"
                which java
                which javac
                java -version
                javac -version
            '''
        }

        stage('Install Trivy (if not present)') {
            sh '''
                if ! command -v trivy &> /dev/null
                then
                    # sudo visudo
                    echo "Trivy not found. Installing..."
                    
                    sudo apt --fix-broken install -y
                    
                    sudo apt-get upgrade -y
                    
                    sudo apt-get install wget apt-transport-https gnupg lsb-release -y
                    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

                    echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
                    | sudo tee /etc/apt/sources.list.d/trivy.list

                    sudo apt-get update -y
                    sudo apt-get install -y trivy
                else
                    echo "Trivy already installed"
                fi
            '''
        }

        stage('Verify Trivy Installation') {
            sh '''
                trivy --version
            '''
        }

        stage('Build Maven Project') {
            sh '''
                ls
                pwd
                cd salary/salary-api/
                mvn clean package -DskipTests
            '''
        }

        stage('Create Report Directory') {
            sh """
                mkdir -p ${REPORT_DIR}
            """
        }

        stage('Trivy Filesystem Scan (Table Output)') {
            sh """
                trivy fs .
            """
        }

        stage('Generate JSON Report') {
            sh """
                trivy fs -f json -o ${REPORT_DIR}/trivy-report.json .
            """
        }

        stage('Generate SARIF Report') {
            sh """
                trivy fs --format sarif -o ${REPORT_DIR}/trivy-report.sarif .
            """
        }

        stage('Cleanup (Optional)') {
            sh "ls -lh ${REPORT_DIR}"
        }

    } catch (err) {
        // Slack notification for failure
        slackSend(
            channel: '#ci-operation-notifications',
            color: 'danger',
            message: """
            Build Failed

            Job: ${env.JOB_NAME}
            Build: #${env.BUILD_NUMBER}
            URL: ${env.BUILD_URL}
            """
        )
        error "Pipeline failed: ${err}"
    } finally {
        // Always archive artifacts
        archiveArtifacts artifacts: "${REPORT_DIR}/*", fingerprint: true
    }
}
