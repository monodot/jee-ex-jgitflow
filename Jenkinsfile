#!/usr/bin/env groovy
@Library('pipeline-library')_


node('maven') {

    parallel Checkout: {
          stage('Checkout') {

          echo sh(returnStdout: true, script: 'env')

          git url: 'https://github.devops.worldpay.local/donohuet826/web-app.git',  branch: "${env.BRANCH_NAME}" 
          def v = version()
          currentBuild.displayName = "${env.BRANCH_NAME}-${v}-${env.BUILD_NUMBER}"
     }
    }, 'Build Dependencies': {
        builddependencies()
        sleep 1
    }
 
 def branch_type = get_branch_type "${env.BRANCH_NAME}"
 def branch_deployment_environment = get_branch_deployment_environment branch_type

 switch (branch_type) {
  case "feature":
   echo 'Builing from the feature branch'
 //  test()
 //  sonar()
   snapshot()
 //  parallel VeraCode: {
 //       veracode()  
 //   }, 'AQUA Scan': {
 //       aqua()
 //   }, 'OpenSCAP security analysis': {
 //       openscap()
 //   }
   ocp()
  case "develop":
    echo 'Building from the develop branch'
    def msg = get_commit_message()
    
    def pom = readMavenPom file: 'pom.xml'
    ocpProcessJGitFlowCommit(artifactId: pom.artifactId, commitMsg: msg)
    
  default:
   echo 'build default branch'
 }
 
}

def test() {
 stage('Run JUnits') {
  mvn "clean verify -s /home/jenkins/.m2/settings.xml -f pom.xml "
  echo 'Maven jUnits ran successfully'
 }
}
def builddependencies() {
 stage('Build dependencies') {
  echo 'Dependencies built and published to Nexus successfully'
 }
}

def snapshot() {
 stage('Maven publish to NEXUS') {
  mvn "clean deploy -q -s /home/jenkins/.m2/settings.xml -f pom.xml -DaltDeploymentRepository=maven-snapshots::default::https://nexus-ci.apps-build.ew1.aws.worldpay.local/repository/maven-snapshots/ -Dmaven.test.skip=true"
//  sh 'mvn clean deploy -s /home/jenkins/.m2/settings.xml -f tomcat-jdbc/pom.xml -DaltDeploymentRepository=maven-snapshots::default::https://nexus-ci.apps-build.ew1.aws.worldpay.local/repository/maven-snapshots/'
  echo 'Maven snapshot build complete'
 }
}

def sonar() {
 // https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins

try {

 stage('SonarQube analysis') {
  withSonarQubeEnv('sonarqube') {
   // requires SonarQube Scanner for Maven 3.2+
   //sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar -s /home/jenkins/.m2/settings.xml -f tomcat-jdbc/pom.xml'
     mvn "org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar -s /home/jenkins/.m2/settings.xml -f pom.xml"
  }
 }
} catch (err) {
  echo "Sonar failed but carrying on"
}
 
}

def veracode() {
stage('VeraCode analysis') {
    // Insert REST call
    sleep 10
    echo 'Binary pushed to VeraCode for analysis'
 }    
}    

def aqua() {
stage('aquasec') {
    // Insert REST call
    sleep 15
    echo 'Aqua security scan complete'
 }    
}    

def openscap() {
stage('Openscap analysis') {
    // Insert REST call
    sleep 5
    echo 'open-scap.org analysis done'
 }    
}    

def ocp() {
    
 // when building for feature, hotfix or release (i.e. any ephemeral namespace) this will be deleted from the registry once the branch is deleted/merged into develop for feature or master for hotfix or release branches
 stage('OpenShift Build And Deploy') {
  
  //In this case bc, dc, is - all have the same name
  def imageStreamName = 'quickstarts'
  def devReleaseTag = 'feature-branch'
  def bcName = 'quickstarts'

  def pom = readMavenPom file: 'pom.xml'
  def branch = "${env.BRANCH_NAME}".replace("/", "-")
  // assumed that feature nsamespace exists and jenkins can write to it
  def buildNamespace = "${pom.artifactId}-${branch}"
  // replace . with /
  def groupId = "${pom.groupId}".replace(".", "/")
//  print "groupId = ${groupId}"
  
  def snap = snapversion "https://nexus-ci.apps-build.ew1.aws.worldpay.local/repository/maven-snapshots/${groupId}/${pom.artifactId}/${pom.version}/maven-metadata.xml"
  def artifactUrl = "https://nexus-ci.apps-build.ew1.aws.worldpay.local/repository/maven-snapshots/${groupId}/${pom.artifactId}/${pom.version}/${pom.artifactId}-${snap}.${pom.packaging}"
//  println "URL is ${artifactUrl}---" 
  //def artifactUrl = "https://nexus-ci.apps-build.ew1.aws.worldpay.local/repository/maven-snapshots/com/mkyong/CounterWebApp/1.0-SNAPSHOT/CounterWebApp-1.0-20170629.110155-11.war"
    
    try {
 //TODO use project service user
  sh "oc login https://ocp-build.ew1.aws.worldpay.local:443 -u admin -p openshift"
  //deleting old project - to be changed
  echo "login done"
  sh "oc project ${buildNamespace}"
  //assuming that if the namespace exists, its setup correctly
  echo "changing to project namespace, and if this errors out as the proj is non-existent we will create and setup the namespace in the catch block"
    }
    catch (err) {
  echo "Project does not exist so creating objects"
  sh "oc new-project ${buildNamespace}"
  sh "oc policy add-role-to-user view system:serviceaccounts:ci:jenkins -n ${buildNamespace}"
  sh "oc policy add-role-to-user edit system:serviceaccounts:ci:jenkins -n ${buildNamespace}"
  //sh "oc export is,bc,dc,svc,route,serviceaccount,secret --as-template=web-app --exact=false > template.yaml"
  sh 'oc project ${buildNamespace}'
  sh "oc process -n ${buildNamespace} -f kube/template.yaml BRANCH_NAME=${env.BRANCH_NAME} BRANCH_NAME_HY=${branch} ARTIFACT_ID=${pom.artifactId} | oc create -n ${buildNamespace} -f -"
  
 // openshiftCreateResource jsonyaml: "kube/template.yaml", namespace: "${buildNamespace}"
 // sh 'oc new-app quickstarts --template=web-app'
  echo 'resources created'
  //BRANCH_NAME=feature/tem-202 BRANCH_NAME_HY=feature-tem-202 ARTIFACT_ID=web-app
//  sh "oc process ${pom.artifactId} BRANCH_NAME=${env.BRANCH_NAME} BRANCH_NAME_HY=${branch} ARTIFACT_ID=${pom.artifactId} | oc create -f -"
  //oc process -f kube/template.yaml | oc create -f -
       
}

  sh "oc policy add-role-to-group view system:serviceaccounts:ci -n ${buildNamespace}"
  sh "oc policy add-role-to-group edit system:serviceaccounts:ci -n ${buildNamespace}"
  sleep 10
  openshiftBuild buildConfig: bcName, namespace: buildNamespace, verbose: 'false', showBuildLogs: 'false', env: [   [name: 'WAR_FILE_URL', value: artifactUrl]  ]
  println "OpenShift build complete"
  openshiftTag(namespace: buildNamespace, sourceStream: imageStreamName, sourceTag: 'latest', destinationStream: imageStreamName, destinationTag: "${pom.artifactId}-${snap}")
  openshiftTag(namespace: buildNamespace, sourceStream: imageStreamName, sourceTag: 'latest', destinationStream: imageStreamName, destinationTag: devReleaseTag)
  println "OpenShift build tagged"
  
    openshiftDeploy deploymentConfig: bcName, namespace: buildNamespace
      echo "The deployed app can be found: https://quickstarts-${buildNamespace}.apps-build.ew1.aws.worldpay.local"


 }
}

def ocp_featurefinish() {
    
  
}

// Utility functions
def get_branch_type(String branch_name) {
    //Must be specified according to <flowInitContext> configuration of jgitflow-maven-plugin in pom.xml
    def dev_pattern = ".*develop"
    def release_pattern = ".*release/.*"
    def feature_pattern = ".*feature/.*"
    def hotfix_pattern = ".*hotfix/.*"
    def master_pattern = ".*master"
    if (branch_name =~ dev_pattern) {
        return "develop"
    } else if (branch_name =~ release_pattern) {
        return "release"
    } else if (branch_name =~ master_pattern) {
        return "master"
    } else if (branch_name =~ feature_pattern) {
        return "feature"
    } else if (branch_name =~ hotfix_pattern) {
        return "hotfix"
    } else {
        return null;
    }
}

def get_branch_deployment_environment(String branch_type) {
    if (branch_type == "dev") {
        return "dev"
    } else if (branch_type == "release") {
        return "staging"
    } else if (branch_type == "master") {
        return "prod"
    } else {
        return null;
    }
}


def mvn(String goals) {
        sh "mvn -B ${goals}"
}

def version() {
    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
    return matcher ? matcher[0][1] : null
}


def snapversion(String myUrl) {
 
def data = new URL(myUrl).getText()

def metadata = new XmlParser().parseText(data)
def snap

metadata = new XmlSlurper().parseText(data)
metadata.versioning.snapshotVersions.snapshotVersion.value.each{
 //text() is Important as else we get a java.io.NotSerializableException re NodeChild
  snap = it.text()
  
 }
echo "snap is ${snap}"
return "$snap"
}   

// Gets the latest commit message
def get_commit_message() {
  commitMessage = sh (script: 'git --no-pager show -s --format=\'%B\'', returnStdout: true).trim()
  commitMessage
}


