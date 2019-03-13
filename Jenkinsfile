pipeline {
    agent { label 'aws' }
    stages {
        stage('Code Quality') {
            environment {
                sonarscanner = tool name: 'ADOP SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            }
            steps {
                cleanWs()
                checkout changelog: false, 
                    poll: false, 
                    scm: [$class: 'GitSCM', 
                    branches: [[name: '*/develop']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', 
                    relativeTargetDir: 'commerce-cloud']], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'adop-jenkins-master', 
                    url: 'git@bitbucket.org:essiloronline/commerce-cloud.git']]]

                checkout changelog: false, 
                    poll: false, 
                    scm: [$class: 'GitSCM', 
                    branches: [[name: '*/master']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', 
                    relativeTargetDir: "${WORKSPACE}/commerce-cloud/commerce-cloud-gd"]], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'adop-jenkins-master', 
                    url: 'git@bitbucket.org:essiloronline/commerce-cloud-gd.git']]]

                checkout changelog: false, 
                    poll: false, 
                    scm: [$class: 'GitSCM', 
                    branches: [[name: '*/master']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', 
                    relativeTargetDir: "${WORKSPACE}/commerce-cloud/commerce-cloud-vd"]], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'adop-jenkins-master', 
                    url: 'git@bitbucket.org:essiloronline/commerce-cloud-vd.git']]]

                checkout changelog: false, 
                    poll: false, 
                    scm: [$class: 'GitSCM', 
                    branches: [[name: '*/master']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [[$class: 'RelativeTargetDirectory', 
                    relativeTargetDir: "${WORKSPACE}/commerce-cloud/storefront-reference-architecture"]], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: 'adop-jenkins-master', 
                    url: 'git@github.com:SalesforceCommerceCloud/storefront-reference-architecture.git']]]

                sh """
                #!/bin/bash
                rm -rf ${WORKSPACE}/*@tmp
                rm -rf ${WORKSPACE}/commerce-cloud/*@tmp
                """
                withSonarQubeEnv ('ADOP Sonar') {
                    sh '''
                    #!/bin/bash
                    ${sonarscanner}/bin/sonar-scanner \
                        -Dsonar.projectKey=essilor \
                        -Dsonar.projectName=essilor-devops-${BUILD_ID} \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.sources=.
                '''
                }
                cleanWs()
            }
        }
        stage('Build') {
            steps {
                sshagent(['adop-jenkins-master']) {
                    checkout changelog: false,
                        poll: false,
                        scm: [$class: 'GitSCM',
                        branches: [[name: '*/master']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        submoduleCfg: [],
                        userRemoteConfigs: [[credentialsId: 'adop-jenkins-master',
                        url: 'git@bitbucket.org:jgbalagtas/essilor-build-suite.git']]]

                    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'css']) {
                        withCredentials([usernamePassword(credentialsId: 'aws-credentials-id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {    
                            sh '''#!/bin/bash
                            ## => RETRIEVE SECRETS FROM AWS <= ##
                            export AWS_DEFAULT_REGION=us-east-1
                            export USERNAME=`aws secretsmanager get-secret-value --secret-id SFCCSandbox/Dev01_login --version-stage AWSCURRENT --output text --query 'SecretString' | awk -F '"' '{print $4}'`
                            export PASSWORD=`aws secretsmanager get-secret-value --secret-id SFCCSandbox/Dev01_login --version-stage AWSCURRENT --output text --query 'SecretString' | awk -F '"' '{print $8}'`

                            ## => SET BUILD VARIABLES <= ##
                            sed -i "s|REVISION|${BUILD_ID}|" ${WORKSPACE}/build/commerce-cloud.json
                            sed -i "s|USERNAME|${USERNAME}|" ${WORKSPACE}/build/commerce-cloud.json
                            sed -i "s|PASSWORD|${PASSWORD}|" ${WORKSPACE}/build/commerce-cloud.json
                            
                            ## => Perform Build <= ##
                            npm install
                            grunt build --project=commerce-cloud
                            '''
                        }
                    }
                }
            }
        }    
        stage('Deploy') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'css']) {
                    sh '''#!/bin/bash
                    ## => UPLOAD CODE <= ##
                    grunt upload --project=commerce-cloud

                    ## => ACTIVATE CODE <= ##
                    grunt activate --project=commerce-cloud
                    '''                
                }
            }
        }
    }
}    

