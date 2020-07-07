import groovy.json.JsonSlurper
import groovy.json.JsonOutput

properties([parameters([string(defaultValue: '1.0.0', description: '', name: 'version', trim: false), string(defaultValue: '1.1.0', description: '', name: 'nextversion', trim: false)])])

pipeline {

  agent any

  options {
    timestamps()
    withFolderProperties()
  }
  environment {
    TIMESTAMP = new java.text.SimpleDateFormat('yyyyMMdd').format(new Date())
    image = "dummy_image"
    //version = "2.1.0"
    //version = "0.1.${env.BUILD_NUMBER}"
    version = "${params.version}"
    nextversion = "${params.nextversion}"
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
        echo "next version is : ${nextversion}"
        echo "branch name is : ${branch_name}"
        // sh 'printenv'
      }
    }


    stage('Checkout') {
      steps {
        checkout scm
        script {
          repo = checkout([
            $class: 'GitSCM',
            branches: [
              [name: '*/master']
            ],
            doGenerateSubmoduleConfigurations: false,
            userRemoteConfigs: [[
              credentialsId: 'github-account-username-password',
              url: "https://github.com/kabiranwar/test-sourcecode.git"
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
    stage("Change version"){
      steps{
        script{
          dir('test-sourcecode/client'){
            def inputFile = readFile('.//package.json')
            def packageJson = new JsonSlurper().parseText(inputFile)
            println("Version number:${packageJson.version}")

            if (packageJson.version==params.version){
              println ("You can not have version ${packageJson.version}")
            }else{
              println "Wollah you have new version to add"
              //Add the new version to object of package.json file
              packageJson.version=params.version
              //Print the newly created json object of package,json  file
              println(packageJson)
              //convert packagejson object to json string to print or save as file
              def newPackageFile = JsonOutput.toJson(packageJson)
              //print the json object as string
              println(newPackageFile)
              //write json string to file with write file 
              //it will replace ols json files version with new json file
              writeFile(file: 'package.json', text:newPackageFile.toString())
              // to print json file which is newly created just reading the file and printing it 
              // this line can be ignored 
              inputFile = readFile('.//package.json')
              //println("-------------------------------------------------------------")
              //println(inputFile)
              //after this we can run the git add acommit and push with jenkins command
            }
          }
        }
      }
    }
  }
}  