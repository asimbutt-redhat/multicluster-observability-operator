pipeline {
    agent {
        docker {
            image 'quay.io/rhn_support_abutt/ginkgo_1_14_2-linux-go'
            args '--network host -u 0:0'
        }
    }
    parameters {
        string(name:'HUB_CLUSTER_NAME', defaultValue: 'abutt-mycluster01', description: 'Name of Hub cluster')
        string(name:'BASE_DOMAIN', defaultValue: 'dev09.red-chesterfield.com', description: 'Base domain of Hub cluster')
        string(name:'OC_CLUSTER_USER', defaultValue: 'kubeadmin', description: 'OCP Hub User Name')
        string(name:'OC_HUB_CLUSTER_PASS', defaultValue: '', description: 'OCP Hub Password')
        string(name:'OC_HUB_CLUSTER_API_URL', defaultValue: 'https://api.abutt-mycluster01.dev09.red-chesterfield.com:6443', description: 'OCP Hub API URL')
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Test Run') {
            steps {
                sh """
                export OC_CLUSTER_USER="${params.OC_CLUSTER_USER}"
                export OC_HUB_CLUSTER_PASS="${params.OC_HUB_CLUSTER_PASS}"
                export OC_HUB_CLUSTER_API_URL="${params.OC_HUB_CLUSTER_API_URL}"
                export HUB_CLUSTER_NAME="${params.HUB_CLUSTER_NAME}"
                export BASE_DOMAIN="${params.BASE_DOMAIN}"
                if [[ -z "${HUB_CLUSTER_NAME}" || -z "${BASE_DOMAIN}" || -z "${OC_CLUSTER_USER}" || -z "${OC_HUB_CLUSTER_PASS}" || -z "${OC_HUB_CLUSTER_API_URL}" ]]; then
                    echo "Aborting test.. OCP HUB details are required for the test execution"
                    exit 1
                else
                    oc login --insecure-skip-tls-verify -u \$OC_CLUSTER_USER -p \$OC_HUB_CLUSTER_PASS \$OC_HUB_CLUSTER_API_URL
                    export KUBECONFIG=~/.kube/config
                    go mod vendor && ginkgo build ./tests/pkg/tests/
                    cd tests
                    cp resources/options.yaml.template resources/options.yaml
                    /usr/local/bin/yq e -i '.options.hub.name="'"\$HUB_CLUSTER_NAME"'"' resources/options.yaml
                    /usr/local/bin/yq e -i '.options.hub.baseDomainame="'"\$BASE_DOMAIN"'"' resources/options.yaml
                    ginkgo -v pkg/tests/ -- -options=resources/options.yaml -v=3
                fi
                """
            }
        }


    }
    post {
        always {
            archiveArtifacts artifacts: 'tests/pkg/tests/*.xml', followSymlinks: false
            junit 'tests/pkg/tests/*.xml'
        }
    }
}