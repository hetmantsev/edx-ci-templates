#!groovy

def makeNode(suite, shard) {
  return {
    echo "I am ${suite}:${shard}, and the worker is yet to be started!"

    node('unit-test-worker-1 || unit-test-worker-2 || unit-test-worker-3 || unit-test-worker-4 || unit-test-worker-5') {
      // Cleaning up previous builds. Heads up! Not sure if `WsCleanup` actually works.
      step([$class: 'WsCleanup'])

      checkout([$class: 'GitSCM', branches: [[name: 'open-release/ficus.master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: 'https://github.com/edx/edx-platform']]])

      sh 'git log --oneline | head'

      timeout(time: 55, unit: 'MINUTES') {
        echo "Hi, it is me ${suite}:${shard} again, the worker just started!"

        try {
          if (suite == 'accessibility') {
            sh './scripts/accessibility-tests.sh'
          } else {
            withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}"]) {
              sh './scripts/all-tests.sh'
            }
          }
        } finally {
          archiveArtifacts 'reports/**, test_root/log/**'

          try {
            junit 'reports/**/*.xml'
          } finally {
            // This works, but only for the current build files.
            deleteDir()
          }
        }
      }
    }
  }
}

def getSuites() {
  return [
    [name: 'lms-unit', 'shards': [
      1,
      2,
      3,
      4,
    ]],
    [name: 'cms-unit', 'shards': ['all']],
  ]
}

def buildParallelSteps() {
  def parallelSteps = [:]

  for (def suite in getSuites()) {
    def name = suite['name']

    for (def shard in suite['shards']) {
      parallelSteps["${name}_${shard}"] = makeNode(name, shard)
    }
  }

  return parallelSteps
}

stage('Prepare') {
  echo 'Starting the build...'
  echo 'It it always nice to have a green checkmark :D'
}

stage('Test') {
  // This commit should be removed via
  parallel buildParallelSteps()
}

stage('Creating coverage report') {
  
}

stage('Done') {
  echo 'I am done, hurray!'
}
