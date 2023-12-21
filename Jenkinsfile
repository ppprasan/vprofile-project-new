def buildNumber = Jenkins.instance.getItem('bean-profile').lastSuccessfulBuild.number

pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "JDK8"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
		    NEXUS_USER = 'admin'
		    NEXUS_PASS = 'password'
		    RELEASE_REPO = 'vprofile-release'
		    CENTRAL_REPO = 'vpro-maven-central'
		    NEXUSIP = '172.31.10.247'
		    NEXUSPORT = '8081'
		    NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexus-creds'
	    SONARSERVER = 'sonar_server'
	    SONARSCANNER = 'SONAR'
	    NEXUSPASS = credentials('nexus-pwd')
	        registryCredential = 'ecr:us-west-1:aws-creds'
        appRegistry = '592255970248.dkr.ecr.us-west-1.amazonaws.com/vprofile-man'
        vprofileRegistry = "https://592255970248.dkr.ecr.us-west-1.amazonaws.com"
	    cluster = "vprofile-cluster-stage"
	    service = "vproappstagesvc"
        ARTIFACT_NAME = "vprofile-v${buildNumber}.war"
        AWS_S3_BUCKET = 'beanstalk-vpro'
        AWS_EB_APP_NAME = 'new-application'
        AWS_EB_ENVIRONMENT = 'New-application-stage'
        AWS_EB_APP_VERSION = "${buildNumber}"
    }

    stages {

        stage('Deploy to Prod Bean'){
          steps {
            withAWS(credentials: 'aws-bean-creds', region: 'us-west-1') {
               sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
          }
        }
    }
}
