#!groovy

import com.michelin.cio.hudson.plugins.rolestrategy.*

library(
  identifier: 'JenkinsSharedLibrary@master',
  retriever: modernSCM(
    [
      $class: 'GitSCMSource',
      remote: 'https://github.com/abcert/JenkinsSharedLibrary.git'
    ]
  )
)

pipeline 
{
    agent any
    /*
    agent {
        docker {
            image 'jenkinsslave:latest'
            registryUrl 'http://8598567586.dkr.ecr.us-west-2.amazonaws.com'
            registryCredentialsId 'ecr:us-east-1:3435443545-5546566-567765-3225'
            args '-v /home/centos/.ivy2:/home/jenkins/.ivy2:rw -v jenkins_opt:/usr/local/bin/opt -v jenkins_apijenkins:/home/jenkins/config -v jenkins_logs:/var/logs -v jenkins_awsconfig:/home/jenkins/.aws --privileged=true -u jenkins:jenkins'
        }
    }
    */
    environment {
        APP_NAME = 'LearnJenkins'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        IMAGE_VERSION="v_${BUILD_NUMBER}"
        GIT_URL="https://github.com/abcert/${APP_NAME}.git"
        GIT_CRED_ID='izleka2IGSTDK+MiYOG3b3lZU9nYxhiJOrxhlaJ1gAA='
        //REPOURL = 'cL5nSDa+49M.dkr.ecr.us-east-1.amazonaws.com'
        SBT_OPTS='-Xmx1024m -Xms512m'
        JAVA_OPTS='-Xmx1024m -Xms512m'
        //WS_PRODUCT_TOKEN='FJbep9fKLeJa/Cwh7IJbL0lPfdYg7q4zxvALAxWPLnc='
        //WS_PROJECT_TOKEN='zwzxtyeBntxX4ixHD1iE2dOr4DVFHPp7D0Czn84DEF4='
        //HIPCHAT_TOKEN = 'SpVaURsSTcWaHKulZ6L4L+sjKxhGXCkjSbcqzL42ziU='
        //HIPCHAT_ROOM = 'NotificationRoomName'
    }

/*
    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5'))
        timestamps()
        retry(3)
        timeout time:10, unit:'MINUTES'
    }

    */

    parameters {
        string(defaultValue: "master", description: 'Branch Specifier', name: 'SPECIFIER')
        booleanParam(defaultValue: false, description: 'Deploy to QA Environment ?', name: 'DEPLOY_QA')
        booleanParam(defaultValue: false, description: 'Deploy to UAT Environment ?', name: 'DEPLOY_UAT')
        booleanParam(defaultValue: false, description: 'Deploy to PROD Environment ?', name: 'DEPLOY_PROD')
    }

    stages {
        stage("Initialize") {
            steps {
                script {
                    notifyBuild('STARTED')
                    echo "${BUILD_NUMBER} - ${env.BUILD_ID} on ${env.JENKINS_URL}"
                    echo "Branch Specifier :: ${params.SPECIFIER}"
                    echo "Deploy to QA? :: ${params.DEPLOY_QA}"
                    echo "Deploy to UAT? :: ${params.DEPLOY_UAT}"
                    echo "Deploy to PROD? :: ${params.DEPLOY_PROD}"
                    //sh 'rm -rf target/universal/*.zip'
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: "${params.SPECIFIER}", url: "${GIT_URL}"
            }
        }
        stage('Build') {
            parallel {
                stage ('Build Java/Scala Projects using Maven'){
                    steps {
                        echo 'Build Java/Scala Project using maven'
                        //sh '/usr/local/bin/opt/bin/sbtGitActivator; /usr/local/bin/opt/play-2.5.10/bin/activator -Dsbt.global.base=.sbt -Dsbt.ivy.home=/home/jenkins/.ivy2 -Divy.home=/home/jenkins/.ivy2 compile coverage test coverageReport coverageOff dist'
                    }
                }
                stage ('Build Scala Projects using SBT'){
                    steps {
                        echo 'Build Scala Project using SBT'
                        //sh '/usr/local/bin/opt/bin/sbtGitActivator; /usr/local/bin/opt/play-2.5.10/bin/activator -Dsbt.global.base=.sbt -Dsbt.ivy.home=/home/jenkins/.ivy2 -Divy.home=/home/jenkins/.ivy2 compile coverage test coverageReport coverageOff dist'
                    }
                }
            }
        }
        stage('Publish Reports') {
            parallel {
                stage('Publish FindBugs Report') {
                    steps {
                        echo "FindBugsPublisher"
                        //step([$class: 'FindBugsPublisher', canComputeNew: false, defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', pattern: 'target/scala-2.11/findbugs/report.xml', unHealthy: ''])
                    }
                }
                stage('Publish Junit Report') {
                    steps {
                        echo "junit allowEmptyResults: true, testResults: 'target/test-reports/*.xml'"
                        //junit allowEmptyResults: true, testResults: 'target/test-reports/*.xml'
                    }
                }
                stage('Publish Junit HTML Report') {
                    steps {
                        publishHTML target: [
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'target/reports/html',
                                reportFiles: 'index.html',
                                reportName: 'Test Suite HTML Report'
                            ]
                    }
                }
                stage('Publish Coverage HTML Report') {
                    steps {
                        publishHTML target: [
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'target/scala-2.11/scoverage-report',
                                reportFiles: 'index.html',
                                reportName: 'Code Coverage'
                            ]
                    }
                }
                stage('Execute Whitesource Analysis') {
                    steps {
                        echo 'execute whitesource analysis'
                        //whitesource jobApiToken: '', jobCheckPolicies: 'global', jobForceUpdate: 'global', libExcludes: '', libIncludes: '', product: "${env.WS_PRODUCT_TOKEN}", productVersion: '', projectToken: "${env.WS_PROJECT_TOKEN}", requesterEmail: ''
                    }
                }    
                stage('SonarQube analysis') {
                    steps {
                        echo 'sh "/usr/bin/sonar-scanner"'
                        //sh "/usr/bin/sonar-scanner"
                    }
                }
                stage('ArchiveArtifact') {
                    steps {
                        echo "ArchiveArtifact"
                        //archiveArtifacts '**/target/universal/*.zip'
                    }
                }
            }    
        }

        stage('Docker Tag & Push') {
            steps {
                script {
                    branchName = getCurrentBranch()
                    shortCommitHash = getShortCommitHash()
                    echo "BranchName-> [" + "${branchName} - " + "] CommitHash->[" + "${shortCommitHash}" + "]"
                    /*
                    IMAGE_VERSION = "${BUILD_NUMBER}-" + branchName + "-" + shortCommitHash
                    sh 'eval $(aws ecr get-login --no-include-email --region us-west-2)'
                    sh "docker-compose build"
                    sh "docker tag ${REPOURL}/${APP_NAME}:latest ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
                    sh "docker push ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
                    sh "docker push ${REPOURL}/${APP_NAME}:latest"

                    sh "docker rmi ${REPOURL}/${APP_NAME}:${IMAGE_VERSION} ${REPOURL}/${APP_NAME}:latest"
                    */
                }
            }
        }
        
        stage('Deploy') {
            parallel {
                stage('Deploy to CI') {
                    steps {
                        echo "Deploying to Dev Environment."
                    }
                }

                stage('Deploy to QA') {
                    when {
                        expression {
                            params.DEPLOY_QA == true
                        }
                    }
                    steps {
                        echo "Deploy to QA..."
                        script{
                            //def role = "Testers"
                            echo "Retrieving users for ${role}..."
                            def users = [:]
                            
                            def currentUser = jenkins.model.Jenkins.instance.getAuthentication().getName();
                            def roleMap= authStrategy.roleMaps.get("globalRoles")

                            def sids= roleMap.getSidsForRole("Testers")
                            if(sids != null && sids.contains(currentUser)) {
                                //result.add("dev1")
                                echo "found testers for approval"
                            }


                            /*
                            def authStrategy = Jenkins.instance.getAuthorizationStrategy()
                            if(authStrategy instanceof RoleBasedAuthorizationStrategy){
                                def sids = authStrategy.roleMaps.globalRoles.getSidsForRole(role)
                                sids.each { sid ->
                                    users[sid] = Jenkins.instance.getUser(sid).fullName
                                }
                            }
                            */
                            println(users)
                        }
                        
                        
                    }
                }
                stage('Deploy to UAT') {
                    when {
                        expression {
                            params.DEPLOY_UAT == true
                        }
                    }
                    steps {
                        echo "Deploy to UAT..."
                    }
                }
                stage('Deploy to Production') {
                    when {
                        expression {
                            params.DEPLOY_PROD == true
                        }
                    }
                    steps {
                        echo "Deploy to PROD..."
                    }
                }
            }
        }
    }

    post {
            /*
            * These steps will run at the end of the pipeline based on the condition.
            * Post conditions run in order regardless of their place in the pipeline
            * 1. always - always run
            * 2. changed - run if something changed from the last run
            * 3. aborted, success, unstable or failure - depending on the status
            */
            always {
                echo "I AM ALWAYS first"
                notifyBuild("${currentBuild.currentResult}")
            }
            aborted {
                echo "BUILD ABORTED"
            }
            success {
                echo "BUILD SUCCESS"
                echo "Keep Current Build If branch is master"
    //            keepThisBuild()
            }
            unstable {
                echo "BUILD UNSTABLE"
            }
            failure {
                echo "BUILD FAILURE"
            }
    }
}

def keepThisBuild() {
    currentBuild.setKeepLog(true)
    currentBuild.setDescription("Test Description")
}

def getShortCommitHash() {
    return sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
}

def getChangeAuthorName() {
    return sh(returnStdout: true, script: "git show -s --pretty=%an").trim()
}

def getChangeAuthorEmail() {
    return sh(returnStdout: true, script: "git show -s --pretty=%ae").trim()
}

def getChangeSet() {
    return sh(returnStdout: true, script: 'git diff-tree --no-commit-id --name-status -r HEAD').trim()
}

def getChangeLog() {
    return sh(returnStdout: true, script: "git log --date=short --pretty=format:'%ad %aN <%ae> %n%n%x09* %s%d%n%b'").trim()
}

def getCurrentBranch () {
    return sh (
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
    ).trim()
}

def isPRMergeBuild() {
    return (env.BRANCH_NAME ==~ /^PR-\d+$/)
}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    def branchName = getCurrentBranch()
    def shortCommitHash = getShortCommitHash()
    def changeAuthorName = getChangeAuthorName()
    def changeAuthorEmail = getChangeAuthorEmail()
    def changeSet = getChangeSet()
    def changeLog = getChangeLog()

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'" + branchName + ", " + shortCommitHash
    def summary = "Started: Name:: ${env.JOB_NAME} \n " +
            "Build Number: ${env.BUILD_NUMBER} \n " +
            "Build URL: ${env.BUILD_URL} \n " +
            "Short Commit Hash: " + shortCommitHash + " \n " +
            "Branch Name: " + branchName + " \n " +
            "Change Author: " + changeAuthorName + " \n " +
            "Change Author Email: " + changeAuthorEmail + " \n " +
            "Change Set: " + changeSet

    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    // Send notifications
    //hipchatSend(color: color, notify: true, message: summary, token: "${env.HIPCHAT_TOKEN}",
    //    failOnError: true, room: "${env.HIPCHAT_ROOM}", sendAs: 'Jenkins', textFormat: true)
    if (buildStatus == 'FAILURE') {
        echo "Email Sent"
        //emailext attachLog: true, body: summary, compressLog: true, recipientProviders: [brokenTestsSuspects(), brokenBuildSuspects(), culprits()], replyTo: 'noreply@yourdomain.com', subject: subject, to: 'mpatel@yourdomain.com'
    }
}
