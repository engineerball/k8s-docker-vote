#!/usr/bin/env groovy

podTemplate(label: 'jnlp-slave', containers: [
  containerTemplate(name: 'jnlp-slave', image: 'engineerball/jenkins-slave-dind-kubernetes-int', ttyEnabled: true,
  privileged: true)
]) {
  node('jnlp-slave'){
    def project = 'engineerball'
    def appName = 'example-voting'
    def votingSvcName = "${appName}-voting"
    def votingImageTag = "${project}/${appName}-voting:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    def resultImageTag = "${project}/${appName}-result:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    def workerImageTag = "${project}/${appName}-worker:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    def kubeServer = 'http://172.17.181.188:8080'

    checkout scm

    stage 'Build Image'
    parallel (
      "voting-app": {sh "cd voting-app; docker build -t ${votingImageTag} . "},
      "result-app": {sh "cd result-app; docker build -t ${resultImageTag} . "},
      "worker": {sh "cd worker; docker build -t ${workerImageTag} . "}
    )

    stage 'Push image to registry'
    docker.withRegistry('https://index.docker.io/v1/','007ab01d-9ee0-48dc-b68b-d04ab2795299'){
      parallel (
        "voting-app": {sh "cp /root/.dockercfg ~/.dockercfg; docker push ${votingImageTag}"},
        "result-app": {sh "cp /root/.dockercfg ~/.dockercfg; docker push ${resultImageTag}"},
        "worker": {sh "cp /root/.dockercfg ~/.dockercfg; docker push ${workerImageTag}"},
      )
    }

    stage 'Deploy application'
    sh("sed -i.bak 's#engineerball/example-voting-voting:null.14#${votingImageTag}#' ./k8s/production/*.yaml")
    sh("sed -i.bak 's#engineerball/example-voting-result:null.14#${resultImageTag}#' ./k8s/production/*.yaml")
    sh("sed -i.bak 's#engineerball/example-voting-worker:null.14#${workerImageTag}#' ./k8s/production/*.yaml")
    sh("kubectl --server ${kubeServer} --namespace=default apply -f k8s/services/")
    sh("kubectl --server ${kubeServer} --namespace=default apply -f k8s/production/")
  }

}
