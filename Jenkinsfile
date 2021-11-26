pipeline {
    agent {
        node { label 'docker' } 
    }
    parameters {
        string(name:'HUB_CLUSTER_NAME', defaultValue: 'abutt-mycluster01', description: 'Name of Hub cluster')
        string(name:'BASE_DOMAIN', defaultValue: 'dev09.red-chesterfield.com', description: 'Base domain of Hub cluster')
        string(name:'OC_CLUSTER_USER', defaultValue: 'kubeadmin', description: 'OCP Hub User Name')
        string(name:'OC_HUB_CLUSTER_PASS', defaultValue: '', description: 'OCP Hub Password')
        string(name:'OC_HUB_CLUSTER_API_URL', defaultValue: 'https://api.abutt-mycluster01.dev09.red-chesterfield.com:6443', description: 'OCP Hub API URL')
        // extendedChoice(
        //     name: 'GIT_BRANCH',
        //     description: 'GIT Branch to use for the job',
        //     visibleItemCount: 10,
        //     multiSelectDelimiter: ',',
        //     defaultValue: "main" ,
        //     type: 'PT_SINGLE_SELECT',
        //     groovyScript: '''
        //             import jenkins.*
        //             import jenkins.model.* 
        //             import hudson.*
        //             import hudson.model.*
        //             def gitURL="github.com/asimbutt-redhat/multicluster-observability-operator.git"
        //             def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        //                     com.cloudbees.plugins.credentials.Credentials.class,
        //                     Jenkins.instance,
        //                     null,
        //                     null
        //             );

        //             for (creds in jenkinsCredentials) {
        //             if(creds.id == "538c7b8c-8b59-448a-9703-88aeb821e728"){
        //                 def username = creds.username;
        //                 def password = creds.password;
        //                 def command = "git ls-remote -h https://" + username + ":" + password + "@" + gitURL
        //                 def proc = command.execute()
        //                 proc.waitFor()         
        //                     if ( proc.exitValue() != 0 ) {
        //                         println "Error, ${proc.err.text}"
        //                         System.exit(0)
        //                     }     
        //                 def branches = proc.in.text.readLines().collect {
        //                     it.replaceAll(/[a-z0-9]*\trefs\/heads\//, '') 
        //                     }   
        //                 return branches.join(",")                        
        //                 }
        //             }
        //     '''
        // )
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Test Setup') {
            steps {
                sh """
                export OC_CLUSTER_USER="${params.OC_CLUSTER_USER}"
                export OC_HUB_CLUSTER_PASS="${params.OC_HUB_CLUSTER_PASS}"
                export OC_HUB_CLUSTER_API_URL="${params.OC_HUB_CLUSTER_API_URL}"
                if [[ -z "${OC_CLUSTER_USER}" || -z "${OC_HUB_CLUSTER_PASS}" || -z "${OC_HUB_CLUSTER_API_URL}" ]]; then
                    echo "Aborting test.. OCP connection details are required for the test execution"
                    exit 1
                else
                    oc login --insecure-skip-tls-verify -u \$OC_CLUSTER_USER -p \$OC_HUB_CLUSTER_PASS \$OC_HUB_CLUSTER_API_URL
                    export KUBECONFIG=~/.kube/config
                    make test-e2e-setup
                fi
                """
            }
        }
        stage('Test Run') {
            agent {
                docker {
                    image 'quay.io/rhn_support_abutt/ginkgo_1_14_2-fedora-32'
                    args '--network host -u 0:0'
                    reuseNode true
                }
            }
            steps {
                sh """
                export HUB_CLUSTER_NAME="${params.HUB_CLUSTER_NAME}"
                export BASE_DOMAIN="${params.BASE_DOMAIN}"
                if [[ -z "${HUB_CLUSTER_NAME}" || -z "${BASE_DOMAIN}" ]]; then
                    echo "Aborting test.. HUB details are required for the test execution"
                    exit 1
                else
                    export KUBECONFIG=~/.kube/config
                    cd tests
                    cp resources/options.yaml.template resources/options.yaml
                    /usr/local/bin/yq e -i '.options.hub.name="'"\$HUB_CLUSTER_NAME"'"' resources/options.yaml
                    /usr/local/bin/yq e -i '.options.hub.baseDomainame="'"\$BASE_DOMAIN"'"' resources/options.yaml
                    ginkgo -v pkg/tests/ -- -options=../../resources/options.yaml -v=3
                fi
                """
            }
        }


    }
    post {
        always {
            archiveArtifacts artifacts: 'results/*', followSymlinks: false
            junit 'results/*.xml'
        }
    }
}