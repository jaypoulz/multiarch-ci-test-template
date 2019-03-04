// Define the openshift pod name, docker repo url, namespace, and service acct
// for the DSL pod template.
env.openshiftDockerRegistryUrl = '172.30.1.1:5000'
Map config = [
    podName:"hdsl-${UUID.randomUUID()}",
    source:'openshift',
    openshift_namespace:'redhat-multiarch-qe',
    openshift_service_account:'jenkins',
    jnlp_image_name:'macit-jenkins-slave',
    jnlp_tag:'v2.0.0',
    beaker_credentials_id:'redhat-multiarch-qe-keytab',
    beaker_configuration_files: [
        [id:'redhat-multiarch-qe-bkrconf', path:'/etc/beaker/client.conf', permissions:'644'],
        [id:'redhat-multiarch-qe-krbconf', path:'/etc/krb5.conf', permissions:'644'],
        [id:'redhat-multiarch-qe-root-it-cert', path:'/etc/beaker/RH-IT-Root-CA.crt', permissions:'644'],
    ],
    beaker_configuration_commands: [
        [
            containerName: 'linchpin-executor',
            workspace: 'linchpin/creds',
            executable: 'kinit',
            options: "-kt \"beaker.creds\"",
            command: 'jenkins/multiarch-qe-jenkins.rhev-ci-vms.eng.rdu2.redhat.com',
            args:''
        ],
        [
            containerName: 'linchpin-executor',
            workspace: '',
            executable: 'bkr',
            options: '',
            command: 'whoami',
            args:''
        ],
    ],
    verbose: true,
]

// Create the HDSL podTemplate
withHdslPod(config) {
    node(hdslPodName) {
        stage("Pre-Flight"){
            deleteDir()
            git(branch:'hdsl',
                url:'https://github.com/jaypoulz/multiarch-ci-test-template')
        }

        stage("Parse Configuration"){
            config = parseConfig(filename:'job.yml') << config
            echo("$config")
        }

        parallelizeInfra(config) {
            singleInfraConfig ->
            try {
                stage('Clone Test Dir') {
                    git(branch:'hdsl',
                        url:'https://github.com/jaypoulz/multiarch-ci-test-template')
                }

                stage("Deploy Infra") {
                    deployInfra(singleInfraConfig)
                }

                stage("Configure Infra") {
                    configureInfra(singleInfraConfig)
                }

                stage("Execute Tests") {
                    executeTests(singleInfraConfig)
                }
            } catch (Exception e) {
                echo("${e}")
            }

            try {
                stage("Destroy Infra") {
                    destroyInfra(singleInfraConfig)
                }
            } catch (Exception e) {
                echo("${e}")
            }

            stage("Archive") {
                saveArtifacts(singleInfraConfig)
            }
        }
    }
}
