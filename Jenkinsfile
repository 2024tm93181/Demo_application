pipeline {
    agent { label 'DevServer' }  // Master server for main pipeline execution

    parameters {
        choice choices: ['staging', 'production'], name: 'select_environment'
    }

    environment {
        NAME = "Himanshu"
    }

    tools {
        maven 'maven'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    file = load "script.groovy"
                    file.hello()
                }
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Test') { 
            parallel {
                stage('Test A') {
                    agent { label 'DevServer' }
                    steps {
                        echo "This is Test A"
                        sh "mvn test"
                    }
                }
                stage('Test B') {
                    agent { label 'DevServer' }
                    steps {
                        echo "This is Test B"
                        sh "mvn test"
                    }
                }
            }
            post {
                success {
                    dir("webapp/target/") {
                        stash name: "maven-build", includes: "*.war"
                    }
                }
            }
        }
        stage('Deploy to Develop') {
            when { branch 'develop' }  
            agent { label 'DevServer' }  // Deploy to Dev server
            steps {
                dir("/var/www/html") {
                    unstash "maven-build"
                }
                sh """
                cd /var/www/html/
                jar -xvf webapp.war
                """
            }
        }

        stage('Deploy to Staging') {
            when { branch 'Staging' }  
            agent { label 'StagingServer' }  // Deploy to staging server
            steps {
                dir("/var/www/html") {
                    unstash "maven-build"
                }
                sh """
                cd /var/www/html/
                jar -xvf webapp.war
                """
            }
        }

        stage('Deploy to Production') {
            when { branch 'master' }
            agent { label 'ProdServer' }  // Deploy to production server
            steps {
                timeout(time:5, unit:'DAYS') {
                    input message: 'Deployment approved?'
                }
                dir("/var/www/html") {
                    unstash "maven-build"
                }
                sh """
                cd /var/www/html/
                jar -xvf webapp.war
                """
            }
        }
    }
}
