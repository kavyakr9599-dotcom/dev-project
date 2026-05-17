pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    parameters {
      string(name: 'sonar_IP', defaultValue: '52.91.52.149', description: 'IP of sonarqube')
      string(name: 'nexus_IP', defaultValue: '54.226.31.233', description: 'IP of nexus')
      string(name: 'deploy_IP', defaultValue:'98.93.165.85', description: 'IP of Deploy Server')  
    }
    environment {
      SONARQUBE_URL="http://${params.sonar_IP}:9000"
      SONARQUBE_TOKEN=credentials('Sonar-token')
      NEXUS_URL="http://${params.nexus_IP}:8081"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/kavyakr9599-dotcom/dev-project.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=MavenProject \
                    -Dsonar.host.url=http://52.91.52.149:9000 \
                    -Dsonar.login=$SONARQUBE_TOKEN
                    """
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Upload to Nexus') {
            steps {
                    sh '''
                    mvn clean deploy -DskipTests \
-DaltDeploymentRepository=maven-releases1::$NEXUS_URL/repository/maven-releases1
                    '''
            }
        }
           stage('Deploy') {
            steps {
                sshagent(credentials:['ec2-ssh']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no \
                        webapp/target/webapp.war \
                        ubuntu@${deploy_IP}:/tmp/webapp.war

                    ssh -o StrictHostKeyChecking=no ubuntu@${deploy_IP} "
                        sudo cp /tmp/webapp.war /home/ubuntu/tomcat/webapps/webapp.war &&
                        /home/ubuntu/tomcat/bin/shutdown.sh
                        /home/ubuntu/tomcat/bin/startup.sh
                    "
                    '''
                }
            }
        }
    }
}
