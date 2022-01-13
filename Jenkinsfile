pipeline {
  agent {
    node {
      label 'kaniko-harbor'
    }

  }
  stages {
    stage('Clone JTT Geo Repo') {
      steps {
        dir(path: 'jtt-geo') {
          checkout([$class: 'GitSCM',
                      // branches: [[name: "${jtt_geo_branch}"]],
                      branches: [[name: "${env.SCM_JTT_GEO_BRANCH}"]],
                      doGenerateSubmoduleConfigurations: false,
                      extensions: [],
                      gitTool: 'Default',
                      submoduleCfg: [],
                      // userRemoteConfigs: [[url: 'https://bitbucket.di2e.net/scm/j7jtt/jtt-geo.git', credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b']]]
                      userRemoteConfigs: [[url: '${SCM_JTT_GEO_URL}', credentialsId: '${env.SCM_REPO_CREDS}']]])
        }

      }
    }

    stage('Install and Transpile JTT Geo') {
      steps {
        dir(path: 'jtt-geo') {
          sh 'npm install'
          sh 'npm run transpile'
        }

      }
    }

    stage('Clone JTT UI Repo') {
      steps {
        dir(path: 'jtt-ui') {
          checkout([$class: 'GitSCM',
                      // branches: [[name: "${jtt_ui_branch}"]],
                      branches: [[name: "${env.SCM_JTT_UI_BRANCH}"]],
                      doGenerateSubmoduleConfigurations: false,
                      extensions: [],
                      gitTool: 'Default',
                      submoduleCfg: [],
                      // userRemoteConfigs: [[url: 'https://bitbucket.di2e.net/scm/j7jtt/jtt-ui.git', credentialsId: '316724fe-6a7e-47c5-8c94-9ae95694fb5b']]]
                      userRemoteConfigs: [[url: '${SCM_JTT_UI_URL}', credentialsId: '${env.SCM_REPO_CREDS}']]])
        }

      }
    }

    stage('Install and Build JTT UI') {
      steps {
        dir(path: 'jtt-ui') {
          sh 'npm install'
          sh 'npm run build'
        }

      }
    }

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
        emailext(attachLog: true, body: "Build URL: ${RUN_DISPLAY_URL}\n \n Artifacts URL: ${RUN_ARTIFACTS_DISPLAY_URL}\n \n Jenkins Build URL (with Anchore Scan Results): https://${JENKINS_SERVER}/job/${JENKINS_PIPELINE_NAME}/lastCompletedBuild/anchore-results/\n \n Harbor Repo (with Trivy Scan Results): https://${env.HARBOR_SERVER}/harbor/projects", subject: "${BUILD_TAG}", to: "${env.EMAIL_RECPTS}")
      }
    }

  }
  tools {
    nodejs 'NodeJS 11.14.0'
  }
  environment {
    SCM_SOURCE = 'Bitbucket'
    SCM_REPO_NAME = 'jenkins-jtt-ui-pipeline'
    SCM_REPO_CREDS = 'j7-bitbucket'
    SCM_REPO_URL = 'bitbucket.di2e.net/scm/ddjtdev/jenkins-jtt-ui-pipeline.git'
    SCM_REPO_BRANCH = 'DDJTDEV-2185-jenkins-pipeline-for-building-jtt-ui'
    SCM_JTT_GEO_URL = 'bitbucket.di2e.net/scm/ddjtdev/jtt-geo-jenkins-pipeline.git'
    SCM_JTT_GEO_BRANCH = 'master'
    SCM_JTT_UI_URL = 'bitbucket.di2e.net/scm/ddjtdev/jtt-ui-jenkins-pipeline.git'
    SCM_JTT_UI_BRANCH = 'master'
    JENKINS_SERVER = 'jenkins-commercial.rke2-app.km.test'
    JENKINS_PIPELINE_NAME = 'jenkins-jtt-ui-pipeline'
    HARBOR_SERVER = 'harbor.rke2-app.km.test'
    HARBOR_REPO1 = 'eaddev'
    HARBOR_REPO2 = 'eaddev'
    IMAGE_NAME = 'jtt_ui'
    IMAGE_TAG = 'latest'
    REGISTRY_CERT_LOC = '/kaniko/ssl/km-test-certs/km_test_ca.crt'
    LOG_LEVEL = 'debug'
    CLAMAV_FILES = '/home/jenkins/agent/workspace/*'
    EMAIL_RECPTS = 'moore.kyle@idsi.com'
  }
  parameters {
    string(name: 'ticket_key', defaultValue: '', description: 'Jira Ticket Project ID')
    string(name: 'ticket_id', defaultValue: '', description: 'Jira Ticket Unique ID')
  }
}