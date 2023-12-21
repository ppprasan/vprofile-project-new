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
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = 'beanstalk-vpro'
        AWS_EB_APP_NAME = 'new-application'
        AWS_EB_ENVIRONMENT = 'New-application-stage'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

	stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

	stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

	stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }  

	    // stage('Build App Image') {
        //     steps {
        //         script {
        //             dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
        //         }
        //     }
        // }
        
        // stage('Upload App Image') {
        //   steps{
        //     script {
        //       docker.withRegistry( vprofileRegistry, registryCredential ) {
        //         dockerImage.push("$BUILD_NUMBER")
        //         dockerImage.push('latest')
        //       }
        //     }
        //   }
        // }

        // stage('Deploy to ECS staging') {
        //            steps {
        //                withAWS(credentials: 'aws-creds', region: 'us-west-1') {
        //                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        //                } 
        //            }
        //        }

        // stage('Ansible deploy to staging') {
        // 	steps {
        //             ansiblePlaybook([
        //             inventory   : 'ansible/stage.inventory',
        //             playbook    : 'ansible/site.yml',
        //             installation: 'ansible',
        //             colorized   : true,
        // 		    credentialsId: 'app-server-creds',
        // 		    disableHostKeyChecking: true,
        //             extraVars   : [
        //                	USER: "admin",
        //                 PASS: "${NEXUSPASS}",
        // 		        nexusip: "172.31.10.247",
        // 		        reponame: "vprofile-release",
        // 		        groupid: "QA",
        // 		        time: "${env.BUILD_TIMESTAMP}",
        // 		        build: "${env.BUILD_ID}",
        //                 artifactid: "vproapp",
        // 		        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
        //             ]
        //          ])
        //         }
        // }

        stage('Deploy to Stage Bean'){
          steps {
            withAWS(credentials: 'aws-bean-creds', region: 'us-west-1') {
               sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
               sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
               sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
          }
        }
    }
}
