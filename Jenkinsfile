#!groovy

worker_name = "worker-1||worker-2||worker-3||worker-4||worker-5||worker-6||worker-7||worker-8||worker-9"
slack_team_domain = "raccoongang"
git_credentials_id = "ef2bf63b-753b-40ac-984a-d95b6e63123b"
git_url = "https://github.com/raccoongang/edx-platform"
slack_credentials_id = "slack-secret-token"
codecov_credentials_id = "rg-codecov-edx-platform-token"
diff_target = "origin/${ghprbTargetBranch}"

/* Global slack messages */
global_start_message = "CI ${env.JOB_NAME}-test started for ${ghprbPullLink}! (<${env.BUILD_URL}|Open>)"
global_finish_message = "CI ${env.JOB_NAME}-test finished for ${ghprbPullLink}! (<${env.BUILD_URL}|Open>)"

def getSuites() {
    if (env.JOB_NAME ==~ /^.*accessibility.*$/){
        return [
            [name: 'accessibility', 'shards': ['all']],
        ]
    }
    if (env.JOB_NAME ==~ /^.*bokchoy.*$/){
        return [
            [name: 'bok-choy', 'shards': [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,]],
        ]
    }
    if (env.JOB_NAME ==~ /^.*javascript.*$/){
    return [
            [name: 'js-unit', 'shards': ['all']],
        ]
    }
    if (env.JOB_NAME ==~ /^.*lettuce.*$/){
        return [
            [name: 'lms-acceptance', 'shards': ['all']],
            [name: 'cms-acceptance', 'shards': ['all']],
        ]
    }
    if (env.JOB_NAME ==~ /^.*quality.*$/){
        return [
            [name: 'quality', 'shards': ['all']],
        ]
    }
    if (env.JOB_NAME ==~ /^.*unit.*$/){
        return [
            [name: 'commonlib-unit', 'shards': ['all']],
            [name: 'lms-unit', 'shards': [1, 2, 3, 4,]],
            [name: 'cms-unit', 'shards': ['all']],
        ]
    }
}

def startParallelSteps() {
    def suiteNames = [:]

    for (def suite in getSuites()) {
        def name = suite['name']

        for (def shard in suite['shards']) {
            if (env.JOB_NAME ==~ /^.*accessibility.*$/){
                suiteNames["${name}_${shard}"] = startAccessibility(name, shard)
            }
            if (env.JOB_NAME ==~ /^.*bokchoy.*$/){
                suiteNames["${name}_${shard}"] = startBokchoy(name, shard)
            }    
            if (env.JOB_NAME ==~ /^.*javascript.*$/){
                suiteNames["${name}_${shard}"] = startJavascript(name, shard)
            }
            if (env.JOB_NAME ==~ /^.*lettuce.*$/){
                suiteNames["${name}_${shard}"] = startLettuce(name, shard)
            }
            if (env.JOB_NAME ==~ /^.*quality.*$/){
                suiteNames["${name}_${shard}"] = startQuality(name, shard)
            }
            if (env.JOB_NAME ==~ /^.*unit.*$/){
                suiteNames["${name}_${shard}"] = startUnit(name, shard)
            }        
        }
    }
    return suiteNames
}

def startAccessibility(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false,  timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    sh './scripts/accessibility-tests.sh'
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    cobertura autoUpdateHealth: false, autoUpdateStability: false, classCoverageTargets: '95, 95, 0', coberturaReportFile: 'reports/a11y/coverage.xml', failUnhealthy: false, failUnstable: false, fileCoverageTargets: '95, 95, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '95, 95, 0', onlyStable: false, packageCoverageTargets: '95, 95, 0', sourceEncoding: 'ASCII', zoomCoverageChart: true                   
                    deleteDir()
                }
            }
        }
    }
}

def startBokchoy(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false,  timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}"]) {
                        sh './scripts/all-tests.sh'
                    }
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    junit '/^reports/bok_choy/shard_\d+/xunit.xml$/'
                    deleteDir()
                }
            }
        }
    }
}

def startJavascript(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}"]) {
                        sh './scripts/all-tests.sh'
                    }
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    junit 'reports/javascript/javascript_xunit-*.xml'
                    cobertura autoUpdateHealth: false, autoUpdateStability: false, classCoverageTargets: '95, 95, 0', coberturaReportFile: 'reports/javascript/coverage-*.xml', failUnhealthy: false, failUnstable: false, fileCoverageTargets: '95, 95, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '95, 95, 0', onlyStable: false, packageCoverageTargets: '95, 95, 0', sourceEncoding: 'ASCII', zoomCoverageChart: true
                    deleteDir()
                }
            }
        }
    }
}

def startLettuce(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}"]) {
                        sh './scripts/all-tests.sh'
                    }
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    deleteDir()
                }
            }
        }
    }
}

def startQuality(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}", "DIFF_TARGET=${diff_target}"]) {
                        sh './scripts/all-tests.sh'
                    }
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'reports/diff_quality/diff_quality_eslint.html', reportName: 'Diff Quality eslint Report', reportTitles: ''])
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'reports/diff_quality/diff_quality_pep8.html', reportName: 'Diff Quality pep8 Report', reportTitles: ''])
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'reports/diff_quality/diff_quality_pylint.html', reportName: 'Diff Quality pylint Report', reportTitles: ''])
                    deleteDir()
                }
            }
        }
    }
}

def startUnit(suite, shard) {
    return {
        node("${worker_name}") {
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])
                try {
                    withEnv(["TEST_SUITE=${suite}", "SHARD=${shard}"]) {
                        sh './scripts/all-tests.sh'
                    }
                } catch (err) {
                    slackSend channel: channel_name, color: 'danger', message: "Test ${suite}-${shard} for ${ghprbPullLink}. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                    throw err
                } finally {
                    archiveArtifacts 'reports/**'
                    stash includes: 'reports/**', name: "artifacts-${suite}-${shard}"
                    junit 'reports/**/*.xml'
                    deleteDir()
                }
            }
        }
    }
}
  
def coverageTest() {
    node("${worker_name}") {
        wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
            cleanWs()
            checkout([$class: 'GitSCM', branches: [[name: '${ghprbSourceBranch}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false,  timeout: 35]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '80bf5c5b-1fc9-41e3-bc48-2ca65c34cfea', url: "${git_url}"]]])

            withCredentials([string(credentialsId: "${codecov_credentials_id}", variable: 'CODE_COV_TOKEN')]) {
                codecov_token = env.CODE_COV_TOKEN
            }
       
            echo "Unstash unit-tests artifacts."
            unstash "artifacts-lms-unit-1"
            unstash "artifacts-lms-unit-2"
            unstash "artifacts-lms-unit-3"
            unstash "artifacts-lms-unit-4"
            unstash "artifacts-commonlib-unit-all"
            unstash "artifacts-cms-unit-all"
            
            try {
                sh """source ./scripts/jenkins-common.sh
                paver coverage -b ${diff_target}
                pip install codecov==2.0.15
                codecov --token=${codecov_token} --branch=${ghprbSourceBranch} --commit=${ghprbActualCommit} --pr=true"""
            } catch (err) {
                slackSend channel: channel_name, color: 'danger', message: "Coverage report for ${ghprbPullLink} failed. Please check build info. (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
                throw err
            } finally {
                archiveArtifacts 'reports/**'
                cobertura autoUpdateHealth: false, autoUpdateStability: false, classCoverageTargets: '95, 95, 0', coberturaReportFile: 'reports/coverage.xml', failUnhealthy: false, failUnstable: false, fileCoverageTargets: '95, 95, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '95, 95, 0', onlyStable: false, packageCoverageTargets: '95, 95, 0', sourceEncoding: 'ASCII', zoomCoverageChart: true
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'reports/diff_coverage_combined.html', reportName: 'Diff Coverage Report', reportTitles: ''])
                deleteDir()
            }                
        }
    }
}

stage('Prepare') {
    echo 'Starting the build...'
    slackSend channel: channel_name, color: 'good', message: "CI ${env.JOB_NAME}-test started for ${ghprbPullLink}! (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
}

stage('Tests') {
    parallel startParallelSteps()
}

if (env.JOB_NAME ==~ /^.*unit.*$/){
    stage('Coverage') {
        coverageTest()
    }
}

stage('Done') {
    echo 'Done! :)'
    slackSend channel: channel_name, color: 'good', message: "CI Tests finished! ${env.JOB_NAME} (<${env.BUILD_URL}|Open>)", teamDomain: "${slack_team_domain}", tokenCredentialId: "${slack_credentials_id}"
}
