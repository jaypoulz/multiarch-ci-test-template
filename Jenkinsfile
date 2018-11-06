properties(
  [
    parameters(
      [
        [$class: 'ValidatingStringParameterDefinition',
         defaultValue: 'x86_64,ppc64le',
         description: 'A comma separated list of architectures to run the test on. Valid values include [x86_64, ppc64le, aarch64, s390x].',
         failedValidationMessage: 'Invalid architecture. Valid values are [x86_64, ppc64le, aarch64, s390x].',
         name: 'ARCHES',
         regex: '^(?:x86_64|ppc64le|aarch64|s390x)(?:,\\s*(?:x86_64|ppc64le|aarch64|s390x))*$'
        ],
        string(
          defaultValue: 'https://github.com/redhat-multiarch-qe/multiarch-ci-libraries',
          description: 'Repo for shared libraries.',
          name: 'LIBRARIES_REPO'
        ),
        string(
          defaultValue: 'v1.3.0',
          description: 'Git reference to the branch or tag of shared libraries.',
          name: 'LIBRARIES_REF'
        ),
        string(
          defaultValue: '',
          description: 'Repo for tests to run. If left blank, the current repo is assumed (*note* this default will only work for multibranch pipelines).',
          name: 'TEST_REPO'
        ),
        string(
          defaultValue: '',
          description: 'Git reference to the branch or tag of the tests repo.',
          name: 'TEST_REF'
        ),
        string(
          defaultValue: 'tests',
          description: 'Directory containing tests to run. Should at least one of the follow: an ansible-playbooks directory containing one or more test directories each of which having a playbook.yml, a scripts directory containing one or more test directories each of which having a run-test.sh',
          name: 'TEST_DIR'
        ),
        string(
          defaultValue: '',
          description: 'Contains the CI_MESSAGE for a message bus triggered build.',
          name: 'CI_MESSAGE'
        )
      ]
    )
  ]
)

library(
  changelog: false,
  identifier: "multiarch-ci-libraries@${params.LIBRARIES_REF}",
  retriever: modernSCM([$class: 'GitSCMSource',remote: "${params.LIBRARIES_REPO}"])
)

List arches = params.ARCHES.tokenize(',')
def errorMessages = ''
def config = MAQEAPI.v1.getProvisioningConfig(this)
config.mode = 'JNLP'
config.jobgroup = 'multiarch-qe'

MAQEAPI.v1.runParallelMultiArchTest(
  this,
  arches,
  config,
  { host ->
    /*********************************************************/
    /* TEST BODY                                             */
    /* @param host               Provisioned host details.   */
    /*********************************************************/
    stage ('Download Test Files') {
      downloadTests()
    }

    stage ('Run Test') {
      runTests(config, host)
    }

    /*****************************************************************/
    /* END TEST BODY                                                 */
    /* Do not edit beyond this point                                 */
    /*****************************************************************/
  },
  { Exception exception, def host ->
    def error = "Exception ${exception} occured on ${host.arch}\n"
    errorMessages += error
    currentBuild.result = 'FAILURE'
  }
)
