// -*- mode: groovy -*-

// Only keep some recent builds and artefacts.
properties([[$class: 'jenkins.model.BuildDiscarderProperty',
             strategy: [$class: 'LogRotator',
                        numToKeepStr: '10',
                        artifactNumToKeepStr: '5']]])

// Build starts here. We put this in a try/except/finally block so that we can
// perform post-build actions (such as Slack notifications).
caughtError = 0
try {

    // Run tests and builds on separate nodes (potentially).
    stage "Test"
    node {

        // Checkout the source of this repository. The credentials and URL are
        // stored in the configuration of the multibranch pipeline job.
        checkout scm

        timeout(time: 5, unit: "MINUTES") {
            sh("py.test complicated_library_test.py")
        }
    }

    // Execute multiple builds in parallel. Each of these targets runs on a
    // different executor. Built objects will be archived successfully, even if
    // other parallel targets fail (because failFast is False).
    //
    // Unfortunately, these objects will not be available as "last successful"
    // artefacts. It is also not possible to build certain targets
    // individually, meaning a failed job will need to be re-run in its
    // entirety to build one artefact that previously failed.
    stage "Build and archive binaries"
    parallel makeFirstTarget: {
        node {
            sh("make")
            archive([includes: "artefacts/complicated_binary_first.deb"])
        }
    }

    // If we got here, it means the build was a success.
    currentBuild.result = "SUCCESS"
}

// Postbuild stuff
catch (caughtError) {
    currentBuild.result = "FAILURE"
    caughtError = caughtError
}

finally {

    // Run this finally block in a node to define readFile.
    node {
        // Add Slack notification. We don't run this on a separate executor
        // since it's a small job. The token is stored on the Jenkins server
        // (in private).

    }
}