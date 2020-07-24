node {
    def mvnHome = tool name: 'Maven_3', type: 'maven'
    def mvnCli = "${mvnHome}/bin/mvn"

    properties([
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')),
        disableConcurrentBuilds(),
        [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: 'https://github.com/prakashmankena/PetClinic.git/'],
        [$class: 'ThrottleJobProperty', categories: [], limitOneJobWithMatchingParams: false, maxConcurrentPerNode: 0, maxConcurrentTotal: 0, paramsToUseForLimit: '', throttleEnabled: true, throttleOption: 'project'],
        pipelineTriggers([githubPush()]),
        parameters([string(defaultValue: 'DEV', description: 'env name', name: 'environment', trim: false)])
    ])
    stage('Checkout SCM'){
        git branch: 'master', credentialsId: 'github-creds', url: 'https://github.com/prakashmankena/petclinic/'
    }
    stage('Read praram'){
        echo "The environment chosen during the Job execution is ${params.environment}"
        echo "$JENKINS_URL"
    }
    stage('maven compile'){
        // def mvnHome = tool name: 'Maven_3.6', type: 'maven'
        // def mvnCli = "${mvnHome}/bin/mvn"
        sh "${mvnCli} clean compile"
    }
    stage('maven package'){
        sh "${mvnCli} package -Dmaven.test.skip=true"
    }
    stage('Archive atifacts'){
        archiveArtifacts artifacts: '**/*.war', onlyIfSuccessful: true
    }
    stage('Archive Test Results'){
        junit allowEmptyResults: true, testResults: '**/surefire-reports/*.xml'
    }
    stage('Deploy To Tomcat'){
        sshagent(['app-server']) {
            sh 'scp -o StrictHostKeyChecking=no target/*.war root@172.31.47.199:/opt/apache-tomcat-9.0.37/webapps'
            
        }
    }
    stage('Smoke Test'){
        sleep 5
        sh "curl 172.31.47.199:8080/petclinic"
    }

}
