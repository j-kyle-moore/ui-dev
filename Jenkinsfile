def jtt_geo_branch = "dev"
def jtt_ui_branch = "dev"
def rtdb_branch = "dev"
def params_ticket_key = ""
def params_ticket_id = ""
def num_ticket_branches = 0

pipeline {
  // agent any
  agent {
    node {
      label 'kaniko-harbor'
    }
  }
  parameters {
    string(name: 'ticket_key', defaultValue: '', description: 'Jira Ticket Project ID')
    string(name: 'ticket_id', defaultValue: '', description: 'Jira Ticket Unique ID')
  }

  environment {
    SCM_SOURCE = "Bitbucket"
    SCM_REPO_NAME = "jenkins-jtt-ui-pipeline"
    SCM_REPO_CREDS = "j7-bitbucket"
    SCM_REPO_URL = "bitbucket.di2e.net/scm/ddjtdev/jenkins-jtt-ui-pipeline.git"
    SCM_REPO_BRANCH = "DDJTDEV-2185-jenkins-pipeline-for-building-jtt-ui"

    SCM_JTT_GEO_URL = "bitbucket.di2e.net/scm/ddjtdev/jtt-geo-jenkins-pipeline.git"
    SCM_JTT_GEO_BRANCH = "master"

    SCM_JTT_UI_URL = "bitbucket.di2e.net/scm/ddjtdev/jtt-ui-jenkins-pipeline.git"
    SCM_JTT_UI_BRANCH = "master"

    JENKINS_SERVER = "jenkins-commercial.rke2-app.km.test"
    JENKINS_PIPELINE_NAME = "jenkins-jtt-ui-pipeline"

    HARBOR_SERVER = "harbor.rke2-app.km.test"
    HARBOR_REPO1 = "eaddev"
    HARBOR_REPO2 = "eaddev"
    IMAGE_NAME = "jtt_ui"
    IMAGE_TAG = "latest"

    REGISTRY_CERT_LOC = "/kaniko/ssl/km-test-certs/km_test_ca.crt"
    LOG_LEVEL = "debug"

    CLAMAV_FILES = "/home/jenkins/agent/workspace/*"

    EMAIL_RECPTS = "moore.kyle@idsi.com"
  }

  tools {
    nodejs 'NodeJS 11.14.0'
  }
  stages {
    // stage('Prepare Workspace') {
    //   steps {
    //   	cleanWs()
    //     script {
    //       params_ticket_key = "${params.ticket_key}"
    //       params_ticket_id = "${params.ticket_id}"
    //       final String url = "https://jira.di2e.net/rest/dev-status/1.0/issue/detail?issueId=$params_ticket_id&applicationType=stash&dataType=pullrequest"
    //
    //       withCredentials([usernameColonPassword(credentialsId: "cdd1102c-beb3-4039-a3a5-a03d5ff6035e", variable: "JENKINS_API_TOKEN")]) {
    //
    //         final def (String response, int code) = sh(script: 'curl -s -H "Authorization: Basic bWF0dGhldy5hdGtpbnM6MndzeGNkZTNAV1NYQ0RFIw==" "' + url + '"', returnStdout: true).trim().tokenize("\n")
    //
    //         //echo "HTTP response: $response"
    //         writeFile file: 'issueDevStatus.json', text: response
    //
    //         def num_branches = sh(script: "jq '.detail[0].branches | length' issueDevStatus.json", returnStdout: true).trim()
    //         num_ticket_branches = Integer.parseInt(num_branches)
    //         echo "Branches for this ticket: ${num_ticket_branches}"
    //
    //         for (int i = 0; i < num_ticket_branches; i++) {
    //           def repo_name = sh(script: "jq -r '.detail[0].branches[$i].repository.name' issueDevStatus.json", returnStdout: true).trim()
    //           echo "Repository: ${repo_name}"
    //
    //           if (repo_name.equalsIgnoreCase("jtt-geo")) {
    //             echo "True for ${repo_name}"
    //             jtt_geo_branch = sh(script: "jq -r '.detail[0].branches[$i].name\' issueDevStatus.json", returnStdout: true).trim()
    //           } else if (repo_name.equalsIgnoreCase("jtt-ui")) {
    //             echo "True for ${repo_name}"
    //             jtt_ui_branch = sh(script: "jq -r '.detail[0].branches[$i].name\' issueDevStatus.json", returnStdout: true).trim()
    //           } else if (repo_name.equalsIgnoreCase("rtdb")) {
    //             echo "True for ${repo_name}"
    //             rtdb_branch = sh(script: "jq -r '.detail[0].branches[$i].name\' issueDevStatus.json", returnStdout: true).trim()
    //           }
    //         }
    //
    //         echo "Jira Ticket Project ID: ${params.ticket_key}"
    //         echo "Jira Ticket Unique ID: ${params.ticket_id}"
    //         echo "Building JTT-Geo Branch: ${jtt_geo_branch}"
    //         echo "Building JTT-UI Branch: ${jtt_ui_branch}"
    //         echo "Building RTDB Branch: ${rtdb_branch}"
    //       }
    //     }
    //   }
    // }
    stage('Clone JTT Geo Repo') {
      steps {
        dir('jtt-geo') {
          checkout([$class: 'GitSCM',
            // branches: [[name: "${jtt_geo_branch}"]],
            branches: [[name: "${env.SCM_JTT_GEO_BRANCH}"]],
            doGenerateSubmoduleConfigurations: false,
            extensions: [],
            gitTool: 'Default',
            submoduleCfg: [],
            // userRemoteConfigs: [[url: 'https://bitbucket.di2e.net/scm/j7jtt/jtt-geo.git', credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b']]]
            userRemoteConfigs: [[url: '${SCM_JTT_GEO_URL}', credentialsId: '${env.SCM_REPO_CREDS}']]]
          )
        }
      }
    }
    stage('Install and Transpile JTT Geo') {
      steps {
        dir('jtt-geo') {
          sh 'npm install'
          sh 'npm run transpile'
        }
      }
    }
    stage('Clone JTT UI Repo') {
      steps {
        dir('jtt-ui') {
          checkout([$class: 'GitSCM',
            // branches: [[name: "${jtt_ui_branch}"]],
            branches: [[name: "${env.SCM_JTT_UI_BRANCH}"]],
            doGenerateSubmoduleConfigurations: false,
            extensions: [],
            gitTool: 'Default',
            submoduleCfg: [],
            // userRemoteConfigs: [[url: 'https://bitbucket.di2e.net/scm/j7jtt/jtt-ui.git', credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b']]]
            userRemoteConfigs: [[url: '${SCM_JTT_UI_URL}', credentialsId: '${env.SCM_REPO_CREDS}']]]
          )
        }
      }
    }
    stage('Install and Build JTT UI') {
      steps {
        dir('jtt-ui') {
          sh 'npm install'
          sh 'npm run build'
        }
      }
    }
    // stage('Build Hardened JTT UI Image') {
    //   steps {
    //     dir('jtt-ui') {
    //       // created credentials for my matthew.atkins account to the Harbor on registry1.dso.mil
    //       withDockerRegistry([credentialsId: '2dd9610b-e4f0-496a-b2a4-8deeea8fb112', url: 'https://registry1.dso.mil']) {
    //         sh "docker build -t docker-J7JTT.di2e.net/jtt-ui:${env.BUILD_ID} ."
    //       }
    //     }
    //   }
    // }
    // stage('Push JTT UI Docker Image') {
    //   steps {
    //     dir('jtt-ui') {
    //       // created credentials for my matthew.atkins account to the Nexus on DI2E
    //       withDockerRegistry([credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b', url: 'https://docker-J7JTT.di2e.net']) {
    //         sh "docker push docker-J7JTT.di2e.net/jtt-ui:${env.BUILD_ID}"
    //       }
    //     }
    //   }
    // }

    stage('Build and push to Harbor with Kaniko') {
      steps {
        echo 'Build and Push with Kaniko'
        container(name: 'kaniko-harbor') {
          sh '''
          /kaniko/executor --dockerfile `pwd`/Dockerfile  --context `pwd` --destination=${HARBOR_SERVER}/${HARBOR_REPO1}/${IMAGE_NAME}:${IMAGE_TAG} --destination=${HARBOR_SERVER}/${HARBOR_REPO1}/${IMAGE_NAME}:$BUILD_NUMBER --verbosity=${LOG_LEVEL} --registry-certificate ${HARBOR_SERVER}=${REGISTRY_CERT_LOC}
          '''
        }
      }
    }

    stage('Scan with Anchore') {
            steps {
                sh 'echo "$HARBOR_SERVER/$HARBOR_REPO1/$IMAGE_NAME:$IMAGE_TAG" ${WORKSPACE}/Dockerfile > anchore_images'
                anchore 'anchore_images'
            }
        }

    stage('Send Email with Build Results') {
          steps {
          	emailext attachLog: true, body: "Build URL: ${RUN_DISPLAY_URL}\n \n Artifacts URL: ${RUN_ARTIFACTS_DISPLAY_URL}\n \n Jenkins Build URL (with Anchore Scan Results): https://${JENKINS_SERVER}/job/${JENKINS_PIPELINE_NAME}/lastCompletedBuild/anchore-results/\n \n Harbor Repo (with Trivy Scan Results): https://${env.HARBOR_SERVER}/harbor/projects", subject: "${BUILD_TAG}", to: "${env.EMAIL_RECPTS}"
            // emailext attachLog: true, body: "test", subject: "test", to: "${env.EMAIL_RECPTS}"
      }
    }
  }
}
    // stage('Bump Helm Chart Version') {
    //   environment {
    //     valuesData = ''
    //     chartData = ''
    //   }
      // steps {
      //   script {
      //     sh 'git config --global credential.helper cache'
      //     sh 'git config --global push.default simple'
      //
      //     checkout([$class: 'GitSCM',
      //       branches: [[name: 'main']],
      //       doGenerateSubmoduleConfigurations: false,
      //       extensions: [],
      //       gitTool: 'Default',
      //       submoduleCfg: [],
      //       userRemoteConfigs: [[url: 'https://bitbucket.di2e.net/scm/~matthew.atkins/jtt-ui-helm.git', credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b']]]
      //     )
      //
      //     sh 'git checkout main'
      //
      //     valuesData = readYaml(file:'values.yaml')
      //     valuesData.image.tag = "${env.BUILD_ID}"
      //     valuesData.replicaCount = 1
      //     valuesData.ingress.enabled = true
      //     sh 'rm values.yaml'
      //     writeYaml(file: 'values.yaml', data: valuesData)
      //
      //     chartData = readYaml(file:'Chart.yaml')
      //     chartData.version = "${env.BUILD_ID}"
      //     chartData.description = "${params.ticket_key}"
      //     sh 'rm Chart.yaml'
      //     writeYaml(file: 'Chart.yaml', data: chartData)
      //
      //     sh 'git commit -am "bump version"'
      //
      //     sh 'git push'
      //   }
      // }
    // }
    // stage('Send Email') {
    //   steps {
    //   	emailext attachLog: true, body: "https://jtt.local.test", subject: 'JTT Test Instance Deployed', to: 'matthew.atkins@capetechsolutions.com'
    //   }
    // }
//   }
// }
