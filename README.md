jee-ex
=======

A sample app to be deployed on openshift environments

Note: to build this repository with maven you must specify "-Popenshift", eg "mvn clean package -Popenshift"

## Integration with Jenkins

To trigger a build when a push is made to GitHub, do the following:

### For Pipeline type jobs

1.  Create a _Pipeline_ job in Jenkins, with the following config:

    - Tick 'GitHub Project' and set Project URL to: `https://github.com/monodot/jgit-flow-demo` - note that Jenkins will add a trailing slash; this is fine. This URL must not end in `.git`
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

1.  Create a _Multibranch Pipeline_ job in Jenkins, with the following config:

    - Add a Source, choosing _Git_ as the type.
    - Specify the Project Repository as: `https://github.com/monodot/jee-ex-jgitflow.git`

