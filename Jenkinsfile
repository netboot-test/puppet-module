pipeline {
  agent { label 'LINUX' }
  stages {
    stage('init') {
      steps {
        deleteDir()
        checkout scm
        script {
          properties([pipelineTriggers([cron('@daily'), [$class: 'PeriodicFolderTrigger', interval: '1d']]), [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10']]])
          path = '/opt/puppetlabs/puppet/bin:/usr/local/bin:/usr/bin'
        }
      }
    }
    stage('bundle') {
      steps {
        withEnv(["PATH=$path"]) {
        sh '''#!/bin/bash
          set +e
          printenv
          bundle install --path=~/.gem/ruby/2.1.0/gems --binstubs=~/.gem/ruby/2.1.0/bin
        '''
        }
      }
    }
    stage('pp_validate') {
      steps {
        withEnv(["PATH=$path"]) {
        sh '''#!/bin/bash
          set +e
          [ -d manifests/ ] && puppet parser validate --color false --render-as s manifests/
        '''
        }
      }
    }
    stage('rake_syntax') {
      steps {
        withEnv(["PATH=$path"]) {
        sh '''#!/bin/bash
          set +e
          bundle exec rake syntax
        '''
        }
      }
    }
    stage('pp_lint') {
      steps {
        withEnv(["PATH=$path"]) {
          sh '''#!/bin/bash
            set +e
            bundle exec puppet-lint --no-autoloader_layout-check --log-format "%{path}:%{line}:%{check}:%{KIND}:%{message}" --no-80chars-check manifests/*.pp
          '''
        }
      }
    }
    stage('rspec_lint') {
      steps {
        withEnv(["PATH=$path"]) {
          sh '''#!/bin/bash
            set +e
            bundle exec rubocop spec/classes/
          '''
        }
      }
    }
    stage('pp_rspec') {
      steps {
        withEnv(["PATH=$path"]) {
          sh '''#!/bin/bash
            set +e
            bundle exec rake spec_clean
            bundle exec rake spec
          '''
        }
      }
    }
  }
}
