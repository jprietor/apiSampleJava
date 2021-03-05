@Library('jenkinsfile-shared-library') _

def config = readYaml text: """
  APP: 'HelloWorld'
"""

config.keySet().each {
    env."${it}" = config[it]
}

"${pipelineSelector(env)}"(env)