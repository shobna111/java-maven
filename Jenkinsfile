podTemplate(containers: [
    containerTemplate(name: 'maven', image: 'maven', command: 'sleep', args: 'infinity')
]) {
  node(POD_LABEL) {
    checkout scm

    container('maven') {
      stage('Build & Test') {
        sh 'mvn -B -ntp -Dmaven.test.failure.ignore verify'
      }

      stage('Publish to Nexus') {
        withCredentials([usernamePassword(
          credentialsId: 'nexus-creds',
          usernameVariable: 'NEXUS_USER',
          passwordVariable: 'NEXUS_PASS'
        )]) {
          sh '''
            ARTIFACT_PATH=$(find target -name "*.jar" | head -n 1)
            VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
            GROUP_ID=$(mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout | tr '.' '/')
            ARTIFACT_ID=$(mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout)
            curl -v -u $NEXUS_USER:$NEXUS_PASS \
              --upload-file $ARTIFACT_PATH \
              http://34.203.14.3:8081/repository/releases/$GROUP_ID/$ARTIFACT_ID/$VERSION/$ARTIFACT_ID-$VERSION.jar
          '''
        }
      }
    }

    stage('Test Report') {
      junit '**/target/surefire-reports/TEST-*.xml'
    }
  }
}
