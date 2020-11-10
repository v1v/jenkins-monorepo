#!/usr/bin/env groovy

@Library('shared-library@current') _

pipeline {
  agent { label 'linux && immutable' }
  options {
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20', daysToKeepStr: '30'))
    timestamps()
    ansiColor('xterm')
  }
  triggers {
    issueCommentTrigger('(?i)^/test\\W+.*$')
  }
  parameters {
    booleanParam(name: 'macosTest', defaultValue: false, description: 'Allow macOS stages.')
  }
  stages {
    stage('Build&Test') {
      steps {
        script {
          def mapParallelTasks = [:]
          def content = readYaml(file: 'Jenkinsfile.yml')
          content['projects'].each { projectName ->
            generateStages(project: projectName).each { k,v ->
              mapParallelTasks["${k}"] = v
            }
          }
          parallel(mapParallelTasks)
        }
      }
    }
  }
}

/**
* This method is the one used for running the parallel stages, therefore
* its arguments are passed by the stagesProject step.
*/
def generateStages(Map args = [:]) {
  def projectName = args.project
  def mapParallelStages = [:]
  def fileName = "${projectName}/Jenkinsfile.yml"
  if (fileExists(fileName)) {
    def content = readYaml(file: fileName)
    if (whenProject(project: projectName, content: content?.when)) {
      mapParallelStages = stagesProject(project: projectName, content: content, function: new RunCommand(steps: this))
    }
  } else {
    log(level: 'WARN', text: "${fileName} file does not exist. Please review the top-level Jenkinsfile.yml")
  }
  return mapParallelStages
}

/**
* This class is the one used for running the parallel stages, therefore
* its arguments are passed by the stagesProject step.
*
*  What parameters/arguments are supported:
*    - label -> the worker labels
*    - project -> the name of the project that should match with the folder name.
*    - content -> the specific stage data in the <project>/Jenkinsfile.yml
*    - context -> the name of the stage, normally <project>-<stage>(-<platform>)?
*/
class RunCommand extends mono.Function {
  public RunCommand(Map args = [:]){
    super(args)
  }
  public run(Map args = [:]){
    if(args?.content?.containsKey('build')) {
      steps.buildRun(context: args.context, command: args.content.build, directory: args.project, label: args.label, id: args.id)
    }
    if(args?.content?.containsKey('test')) {
      steps.testRun(context: args.context, command: args.content.test, directory: args.project, label: args.label, id: args.id)
    }
  }
}

def buildRun(Map args = [:]) {
  def context = args.context
  def command = args.command
  def directory = args.get('directory', '')
  node(args.label) {
    echo 'Prepare some context for running the build'
    if(isUnix()) {
      sh(label: "${args.id?.trim() ? args.id : env.STAGE_NAME} - ${command}", script: "${command}")
    } else {
      bat(label: "${args.id?.trim() ? args.id : env.STAGE_NAME} - ${command}", script: "${command}")
    }
  }
}

def testRun(Map args = [:]) {
  def context = args.context
  def command = args.command
  def directory = args.get('directory', '')
  node(args.label) {
    echo 'Prepare some context for running the tests'
    if(isUnix()) {
      sh(label: "${args.id?.trim() ? args.id : env.STAGE_NAME} - ${command}", script: "${command}")
    } else {
      bat(label: "${args.id?.trim() ? args.id : env.STAGE_NAME} - ${command}", script: "${command}")
    }
  }
}

