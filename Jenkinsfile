@Library('jenkinsfile-shared-library') _

def config = readYaml text: """
  APP: 'ApiSampleJava'
  VERSION: 'v1'
  DOCKER_IMAGE: 'prietor/apiSampleJava'
  DOCKERFILE_LOCATION: '.'
"""

config.keySet().each {
    env."${it}" = config[it]
}

"${pipelineSelector(env)}"(env)