#!groovy​
// Jenkinsfile

// Test
// 192.168.1.98     home
// 10.0.0.194       office

// Production
// 192.168.1.100    home
// 10.0.0.203       office

/*
testSrv = "192.168.1.98"
prodSrv = "192.168.1.100"
*/

testSrv = "10.0.0.194"
prodSrv = "10.0.0.203"

def mvnHome

stage('Preparation') {

    node {
          // Get some code from a GitHub repository
          git 'https://github.com/ecs-digital-sunil-test-org/jpetstore-6.git'

          def pom = readMavenPom file: 'pom.xml'

            version = pom.version.replace("-SNAPSHOT", ".${currentBuild.number}")
          // version = pom.version
          artId = pom.artifactId

          // 6.0.3.41         jpetstore       41
          echo "version: ${version}    artifactId: ${artId}       buildNo: ${currentBuild.number}"

          nexusArtId = "${artId}-${version}"

          // Get the Maven tool.
          // ** NOTE: This 'M3' Maven tool must be configured
          // **       in the global configuration.
          mvnHome = tool 'Maven3'
    } // end-node
} // end-stage


stage('Build') {
    node {
        try {
            // Run the maven build
            sh "'${mvnHome}/bin/mvn' -Dclean package"
        } catch (error) {
        } finally {
            // failed build
        }
    } // end-node
} // end-stage


stage('Static Analysis') {
    node {
        try {
            withSonarQubeEnv {
                sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.1.1:sonar"
            }
        } catch (error) {
        } finally {
            // failed to send to Analysis
        }
    } // end-node
} // end-stage

stage('Unit Tests') {
    node {
        try {
            junit '**/target/surefire-reports/TEST-*.xml'
            archive 'target/*.jar'
        } catch (error) {
        } finally {
          // failed junit tests
        }
    } // end-node
} // end-stage


if  ("${env.BRANCH_NAME}" == "master") {

    stage('Publish to Artifactory') {
        node {
            try {
                    echo "Pushed war file to Artifactory: ${version}"
                    echo "SUNIL    -- ${nexusArtId}  --- ${artId}"
                    nexusPublisher nexusInstanceId: 'localNexus', nexusRepositoryId: 'releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: 'target/jpetstore.war']], mavenCoordinate: [artifactId: 'jpetstore', groupId: 'jpetstore', packaging: 'war', version: "${version}" ]]]
            } catch (error) {
            } finally {
                // failed to push to Nexus
            }
        } // end-node
    } // end-stage

    stage('Deploy to Testing Environment') {
        node {
            try {
                echo "SUNIL BRANCH_NAME : ${env.BRANCH_NAME} "
                // tomcat8 server
                // 192.168.1.98     home
                // 10.0.0.194       office
                if  ("${env.BRANCH_NAME}" == "master") {
                    sh "scp target/jpetstore.war root@${testSrv}:/var/lib/tomcat8/webapps/"
                    currentBuild.result = 'SUCCESS'
                }
            } catch (error) {
            } finally {
                // failed to push test environment
            }
        } // end-node
    } // end-stage

    stage('Deploy to Production Environment') {
        input 'Do you approve deployment to Production?'
        node {
            try {
                echo "SUNIL BRANCH_NAME : ${env.BRANCH_NAME} "
                // tomcat8 server
                // 192.168.1.100    home
                // 10.0.0.203       office
                if  ("${env.BRANCH_NAME}" == "master") {
                    sh "scp target/jpetstore.war root@${prodSrv}:/var/lib/tomcat8/webapps/"
                    currentBuild.result = 'SUCCESS'
                } else {
                    currentBuild.result = 'FAILURE'
                }
            } catch (error) {
            } finally {
                // failed to push to production
            }
        } // end-node
    } // end-stage
} // end-if
