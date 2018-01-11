#!groovy

def startTests(suite, shard) {
  return {
    echo "I am ${suite}:${shard}, and the worker is yet to be started!"

    node('${suite}-${shard}-worker') {
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
	  stash includes: 'reports/**, test_root/log/**', name: "artifacts-${suite}-${shard}"

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
    [name: 'cms-unit', 'shards': ['all']],
  ]
}

def buildParallelSteps() {
  def parallelSteps = [:]

  for (def suite in getSuites()) {
    def name = suite['name']

    for (def shard in suite['shards']) {
      parallelSteps["${name}_${shard}"] = startTests(name, shard)
    }
  }

  return parallelSteps
}

stage('Prepare') {
  echo 'Starting the build...'
}

stage('Test') {
  parallel buildParallelSteps()
}

stage('Analysis') {
  node('coverage-report-worker') {
      // Cleaning up previous builds. Heads up! Not sure if `WsCleanup` actually works.
      step([$class: 'WsCleanup'])

      checkout([$class: 'GitSCM', branches: [[name: 'open-release/ficus.master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: 'https://github.com/edx/edx-platform']]])

      sh 'git log --oneline | head'

      timeout(time: 55, unit: 'MINUTES') {
        echo "Hi, it is me coverage agent again, the worker just started!"

        try {
	  unstash 'artifacts-lms-unit-1'
	  unstash 'artifacts-lms-unit-2'
	  unstash 'artifacts-lms-unit-3'
	  unstash 'artifacts-lms-unit-4'
	  unstash 'artifacts-cms-unit-all'
	  sh ("""
	  find reports/.coverage* -type f -exec sed -i 's/lms-unit-1/coverage-report/g' {} \;
	  find reports/.coverage* -type f -exec sed -i 's/lms-unit-2/coverage-report/g' {} \;
	  find reports/.coverage* -type f -exec sed -i 's/lms-unit-3/coverage-report/g' {} \;
	  find reports/.coverage* -type f -exec sed -i 's/lms-unit-4/coverage-report/g' {} \;
	  find reports/.coverage* -type f -exec sed -i 's/cms-unit-1/coverage-report/g' {} \;
	  """)
          sh 'source ~/edx-venv/bin/activate'
	  sh 'paver coverage'
          } finally {
            archiveArtifacts 'reports/**, test_root/log/**'
 	    cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'reports/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
          }
      }
    }
}

stage('Done') {
  echo 'Done! :)'
}
