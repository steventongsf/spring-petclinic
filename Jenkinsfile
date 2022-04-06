def COLOR_MAP = [
    "SUCCESS": 'good',
    "FAILURE": 'danger',
]
pipeline {
    agent any
    tools {
        jdk "java-jdk-17"
        maven "maven-3"
    }
    environment {
        //JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
        SNAP_REPO = "sfhuskie-snapshot"
        NEXUS_LOGIN = "nexus-admin-user"
        NEXUS_USER = "admin"
        NEXUS_PASS = "nexus"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUSIP = "nexus01"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "sfhuskie-maven-group"
        RELEASE_REPO = "sfhuskie-release"
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
                    sh '''${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=spring-petclinic \
                    -Dsonar.projectName=spring-petclinic \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/main/java \
                    -Dsonar.tests=src/test/java \
                    -Dsonar.java.binaries=target/test-classes/**/* \
                    -Dsonar.junit.reportsPaths=target/surefire-reports \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.dynamicAnalysis=reuseReports \
                    -Dsonar.verbose=true
                    '''
                    //-Dsonar.java.checkstyle.reportPaths=target/check-style.result.xml
                }
            }
        }
        stage("quality gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("upload") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: "nexus3",
                    protocol: "http",
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: "com.sfhuskie.demo",
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [
                            artifactId: "spring-petclinic",
                            classifier: "",
                            file: "target/spring-petclinic-3.0.0-SNAPSHOT.jar",
                            type: "jar"
                        ]
                    ]
                )
            }
        }
    }
    post {
        always {
            slackSend channel: "#jenkins-cicd",
            color: COLOR_MAP[currentBuild.currentResult],
            tokenCredentialId: "slack-jenkins-cicd",
            message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nSee: ${env.BUILD_URL}"
        }
    }
}
