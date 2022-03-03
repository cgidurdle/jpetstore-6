#!groovy​

testSrv = "10.0.0.194"
prodSrv = "10.0.0.203"

def mvnHome

stage('Preparation') {

    node {
          git 'https://github.com/ecs-digital-sunil-test-org/jpetstore-6.git'
          def pom = readMavenPom file: 'pom.xml'
            version = pom.version.replace("-SNAPSHOT", ".${currentBuild.number}")
          artId = pom.artifactId
          echo "version: ${version}    artifactId: ${artId}       buildNo: ${currentBuild.number}"
          nexusArtId = "${artId}-${version}"
          mvnHome = tool 'Maven3'
    }
} 


stage('mvn{
    node {
        try {
            sh "'${mvnHome}/bin/mvn' -Dclean package"
        } catch (error) {
        } finally {
        }
    } 
} 


stage('sqube') {
    node {
        try {
            withSonarQubeEnv {
                sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.1.1:sonar"
            }
        } catch (error) {
        } finally {
                    }
    } 
} 

stage('sunfire') {
    node {
        try {
            junit '**/target/surefire-reports/TEST-*.xml'
            archive 'target/*.jar'
        } catch (error) {
        } finally {
           }
    } 
} 


if  ("${env.BRANCH_NAME}" == "master") {

    stage('npub') {
        node {
            try {
                    echo "Pushed war file to Artifactory: ${version}"
                    echo "SUNIL    -- ${nexusArtId}  --- ${artId}"
                    nexusPublisher nexusInstanceId: 'localNexus', nexusRepositoryId: 'releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: 'target/jpetstore.war']], mavenCoordinate: [artifactId: 'jpetstore', groupId: 'jpetstore', packaging: 'war', version: "${version}" ]]]
            } catch (error) {
            } finally {
                            }
        } 
    } 

    stage('testing env') {
        node {
            try {
                echo "SUNIL BRANCH_NAME : ${env.BRANCH_NAME} "
                if  ("${env.BRANCH_NAME}" == "master") {
                    sh "scp target/jpetstore.war root@${testSrv}:/var/lib/tomcat8/webapps/"
                    currentBuild.result = 'SUCCESS'
                }
            } catch (error) {
            } finally {
            }
        } // end-node
    } // end-stage

    stage('prod env') {
        input 'Do you approve ?'
        node {
            try {
                echo "SUNIL BRANCH_NAME : ${env.BRANCH_NAME} "
                if  ("${env.BRANCH_NAME}" == "master") {
                    sh "scp target/jpetstore.war root@${prodSrv}:/var/lib/tomcat8/webapps/"
                    currentBuild.result = 'SUCCESS'
                } else {
                    currentBuild.result = 'FAILURE'
                }
            } catch (error) {
            } finally {
            }
        } 
    } 
} 
