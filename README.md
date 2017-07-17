jee-ex
=======

A sample app to be deployed on openshift environments

Note: to build this repository with maven you must specify "-Popenshift", eg "mvn clean package -Popenshift"

## Integration with Jenkins

To trigger a build when a push is made to GitHub, do the following:

### For Pipeline type jobs

1.  Create a _Pipeline_ job in Jenkins, with the following config:

    - Tick 'GitHub Project' and set Project URL to: `https://github.com/monodot/jee-ex-jgitflow` - note that Jenkins will add a trailing slash; this is fine. This URL must not end in `.git`
    - Tick the box labelled 'GitHub hook trigger for GITScm polling'
    - Configure the job with this pipeline:

            node {
                stage('Hello') {
                    git url: 'https://github.com/monodot/jgit-flow-demo.git', branch: "master"
                }
            }

2.  Edit the job (using Configure) and then Save it again. (This seems to be required by the plugin.)

3.  Now build the job manually. The job must have run successfully at least once in order for the GitHub trigger to be fired.

4.  Now make a commit and push to GitHub. This should trigger the job in Jenkins.

### For Multibranch type jobs

Multibranch jobs in Jenkins allow a job to be created automatically for each branch in the repository. Note that each branch in the remote repository must contain a _Jenkinsfile_ in its root.

1.  Create a _Multibranch Pipeline_ job in Jenkins, with the following config:

    - Add a Source, choosing _Git_ as the type.
    - Specify the Project Repository as: `https://github.com/monodot/jee-ex-jgitflow.git`
    - Add a Pipeline Library:
        - Name: `pipeline-library`
        - Default version: `master`
        - Project repo

2.  In GitHub, go to Settings &rarr; Integrations &amp; services and add the _Jenkins (Git plugin)_ integration.

    - Set the Jenkins URL to (e.g.) `http://yourjenkins:8080` or `http://abc123456.ngrok.io`

3.  Now whenever you push to GitHub, a job will be triggered in Jenkins for the given branch.

#### Testing with local Jenkins

The _Jenkinsfile_ in this repository assumes that Jenkins will be executing builds on **maven** slave nodes (e.g. using OpenShift/Kubernetes). If using a local instance of Jenkins to test, you will need to define a **maven** node, so that the pipeline can run:

1.  Go to Manage Jenkins &rarr; Manage Nodes &rarr; New Node.

2.  Add a new node with the following properties:

    - Name: `maven`
    - Remote root directory: `/path/to/a/writeable/directory`
    - Usage: _Only build jobs with label expressions matching this node_
    - Launch method: _Launch slave agents via SSH_
    - Host: `localhost`
    - Credentials: (add your SSH credentials here)
    - Host Key Verification Strategy: _Non verifying Verification Strategy_


