pipeline {
    agent { label 'built-in' }               // run pipeline on master for build steps
    stages {
        
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                stash name: 'myapp-0.0.1-SNAPSHOT.jar', includes: 'target/*.jar'
            }
        }
        stage('Deploy') {
            agent { label 'ubuntu-worker' }      // switch to worker node for deploy
            steps {
                unstash 'myapp-0.0.1-SNAPSHOT.jar'
                sh '''
                  # stop existing app (if applicable)
                  pkill -f myapp-0.0.1-SNAPSHOT.jar || true

                  # deploy new JAR
                  echo "=== Starting deploy at $(date)" >> deploy.log
                  export JENKINS_NODE_COOKIE=dontKillMe && nohup java -jar target/myapp-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
                  echo "PID: $(pgrep -f myapp-0.0.1-SNAPSHOT.jar)" >> deploy.log
                '''
            }
        }
    }
    post {
        success {
            echo "Deployment succeeded for ${env.BUILD_NUMBER}"
        }
        failure {
            echo "Deployment FAILED for ${env.BUILD_NUMBER}"
        }
    }
}
