@Library('jenkinsfile-shared-library') _

def config = readYaml text: """
  APP: 'ApiSampleJava'
"""

config.keySet().each {
    env."${it}" = config[it]
}

"${pipelineSelector(env)}"(env)