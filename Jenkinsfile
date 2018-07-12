#!groovy
properties([
    pipelineTriggers([
        [$class: "SCMTrigger", scmpoll_spec: "H * * * *"],
    ])
])

def appName = 'mongodb'
def namespace = 'mongodb'
def imageTag = "${appName}:v${env.BUILD_NUMBER}"
def prevImageTag = ''
def prevBuildNum = ''
def firstDeploy = false
def deployResult = ''
def helmStatus = ''
def helmInstallStatus = ''
def createNamespace = ''
def rbacStatus = ''

node {
  stage ('Poll') {
      git(
      poll: true,
          url: 'https://github.com/gorzek/chart-packages',
          branch: 'master'
      )
    }
  stage ('Get Helm') {
      helmStatus = sh(
          script: "curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash",
          returnStdout: true)
      echo "${helmStatus}"
  }
  stage ('Check Deployment') {
      try {
        prevImageTag = sh(
        script: "kubectl get deployment mongodb -n ${namespace} -o jsonpath='{.spec.template.spec.containers[0].image}'",
        returnStdout: true
        ).trim()
        echo "Previous Image: ${prevImageTag}"
        prevBuildNum = prevImageTag.split(':')[1]
        echo "Previous Build Version: ${prevBuildNum}"
      } catch (err) {
        echo "No Previous Deployment"
        firstDeploy = true
      }
      try {
        createNamespace = sh(
        script: "kubectl create namespace ${namespace}",
        returnStdout: true
        )
        echo "${createNamespace}"
      } catch (err) {
        echo "Namespace already present"
      }
    }
  stage ('Deploy') {
    echo 'Deploy section'
    if (firstDeploy == true) {
        deployResult = sh(
            script: "helm install mongodb.tgz --wait --namespace ${namespace} --name ${appName}",
            returnStdout: true)
        echo "First deployment"
        echo "${deployResult}"
    }
    else {
        deployResult = sh(
            script: "helm upgrade ${appName} mongodb.tgz --wait --namespace ${namespace}",
            returnStdout: true)
        echo "Subsequent deployment"
        echo "${deployResult}"
    }
    sh("kubectl --namespace=${namespace} label deployment mongodb --overwrite version=${appName}-deploy-${env.BUILD_NUMBER}")
  }
}
