#!groovy

def startTests(suite, shard) {
  return {
    echo "I am ${suite}:${shard}, and the worker is yet to be started!"

    node("${suite}-${shard}-worker") {
      // Cleaning up previous builds. Heads up! Not sure if `WsCleanup` actually works.
      step([$class: 'WsCleanup'])

      checkout scm

      sh 'git log --oneline | head'

      timeout(time: 55, unit: 'MINUTES') {
        echo "Hi, it is me ${suite}:${shard} again, the worker just started!"
	githubNotify account: "hetmantsev", context: "${suite}-${shard}", credentialsId: "hetmantsev", repo: "edx-platform", sha: "2295341f79d0c7f38425839a9e9067490ef0c7e1", status: "PENDING", targetUrl: "https://jenkins.raccoongang.com"

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
	    githubNotify account: "hetmantsev", context: "${suite}-${shard}", credentialsId: "0fa04582-7bd4-4be9-bd80-0133a235af9e", repo: "edx-platform", sha: "2295341f79d0c7f38425839a9e9067490ef0c7e1", status: "SUCCESS", targetUrl: "https://jenkins.raccoongang.com"
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
    [name: 'cms-unit', 'shards': ['all'],
    [name: 'commonlib-unit', 'shards': ['all']]
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

      checkout scm

      sh 'git log --oneline | head'

      timeout(time: 55, unit: 'MINUTES') {
        echo "Hi, it is me coverage agent again, the worker just started!"

        try {
	  unstash 'artifacts-lms-unit-1'
	  unstash 'artifacts-lms-unit-2'
	  unstash 'artifacts-lms-unit-3'
	  unstash 'artifacts-lms-unit-4'
	  unstash 'artifacts-cms-unit-all'
          unstash 'artifacts-commonlib-unit-all'
	  withEnv(["TARGET_BRANCH=open-release/ficus.master"]) {
              sh './scripts/jenkins-report.sh'
          }
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
