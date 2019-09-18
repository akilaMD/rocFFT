#!/usr/bin/env groovy
// This shared library is available at https://github.com/ROCmSoftwarePlatform/rocJENKINS/
@Library('rocJenkins') _

// This is file for internal AMD use.
// If you are interested in running your own Jenkins, please raise a github issue for assistance.

import com.amd.project.*
import com.amd.docker.*

////////////////////////////////////////////////////////////////////////
// Mostly generated from snippet generator 'properties; set job properties'
// Time-based triggers added to execute nightly tests, eg '30 2 * * *' means 2:30 AM
properties([
//    pipelineTriggers([cron('0 1 * * *'), [$class: 'PeriodicFolderTrigger', interval: '5m']]),
    buildDiscarder(logRotator(
      artifactDaysToKeepStr: '',
      artifactNumToKeepStr: '',
      daysToKeepStr: '',
      numToKeepStr: '10')),
    disableConcurrentBuilds(),
    // parameters([booleanParam( name: 'push_image_to_docker_hub', defaultValue: false, description: 'Push rocfft image to rocm docker-hub' )]),
    [$class: 'CopyArtifactPermissionProperty', projectNames: '*']
   ])

////////////////////////////////////////////////////////////////////////
// import hudson.FilePath;
import java.nio.file.Path;

rocFFTCI:
{

    def rocfft = new rocProject('rocFFT')
    // customize for project
    rocfft.paths.build_command = './install.sh -c'

    // Define test architectures, optional rocm version argument is available
    def nodes = new dockerNodes(['gfx803 && ubuntu', 'gfx803 && centos', 'gfx803 && sles'], rocfft)

    boolean formatCheck = true

    def compileCommand =
    {
        platform, project->

        project.paths.construct_build_prefix()
        
        def command 
        
        if(platform.jenkinsLabel.contains('hip-clang'))
        {
            command = """#!/usr/bin/env bash
                    set -x
                    cd ${project.paths.project_build_prefix}
                    LD_LIBRARY_PATH=/opt/rocm/lib CXX=/opt/rocm/bin/hipcc ${project.paths.build_command} --hip-clang
                """
        }
        else
        {
            command = """#!/usr/bin/env bash
                    set -x
                    cd ${project.paths.project_build_prefix}
                    LD_LIBRARY_PATH=/opt/rocm/hcc/lib CXX=/opt/rocm/bin/hcc ${project.paths.build_command}
                """
        }
        
        platform.runCommand(this, command)
    }

    def testCommand =
    {
        platform, project->

        def command = """#!/usr/bin/env bash
                    set -x
                    cd ${project.paths.project_build_prefix}/build/release/clients/staging
                """
        
        platform.runCommand(this, command)
        junit "${project.paths.project_build_prefix}/build/release/clients/staging/*.xml"
    }


    def packageCommand =
    {
        platform, project->
       
        if(platform.jenkinsLabel.contains('hip-clang')) packageCommand = null
        else
        {
            def packageHelper

            if(platform.jenkinsLabel.contains('ubuntu')) packageHelper = platform.makePackage(platform.jenkinsLabel,"${project.paths.project_build_prefix}/build/release",true) 
            else if(platform.jenkinsLabel.contains('sles')) packageHelper = platform.makePackage(platform.jenkinsLabel,"${project.paths.project_build_prefix}/build/release",false,true)
            else packageHelper = platform.makePackage(platform.jenkinsLabel,"${project.paths.project_build_prefix}/build/release") 
            
            platform.runCommand(this, packageHelper[0])
            platform.archiveArtifacts(this, packageHelper[1])
        }
    }

    buildProject(rocfft, formatCheck, nodes.dockerArray, compileCommand, testCommand, packageCommand)

}
