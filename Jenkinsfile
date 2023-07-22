pipeline {
    agent any
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK17'
    }
    environment {
       TOMCAT_CREDS = credentials('Tomcat_Creds') //Tomcat Credentials
       NEXUS_VERSION = "nexus3"
       NEXUS_PROTOCOL = "http"
       NEXUS_URL = "34.16.128.227:8081"
       NEXUS_REPO = "allrepo-mixed"
    }
    stages {
        stage ("clone") {
            steps {
                //clone the repo, first gor to pipeline syntax and generate the snippet
                git credentialsId: 'git-jenkins-creds', url: 'https://github.com/HarithaDevops1/spring3-mvc-maven-xml-hello-world.git'
            }
        }
        stage ("Build") {
            steps {
                sh "mvn --version"
                sh "mvn clean package -Dmaven.test.failure.ignore=true"
                //-Dcheckstyle.skip
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
                }
            }
        }
    stage ("Nexus") {
        steps {
            script {
                pom = readMavenPom file: "pom.xml" // reading pom.xml file and saving into pom
                files = findFiles(glob: "target/*.${pom.packaging}")
                //the below command is just for verification, no use
                echo "echo ${files[0].name} ${files[0].path} ${files[0].directory} ${files[0].length} ${files[0].lastModified}"
                // get the path location
                artifactPath = files[0].path;
                //validating if file exists a d storing in a variable
                artifactExists = fileExists artifactPath;
                //echo "If artifact exists"
                echo "${artifactExists}"
                //code
                if (artifactExists) {
                  //run this piece of code
                  echo "************ Artifact is available, going to deploy to Nexus"
                  echo "File is : ${artifactPath} , Package is : ${pom.packaging}, Version is : ${pom.version}, GroupID is : ${pom.groupId}"
                  // We need to deploy to Nexus, using a plugin 'Nexus Artifact Uploader'
                  // For documentation https://plugins.jenkins.io/nexus-artifact-uploader/
                   nexusArtifactUploader (
                    nexusVersion: "${env.NEXUS_VERSION}",
                    protocol: "${env.NEXUS_PROTOCOL}",
                    nexusUrl: "${env.NEXUS_URL}", // env. | params. |pipeline.pararms.
                    groupId: "${pom.groupId}",
                    //version: "${pom.version}",
                    version: "${BUILD_NUMBER}",
                    repository: "$env.NEXUS_REPO",
                    credentialsId: "Nexus-creds",
                               artifacts: [
                                  [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                  ]
                               ]
                )
                }
                else {
                  error "******** ${artifactPath} is not available"
                }
        }
        }
        }
        stage ("Deploy to tomcat") {
            steps {
                // In tomcat, tomcatusers.xml file we need to mention <user username="haritha" password="Santu@0908" roles="manager-gui,admin-gui,manager-script,admin-script,manager,manger-jmx,manager-status"/>
                // curl commands
                //https://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html#Deploy_A_New_Application_Archive_(WAR)_Remotely
                sh "curl -v -u ${TOMCAT_CREDS_USR}:${TOMCAT_CREDS_PSW} -T /var/lib/jenkins/workspace/allpipeline/target/spring3-mvc-maven-xml-hello-world-1.0-SNAPSHOT.war 'http://34.125.233.21:8080/manager/text/deploy?path=/spring-hello&update=true'"
            }
        }
    }
}
