pipeline {
    agent any
    tools {
        jdk "java-jdk-17"
        maven "maven-3"
    }
    environment {
        //JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
        SNAP_REPO = "sfhuskie-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "nexus"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUSIP = "nexus01"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "sfhuskie-maven-group"
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/steventongsf/spring-petclinic'
            }
        }
        stage('build') {
            steps {
                sh "mvn clean deploy -e -DskipTests -s settings.xml -Dcheckstyle.skip=true"
            }
        }
        stage('test') {
            steps {
                sh "mvn test -e  -Dcheckstyle.skip=true"
            }
        }
    }
}
