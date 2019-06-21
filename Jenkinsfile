#!groovy
// -*- coding: utf-8; mode: Groovy; -*-
@Library('common-utils') _

prettify_pipeline(telegram_creds: "lm-devops-swat-bot") {
    node("dockerhost") {
        if ( env.gitlabSourceRepoSshUrl.contains("ansible-docker-") ) {
            repository = (env.gitlabSourceRepoSshUrl =~ /ansible-docker+\-(.+?).git$/)[0][1];
        } else {
            repository = (env.gitlabSourceRepoSshUrl =~ /ansible+\-(.+?).git$/)[0][1];
        }
        try {
            stage('Checkout'){
            deleteDir()
            updateGitlabCommitStatus name: 'Jenkins', state: 'running'
            checkout scm: [$class                           : 'GitSCM',
                           branches                         : [[name: "${env.gitlabSourceBranch}"]],
                           doGenerateSubmoduleConfigurations: false,
                           extensions                       : [[$class: 'PreBuildMerge', options: [fastForwardMode: 'FF', mergeRemote: 
                                                              'origin', mergeStrategy: 'DEFAULT', mergeTarget: "${env.gitlabTargetBranch}"]],
                                                              [$class: 'LocalBranch', localBranch: "${env.gitlabTargetBranch}"]],
                           userRemoteConfigs: [[credentialsId: "id_rsa_jenkins_gitlab", url: "${env.gitlabSourceRepoSshUrl}"]]],
                    changelog: true, poll: true
            updateGitlabCommitStatus name: 'Jenkins', state: 'pending'
        }
            stage('Molecule'){
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                    withDockerContainer(args: '-v /var/run/docker.sock:/var/run/docker.sock -e TERM=xterm -e ANSIBLE_FORCE_COLOR=1', image: 'nexus.lmru.adeo.com:5001/img-molecule-ansible') {
                        sshagent(['id_rsa_jenkins_gitlab']) {
                        sh """mkdir -p molecule/default/roles
                        ln -sf `pwd` molecule/default/roles/${repository}"""
                        sh "molecule test"
                        }
                    }
                }
                updateGitlabCommitStatus name: 'Jenkins', state: 'success'
                addGitLabMRComment comment: "<p>:white_check_mark: Jenkins Build Success</p><p>Results available at: <a href=${env.BUILD_URL}>Jenkins [${env.JOB_NAME} ${BUILD_DISPLAY_NAME}]</a></p>"
            }
        } catch(all) {
            all
            currentBuild.result = 'FAILURE'
            updateGitlabCommitStatus name: 'Jenkins', state: 'failed'
            addGitLabMRComment comment: "<p>:negative_squared_cross_mark: Jenkins Build FAILURE</p><p>Results available at: <a href=${env.BUILD_URL}>Jenkins [${env.JOB_NAME} ${BUILD_DISPLAY_NAME}]</a></p>"
        }
    }
}