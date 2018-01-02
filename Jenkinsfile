pipeline {
	agent none
    stages {
        stage('Testing') {
			parallel {
                stage('lms-unit-1') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '1'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'lms-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
				stage('lms-unit-2') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '2'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'lms-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
				stage('lms-unit-3') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '3'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'lms-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
				stage('lms-unit-4') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '4'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'lms-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
				stage('cms-unit-1') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '1'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'cms-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
				stage('common-unit-1') {
                    steps {
						build job: 'unit-test-template', parameters: [[$class: 'StringParameterValue', name: 'sha1', value: 'open-release/ficus.master'], [$class: 'StringParameterValue', name: 'SHARDS', value: '1'], [$class: 'StringParameterValue', name: 'TEST_SUITE', value: 'common-unit'], [$class: 'StringParameterValue', name: 'PARENT_BUILD', value: 'unit-test-pipeline']]
                    }
                    post {
                        always {
                            junit '**/TEST-*.xml'
							archiveArtifacts 'edx-platform/reports/**/*,test_root/log/**/*.png,test_root/log/**/*.log, test_root/log/**/hars/*.har,**/nosetests.xml,**/TEST-*.xml,*.log'
                        }
                    }
                }
            }
        }
    }
}
