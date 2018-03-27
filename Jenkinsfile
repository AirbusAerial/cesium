def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

podTemplate(label: label, containers: [
  containerTemplate(name: 'docker-gcp', image: 'airbusaerial/docker-gcp:1.0', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'node', image: 'node:8.3.0-alpine', ttyEnabled: true, command: 'cat',
    envVars: [
      secretEnvVar(key: 'CI_NEXUS_CREDENTIALS', secretName: 'nexus-credentials', secretKey: 'username_password')]
  )],
    volumes: [
        secretVolume(secretName: 'instance-credentials', mountPath: '/secrets/google')]) {

  node(label) {
    checkout scm

    stage('Build') {
      // This will eventually have tests running here
      container('node') {
        withNPM(npmrcConfig: 'npm-all') {
          sh 'npm install && npm run build'
        }
      }
    }

    if(env.BRANCH_NAME == 'master') {
      stage('NPM Publish') {
        try {
          container('node') {
            withNPM(npmrcConfig: 'npm-internal') {
              sh 'npm publish'
            }
          }
        } catch(err) {
          echo "${err}"
          currentBuild.result = 'SUCCESS'
        }
      }
    }
  }
}
