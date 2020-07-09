import groovy.json.JsonSlurper
import groovy.json.JsonOutput

//properties([parameters([string(defaultValue: '1.0.0', description: '', name: 'version', trim: false), string(defaultValue: '1.1.0', description: '', name: 'nextversion', trim: false)])])

pipeline {

  agent any

  options {
    timestamps()
    withFolderProperties()
  }
  environment {
    TIMESTAMP = new java.text.SimpleDateFormat('yyyyMMdd').format(new Date())
    image = "dummy_image"
    version = "2.1.0"
    //version = "0.1.${env.BUILD_NUMBER}"
    //version = "${params.version}"
    //nextversion = "${params.nextversion}"
    branch_name="${env.GIT_BRANCH}".replace("origin/","")
    tag = "${branch_name =="master" ? "${version}" : "${version}-${branch_name}"}"
    imageTag = "/${image}:${tag}"
  }

  stages {   
    stage('Cleanup') {
      steps {
        //echo "This is ${env.GIT_BRANCH}"
        echo "This is ${branch_name}"
        //echo "${env.BRANCH_OVERRIDE}"
        echo "image tag is: ${imageTag}"
        //echo "image tag is: ${env.imageTag}"
        //echo "tag is:  ${env.tag}"
        echo "tag is : $tag"
        echo "new version is: ${version}"
        //echo "next version is : ${nextversion}"
        echo "branch name is : ${branch_name}"
        // sh 'printenv'
      }
    }

    stage('Checkout') {
      steps {
        //checkout scm
        script {
          sh 'rm -Rf test-sourcecode/*'
          repo = checkout([
            $class: 'GitSCM',
            branches: [
              [name: '*/master']
            ],
            doGenerateSubmoduleConfigurations: false,
            userRemoteConfigs: [[
              credentialsId: 'github-ssh-username-private-key',
              url: "git@github.com:kabiranwar/test-sourcecode.git"
            ]],
            extensions: [
              [$class: 'CleanBeforeCheckout'], 
              [$class: 'RelativeTargetDirectory', relativeTargetDir:'test-sourcecode']
            ],
            submoduleCfg: [],
          ])
        }
      }
    }
    stage("Read version"){
      steps{
        script{
          dir('test-sourcecode/client'){
            def inputFile = readFile('.//package.json')
            def packageJson = new JsonSlurper().parseText(inputFile)
            version=println("Version number:${packageJson.version}")
            echo $version
            

          
          }
        }
      }
    }
    /*stage("Commit"){
      steps{
        script{
          dir('test-sourcecode'){
            // sshagent(credentials: ['github-ssh-username-private-key']){
              sh """ git config --global user.name "kabiranwar" """
              sh """ git config --global user.email "anwarkabir2011@gmail.com" """
              sh "git checkout -b ${params.version}-snapshot origin/master"
              //after this we can run the git add acommit and push with jenkins command
              sh 'git add client/package.json'
              sh """git commit -m "Created new version ${params.version}" """
              // sh("git push https://${username}:${password}@github.com/kabiranwar/test-sourcecode.git")
              sh "git push origin ${params.version}-snapshot"
            // }
          }
        }
      }
    }*/
  }
  post {
    success {
      cleanWs()// deleteDirs: true
    }
  }
}