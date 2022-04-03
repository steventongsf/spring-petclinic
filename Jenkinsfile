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
        SONARSERVER = "sonarserver"
        SONARSCANNER = "sonarscanner"
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/steventongsf/spring-petclinic'
            }
        }
        stage('build') {
            steps {
                sh "mvn clean install -e -DskipTests -s settings.xml -Dcheckstyle.skip=true"
            }
            post {
                success {
                    echo "Archiving"
                    archiveArtifacts artifacts: '**/*.jar'
                }
            }
        }
        stage('checkstyle') {
            steps {
                sh "mvn test -e -s settings.xml -Dcheckstyle.skip=true -Dmaven.test.failure.ignore=true"
            }
        }
        stage('sonar') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=spring-petclinic \
                    -Dsonar.projectName=spring-petclinic \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/
                    -Dsonar.junit.reportsPaths=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPaths=target/jacoco.exec \
                    -Dsonar.verbose=true
                    '''
                    //-Dsonar.java.checkstyle.reportPaths=target/check-style.result.xml
                }
            }
        }

    }
}
