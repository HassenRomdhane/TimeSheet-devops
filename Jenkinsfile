pipeline { 
	environment 
	{
	    registry = "ziedcloud2020/devopsimage" 
	    registryCredential = 'dockerhub_id'
	    dockerImage = ''
	    NEXUS_VERSION = "nexus3"
	    NEXUS_PROTOCOL = "http"
	    NEXUS_URL = "localhost:8081"
	    NEXUS_REPOSITORY = "maven-releases"
	    NEXUS_CREDENTIAL_ID = "nexus_cred"
    	}
	agent any 
	stages {
	    
		stage('Cloning our Git') {
		steps { git 'https://github.com/zied-tech/TimeSheet-devops.git'
		}
		}
		stage('Mvn Clean') {
		steps {sh "mvn clean" }
		}
		stage('Mvn Compile') {
		steps {sh "mvn compile" }
		}
		stage('Mvn Install') {
		steps {sh "mvn install" }
		}
		stage("Publish to Nexus Repository Manager") 
		{
            	steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
    	}
		stage('push Artifact to Nexus') {
		steps {sh "mvn install" }
		}
		stage('Mvn test Sonar') {
		steps {sh "mvn sonar:sonar" }
		}
		stage('Mvn test JUNIT') {
		steps {sh "mvn test" }
		}
		stage('Mvn Package') {
		steps {sh "mvn package" }
		}
		stage('Building our image') {
		steps { script { dockerImage = docker.build registry + ":$BUILD_NUMBER" } }
		}
		
	stage('Cleaning up')
		{
    	steps
			{ 
    			sh "docker rmi $registry:$BUILD_NUMBER" 
    		} 
		}
	stage('email notifiication')
	{
		steps 
		{
        	emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
        	recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
        	subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
    	}
	}
}
}
