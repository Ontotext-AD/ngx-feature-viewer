pipeline {

  parameters {
    gitParameter name: 'branch',
                description: 'The branch to check out',
                branchFilter: 'origin/(.*)',
                defaultValue: 'master',
                selectedValue: 'DEFAULT',
                type: 'PT_BRANCH',
                listSize: '0',
                quickFilterEnabled: true

    string name: 'ReleaseVersion',
         description: 'Version to release',
         defaultValue: ''
  }

  agent {
    label 'graphdb-jenkins-node'
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 15, unit: 'MINUTES')
    timestamps()
  }

  environment {
    CI = "true"
    NPM_TOKEN = credentials('npm-token')
  }

  stages {
    stage ('Prepare') {
      steps {
        // Change versions
        sh "(cd projects/ngx-feature-viewer && npm version --git-tag-version=false ${ReleaseVersion})"

        // Install
        sh "npm run ci"

        // Build
        sh "npm run build:prod"
      }
    }

    stage ('Publish') {
      steps {
        sh "npm set //registry.npmjs.org/:_authToken ${NPM_TOKEN}"
        // Publish on npm
        sh "(cd dist/ngx-feature-viewer/ && npm publish)"
      }
    }
  }

  post {
    success {
      // Commit, tag and push the changes in Git
      sh "git commit -a -m 'Release ${ReleaseVersion}'"
      sh "git tag -a v${ReleaseVersion} -m 'Release v${ReleaseVersion}'"
      sh "git push --set-upstream --force origin ${branch} && git push --tags"
    }

    failure {
      wrap([$class: 'BuildUser']) {
        emailext(
          to: env.BUILD_USER_EMAIL,
          from: "Jenkins <hudson@ontotext.com>",
          subject: '''[Jenkins] $PROJECT_NAME - Build #$BUILD_NUMBER - $BUILD_STATUS!''',
          mimeType: 'text/html',
          body: '''${SCRIPT, template="groovy-html.template"}'''
        )
      }
    }
  }
}
