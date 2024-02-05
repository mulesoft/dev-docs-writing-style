#!/bin/env groovy

def defaultBranch = 'master'
def everythingInsideMulesoftDir = 'MuleSoft/**'
def githubCredentialsId = 'GH_TOKEN'

def nodeVersion = '20'

// update this to the latest version showing in https://github.com/errata-ai/vale/releases
def valeVersion = '3.0.7'

pipeline {
  agent any
  stages {
    stage('Set Up') {
      steps {
        installNode(nodeVersion)
        installNodeDependencies()
        installVale(valeVersion)
      }
    }
    stage('Test') {
      when {
        allOf {
          not { branch defaultBranch }
        }
      }
      steps {
        sh 'which asciidoctor'
        sh './vale .'
        sh 'make test'
      }
    }
    stage('Release') {
      when {
        allOf {
          branch defaultBranch
          changeset everythingInsideMulesoftDir
        }
      }
      steps {
        withCredentials([
          string(credentialsId: githubCredentialsId, variable: 'GH_TOKEN')]) {
            sh 'export GIT_BRANCH=env.GIT_BRANCH'
            sh 'npm run release'
          }
      }
    }
  }
}

void installNode(String nodeVersion) {
  withCredentials([string(credentialsId: 'NPM_TOKEN', variable: 'NPM_TOKEN')]) {
    sh "curl -fsSL https://deb.nodesource.com/setup_${nodeVersion}.x | sudo -E bash - && sudo apt-get install -y nodejs"
  }
}

void installNodeDependencies() {
  sh 'npm ci --cache=.cache/npm --no-audit'
}

void installVale(String valeVersion) {
  // see https://vale.sh/docs/vale-cli/installation/#github-releases
  sh "wget https://github.com/errata-ai/vale/releases/download/v${valeVersion}/vale_${valeVersion}_Linux_64-bit.tar.gz"
  sh "tar -xvzf vale_${valeVersion}_Linux_64-bit.tar.gz -C ./"
}
