@Library('jenkinsfile-shared-library') _

def config = readYaml text: """
  APP: 'ApiSampleJava'
  VERSION: 'v1'
  DOCKER_IMAGE: 'prietor/api-sample-java'
  DOCKERFILE_LOCATION: '.'
"""

config.keySet().each {
    env."${it}" = config[it]
}

"${pipelineSelector(env)}"(env)