#!/usr/bin/groovy
@Library('gogs/gogsadmin/fabric8-pipeline-library@offline')
def utils = new io.fabric8.Utils()
clientsNode{
  def envStage = utils.environmentNamespace('staging')
  def envProd = utils.environmentNamespace('production')
  def newVersion = ''

  git 'https://github.com/haikuo81/devops-go-test.git'

  stage 'Canary release'
  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  if (!fileExists ('Dockerfile')) {
    writeFile file: 'Dockerfile', text: 'FROM 10.253.0.11/library/golang:onbuild'
  }

  newVersion = performCanaryRelease {}
  
  def rc = getKubernetesJson {
    port = 8080
    label = 'golang'
    icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/gopher.png'
    version = newVersion
    imageName = clusterImageName
  }

  stage 'Rollout Staging'
  kubernetesApply(file: rc, environment: envStage)

  stage 'Approve'
  approve{
    room = null
    version = canaryVersion
    console = fabric8Console
    environment = envStage
  }

  stage 'Rollout Production'
  kubernetesApply(file: rc, environment: envProd)

}
