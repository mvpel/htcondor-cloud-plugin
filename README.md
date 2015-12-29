This Jenkins plugin submits batch jobs to the Sun Grid Engine (SGE) batch system.

# Project Status

`sge-cloud-plugin` was forked from [lsf-cloud-plugin](https://github.com/jenkinsci/lsf-cloud-plugin) and modified to work with SGE instead of LSF.  The immediate goal is to prove the feasibility of submitting SGE jobs from Jenkins.  At this point, only the functional changes required to make it function with SGE have been implemented.  As you see below, the GUI labels still say "LSF", not "SGE".  Even the name of the plugin is still `lsf-cloud`.

Nonetheless, it does run with SGE and we are serious about using it in industrial production with our company's Grid Engine compute farm.

In the future I hope to integrate `sge-cloud-plugin` with `lsf-cloud-plugin` to create a single plugin that supports LSF, SGE and perhaps other batch queuing systems such as Condor.
 
# Features

This plugin adds a new type of build step *Run job on LSF* that submits batch jobs to SGE. The build step monitors the job status and periodically (default one minute) appends the progress to the build's *Console Output*. Should the build fail, errors and the exit status of the job also appear. If the job is terminated in Jenkins, it is also terminated in SGE.

Builds are submitted to SGE by a new type of cloud, *LSF Cloud*.  The cloud is given a label like any other slave.  When job with a matching label is run, *LSF Cloud* submits the build to SGE.

Files can be uploaded and sent to SGE before the execution of the job and downloaded from SGE after the job finishes.  	Currently this feature only supports shared file systems.

The job owner can select whether SGE should send an email when the job finishes.

# Installation

This plugin is not an official Jenkins plugin, so you must compile and load it yourself.  After you clone the git repository, build it using Maven:

    cd sge-cloud-plugin/
    mvn install

In *Manage Jenkins > Plugin Manager*, select the *Advanced* tab.  Use *Upload Plugin* to upload the plugin file `sge-cloud-plugin/target/lsf-cloud.hpi`to Jenkins.

# Set Up Jenkins

In SGE, add your Jenkins master host as a submit host.

In *Manage Jenkins > Configure System*, add *Environment Variables*:

* Name `SGE_ROOT` value `/path/to/sge`
* Name `SGE_BIN` value `/path/to/sge/bin/linux-x64`

There are various other [ways to add environment variables](http://stackoverflow.com/questions/5818403/jenkins-hudson-environment-variables/), but the above is one of the most dependable.

In *Manage Jenkins > Configure System*, add a new cloud of type *LSF Cloud*.  Fill in the required information for the newly created cloud.

# Set Up a Project to run on SGE

In a project, specify the *Label* that you specified in *LSF Cloud*.

Add a *Run job on LSF* build step and specify the batch job script you want to run on SGE.

Now, when Jenkins runs the project, it will run on the *LSF Cloud* that has the matching label.

# Job States

## Unfinished Jobs

The [qstat man page](http://gridscheduler.sourceforge.net/htmlman/htmlman1/qstat.html) describes the following job states (job status) defined in SGE.  Each state is a string whose first character is most meaningful:

* "d", for deletion
* "E", for error
* "h", for hold
* "r", for running
* "R", for restarted
* "s", for suspended
* "S", for queue suspended
* "t", for transfering
* "T", for threshold
* "w", for waiting

## Finished Jobs

The above states only describe jobs that have not yet finished, while the Jenkins SGE plugin expects that completed jobs also have a state.  Therefore the SGE plugin derives the state of a finished job from its shell exit status:

* "0" (zero) for a successfully finished job
* "1" through "255" for a job that failed with a nonzero exit status

Exit status above 128 indicates that a signal terminated the job.  See the wiki for an explanation of some exit statuses.

Finally, When The Jenkins SGE plugin could not even submit the job to SGE, the job is given the state:

* "J"

# Environment Variables

Jenkins [adds environment variables to the environment](https://wiki.jenkins-ci.org/display/JENKINS/Building+a+software+project#Buildingasoftwareproject-JenkinsSetEnvironmentVariables), and these are imported into the SGE job environment.  Then [SGE adds some more](http://gridscheduler.sourceforge.net/htmlman/htmlman1/qsub.html).  Since SGE overwrites Jenkins' `JOB_NAME`, it is redefined as `JENKINS_JOB_NAME`.
