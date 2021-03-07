@Library('jenkinsfile-shared-library') _

def config = readYaml text: """
  APP: 'ApiSampleJava'
  VERSION: 'v3'
  DOCKER_IMAGE: 'prietor/api-sample-java'
  DOCKERFILE_LOCATION: '.'
  SVC_NAME: 'api-sample-java'
"""

config.keySet().each {
    env."${it}" = config[it]
}

"${pipelineSelector(env)}"(env)