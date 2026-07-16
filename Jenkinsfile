pipeline {
    agent any
    tools {
        jdk 'jdk'
        maven 'maven'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()  // Clean the workspace before starting the builddjhdjdjkd
            }
        }

        // Git Checkout
        stage('Git Checkout') {
            steps {
                git branch: 'master', changelog: false, poll: true, url: 'https://github.com/anantha3267/Java-CICD.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }


        // Compile Stage
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        // Run Test Cases
        stage('Test Cases') {
            steps {
                sh "mvn test"
            }
        }

        // SonarQube Analysis
        stage('Sonar analysis') {
            steps {
                script {
                    // Use the SonarQube environment
                    withSonarQubeEnv('SonarQube') {
                        sh "mvn clean package"
                        sh ''' mvn sonar:sonar \
                                -Dsonar.projectName=Scoreme \
                                -Dsonar.java.binaries=. \
                                -Dsonar.projectKey=Scoreme '''
                    }
                }
            }
        }

        //Wait for SonarQube Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('Cyclomatic Complexity') {
    steps {
        // Run Lizard and generate the report
        sh 'lizard src/main/java/com/example/*.java src/test/java/com/example/*.java --html > lizard-report.html'
    }
}

        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'Dependency check',
                 stopBuild: true // Stop on any critical vulnerabilities
                 
            }     
        }
        // JaCoCo Code Coverage Report
        stage('JaCoCo Report') {
            steps {
                // Run JaCoCo and generate the report
                sh "mvn clean test jacoco:report"
            }
        }

    }
    post {
    always {
        // Publish the Dependency-Check HTML report
        publishHTML(
            target: [
                reportName: 'Dependency-Check Report',
                reportDir: '', // No need to specify a directory if the report is in the workspace root
                reportFiles: 'dependency-check-report.html', // The HTML report generated
                keepAll: true
            ]
        )
        // Publish the Cyclomatic Complexity HTML report in Jenkins
        publishHTML(target: [
            reportName: 'Cyclomatic Complexity',
            reportDir: '.',
            reportFiles: 'lizard-report.html'
        ])

        // Publish the JaCoCo HTML report in Jenkins
        publishHTML(
            target: [
                reportName: 'JaCoCo Report',
                reportDir: 'target/site/jacoco',
                reportFiles: 'index.html'
            ]
        )
    }

        success {
            mail to: 'ananthamchiranjeevi@gmail.com',
                 subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """Build was successful:

                 Job: ${env.JOB_NAME}
                 Build Number: ${env.BUILD_NUMBER}
                 Build URL: ${env.BUILD_URL}"""
        }
        failure {
            mail to: 'ananthamchiranjeevi@gmail.com',
                 subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """Build failed:

                 Job: ${env.JOB_NAME}
                 Build Number: ${env.BUILD_NUMBER}
                 Build URL: ${env.BUILD_URL}"""
        }
}

}
