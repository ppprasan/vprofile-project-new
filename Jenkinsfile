pipeline {
    agent any
    
    environment {
	    NEXUSPASS = credentials('nexus-pwd')
    }

    stages {

        stage('Setup parameters') {
            steps {
                script { 
                    properties([
                        parameters([
                            string(
                                defaultValue: '', 
                                name: 'BUILD', 
                            ),
							string(
                                defaultValue: '', 
                                name: 'TIME', 
                            )
                        ])
                    ])
                }
            }
		}

        stage('Ansible deploy to prod') {
            steps {
                    ansiblePlaybook([
                    inventory   : 'ansible/prod.inventory',
                    playbook    : 'ansible/site.yml',
                    installation: 'ansible',
                    colorized   : true,
                    credentialsId: 'prod-app-server-creds',
                    disableHostKeyChecking: true,
                    extraVars   : [
                        USER: "admin",
                        PASS: "${NEXUSPASS}",
                        nexusip: "172.31.10.247",
                        reponame: "vprofile-release",
                        groupid: "QA",
                        time: "${env.TIME}",
                        build: "${env.BUILD}",
                        artifactid: "vproapp",
                        vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
                    ]
                ])
                }
        }
    }
}