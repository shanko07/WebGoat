pipeline {
  agent any

  environment {
    PROJECT_NAME = "Shanko-Webgoat-Demo"
    PROJECT_VERSION = "dev"
    POLARIS_ACCESS_TOKEN = credentials('polaris-token')
    BLACKDUCK_ACCESS_TOKEN = credentials('BlackDuck-AuthToken')
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn -e clean package -DskipTests'
      }
    }
    stage('SAST - Coverity') {
      steps {
        echo "Running Coverity on Polaris"
        sh '''
          rm -fr /tmp/polaris
          wget -q ${POLARIS_SERVER_URL}/api/tools/polaris_cli-linux64.zip
          unzip -j polaris_cli-linux64.zip -d /tmp
          /tmp/polaris --persist-config --co project.name="${PROJECT_NAME}" --co project.branch="${PROJECT_VERSION}" --co capture.build.buildCommands='[{"shell":["mvn", "-e", "clean", "package", "-DskipTests"]}]' --co capture.build.cleanCommands='[{"shell":["mvn", "clean"]}]' --co analyze.coverity.cov-analyze='["--all", "--webapp-security"]'  configure
          /tmp/polaris analyze -w
        '''
      }
    }
    stage('SCA - Blackduck') {
      steps {
        echo "Running Blackduck"
        sh '''
          rm -fr /tmp/detect7.sh
          curl -s -L https://detect.synopsys.com/detect7.sh > /tmp/detect7.sh
          bash /tmp/detect7.sh --blackduck.url="${BLACKDUCK_URL}" --blackduck.api.token="${BLACKDUCK_ACCESS_TOKEN}" --detect.project.name="${PROJECT_NAME}" --detect.project.version.name=${PROJECT_VERSION} --blackduck.trust.cert=true
        '''
      }
    }
    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
  }
}
