---
layout: default
title: Running Your First CHTC Jobs
redirect_from:
   - /helloworld.shtml
---

<p>
So, you have an account on a submit node,
and you are ready to run your first job in the CHTC. 
As we described in <a href="/approach.shtml">Our Approach</a>, 
the CHTC is a collection of distributed resources. 
The magic that enables you to run jobs on these resources is software, 
called <a href="http://research.cs.wisc.edu/htcondor/">HTCondor</a>,
developed at the UW-Madison.
</p>	
	
<h1>Contents</h1>
1.  <a href="#first">Let's first do, and then ask why</a>
2.  <a href="#else">What Else?</a>
    <ul>
    <li><a href="#remove">Removing Jobs</a></li>	
    <li><a href="#importance">The Importance of Testing</a></li>
    <li><a href="#resource">Getting the Right Resources</a></li>	
    <li><a href="#homework">Time for a little homework</a></li>
    </ul>

<a name="first"></a>
<h1>1. Let's first do, and then ask why</h1>
<p>
Rather than having you read a bunch of stuff before hand, 
let's just run some jobs so you can see what happens,
 and we'll provide some additional discussion along the way. 
</br></br>
We are going to run the traditional 'hello world' program with a CHTC twist.
In order to demonstrate the distributed resource nature of the CHTC, 
we will produce a 'Hello CHTC' message 3 times, where each time is its own job.
 Since you are not directly invoking the execution of each job, 
you need to tell HTCondor <i>how</i> to run the jobs for you. 
The information needed is placed into a <i>submit file</i>, which
 defines variables that describe the set of jobs.
</p>
<p>
<strong><i>Note: You must be logged into an HTCondor submit machine 
for the following example to work</i></strong>
<p>
<b>1.</b> Copy the highlighted text below,
and paste it into file called <code>hello-chtc.sub</code>, the submit file,
 in your home directory on the submit machine.
</p>
<pre class="sub">
# hello-chtc.sub
# My very first HTCondor submit file
#
# Specify the HTCondor Universe (vanilla is the default and is used
#  for almost all jobs), the desired name of the HTCondor log file,
#  and the desired name of the standard error file.  
#  Wherever you see $(Cluster), HTCondor will insert the queue number
#  assigned to this set of jobs at the time of submission.
universe = vanilla
log = <i>hello-chtc_$(Cluster)</i>.log
error = <i>hello-chtc_$(Cluster)_$(Process)</i>.err
#
# Specify your executable (single binary or a script that runs several
#  commands), arguments, and a files for HTCondor to store standard
#  output (or "screen output").
#  $(Process) will be a integer number for each job, starting with "0"
#  and increasing for the relevant number of jobs.
executable = <i>hello-chtc</i>.sh
arguments = <i>$(Process)</i>
output = <i>hello-chtc_$(Cluster)_$(Process)</i>.out
#
# Specify that HTCondor should transfer files to and from the
#  computer where each job runs. The last of these lines *would* be
#  used if there were any other files needed for the executable to run.
should_transfer_files = <i>YES</i>
when_to_transfer_output = <i>ON_EXIT</i>
# transfer_input_files = file1,/absolute/pathto/file2,etc
#
# Tell HTCondor what amount of compute resources
#  each job will need on the computer where it runs.
request_cpus = <i>1</i>
request_memory = <i>1GB</i>
request_disk = <i>1MB</i>
#
# Tell HTCondor to run 3 instances of our job:
queue <i>3</i>
</pre>

<blockquote>For a "template" version of this submit file without 
the comments, <a href="/files/template.sub">click here</a>.</blockquote>

<P>
<b>2.</b> Now, create the executable that we specified above: copy the text
 below and paste it into a file called <code>hello-chtc.sh</code>
<P>
<pre class="file">
#!/bin/bash
#
# hello-chtc.sh
# My very first CHTC job
#
# print a 'hello' message to the job's terminal output:
echo "Hello CHTC from Job $1 running on `whoami`@`hostname`"
#
# keep this job running for a few minutes so you'll see it in the queue:
sleep 180
</pre>
When HTCondor runs this executable,
 it will pass the $(Process) value for each job and <code>hello-chtc.sh</code>
 will insert that value for "$1", above.
<P>
<b>3.</b> Now, submit your job to the queue using <code>condor_submit</code>:
<P>
<pre class="term">
[alice@submit]$ condor_submit <i>hello-chtc</i>.sub
</pre>
<P>
The <code>condor_submit</code> command
actually submits your jobs to HTCondor. 
If all goes well, you will see output from
the <code>condor_submit</code> command that appears as:
<P>
<pre class="term">
Submitting job(s).....
3 job(s) submitted to cluster 436950.
</pre>
<P>
<b>4.</b> To check on the status of your jobs, run the following command:
<P>
<pre class="term">
[alice@submit]$ condor_q
</pre>
<P>
 (If you want to see everyone's jobs, use <code>condor_q -all</code>.)
<P>
The output of <code>condor_q</code> should look like this: </p>
<pre class="term">
-- Schedd: submit-2.chtc.wisc.edu : <128.104.101.92:9618?... @ 04/05/19 15:35:17
OWNER  BATCH_NAME     SUBMITTED   DONE   RUN    IDLE  TOTAL  JOB_IDS
alice  ID: 436950     4/5  15:34     _     _       3      3  436950.0-2

3 jobs; 0 completed, 0 removed, 3 idle, 0 running, 0 held, 0 suspended
</pre>
<P>
You can run the <code>condor_q</code> command periodically to see the progress of your jobs.
By default, <code>condor_q</code> shows jobs grouped into batches by batch name (if provided), 
or executable name.  To show all of your jobs on individual lines, add the <code>-nobatch</code>
option.  For more details on this option, and other options to <code>condor_q</code>, see our 
<a href="/condor_q.shtml">condor_q guide</a>.

<blockquote>
<b>Potential Failures</b><br>
<p>If your jobs go on hold and you usually use a Windows laptop 
or desktop, please see <a href="/dos-unix.shtml">
this page</a> for a potential diagnosis and solution.  
</blockquote>

</br></br>
<b>5.</b> When your jobs complete after a few minutes, they'll leave the queue.
 If you do a listing of your home directory with the command <code>ls -l</code>,
 you should see something like:
<P>
<pre class="term">[alice@submit]$ ls -l
total 28
-rw-r--r-- 1 alice alice    0 Apr  5 15:37 hello-chtc_436950_0.err
-rw-r--r-- 1 alice alice   60 Apr  5 15:37 hello-chtc_436950_0.out
-rw-r--r-- 1 alice alice    0 Apr  5 15:37 hello-chtc_436950_1.err
-rw-r--r-- 1 alice alice   60 Apr  5 15:37 hello-chtc_436950_1.out
-rw-r--r-- 1 alice alice    0 Apr  5 15:37 hello-chtc_436950_2.err
-rw-r--r-- 1 alice alice   60 Apr  5 15:37 hello-chtc_436950_2.out
-rw-r--r-- 1 alice alice 5111 Apr  5 15:37 hello-chtc_436950.log
-rw-rw-r-- 1 alice alice  241 Apr  5 15:33 hello-chtc.sh
-rw-rw-r-- 1 alice alice 1387 Apr  5 15:33 hello-chtc.sub
</pre>

<P>
<b>Useful information is provided in the user log and the output files.</b>
<P>
HTCondor creates a transaction log of everything that happens to your jobs.
Looking at the log file is very useful for
debugging problems that may arise.
An excerpt from 
<code>hello-chtc_845638.log</code>
produced due the submission of the 3 jobs will look something like this:
</P>
<pre class="file">

000 (436950.000.000) 04/05 15:34:33 Job submitted from host: <128.104.101.92:9618?addrs=128.104...>
...
040 (436950.000.000) 04/05 15:34:50 Started transferring input files
	Transferring to host: <128.104.55.170:9618?addrs=128.104....>
...
040 (436950.000.000) 04/05 15:34:50 Finished transferring input files
...
001 (436950.000.000) 04/05 15:34:51 Job executing on host: <128.104.55.170:9618?addrs=128.104...>
...
006 (436950.000.000) 04/05 15:35:00 Image size of job updated: 368
	1  -  MemoryUsage of job (MB)
	292  -  ResidentSetSize of job (KB)
...
040 (436950.000.000) 04/05 15:37:51 Started transferring output files
...
040 (436950.000.000) 04/05 15:37:51 Finished transferring output files
...
005 (436950.000.000) 04/05 15:37:51 Job terminated.
	(1) Normal termination (return value 0)
		Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
		Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
	60  -  Run Bytes Sent By Job
	241  -  Run Bytes Received By Job
	60  -  Total Bytes Sent By Job
	241  -  Total Bytes Received By Job
	Partitionable Resources :    Usage  Request Allocated 
	   Cpus                 :        0        1         1 
	   Disk (KB)            :       24     1024    908236 
	   Ioheavy              :                           0 
	   Memory (MB)          :        1     1024      1024 
...

</pre>
And, if you look at one of the output files,
you should see something like this:
</P>
<pre class="file">
Hello CHTC from Job 0 running on alice@e389.chtc.wisc.edu
</pre>
<strong>Congratulations.</strong> You've run your first jobs in the CHTC!
<br></br>

<!-- <blockquote>

<p>You are likely seeing a Windows/Linux incompatibility error. There's 
a difference between the Windows/DOS line-ending character and 
the Linux/UNIX line-ending character, and bash will not be able to run 
the <code>hello-chtc.sh</code> script if it has the DOS line endings.  </p>

<p>You can see the line endings if you open the file in vi's "binary" mode:
 <pre class="term">[alice@submit]$ vi -b hello-chtc.sh</pre>
(type ":q" to quit vi)  The "^M" characters are the dos line endings.  </p>

<p>Luckily, there is an easy fix!  To convert the script to 
unix line endings, you can run: 
<pre class="term">[alice@submit]$ dos2unix hello-chtc.sh</pre>
on the submit node and it will change the format for you.  If you 
re-submit the jobs, they should now run successfully.</blockquote> -->

<a name="else"></a>
<h1>2. What Else?</h1>
<a name="remove"></a>
<h2>A. Removing Jobs</h2>
<p>
To remove a specific job, specify the job ID nubmer from the queue (format:
 Cluster.Process). Example:
<pre class="term">
[alice@submit]$ condor_rm <i>845638.0</i>
</pre>
You can even remove all of the jobs of the same cluster by specifying
 only the Cluster value for that batch.</br>
</br>
To remove <b>all of your jobs</b>:
<pre class="term">
[alice@submit]$ condor_rm <i>$USER</i>
</pre>
</p>
	
<a name="importance"></a>
<h2>B. The Importance of Testing</h2>
<P>
1. <b>Examining Job Success.</b> Within the log file, you can see
 information about the completion of each job, including a system
 error code (as seen in "return value 0"). You can use this code, as well
 as information in your ".err" file and other output files, to determine what
 issues your job(s) may have had, if any.</br>
</br>
2. <b>Determining Memory and Disk Requirements.</b> The log file also
 indicates how much memory and disk each job used, so that you
 can first test a few jobs before submitting many more with more accurate
 request values. When you request too little, your jobs will be "evicted"
 from the computer they're running on, and HTCondor will have to try to
 rerun them (maybe many times) until it requests enough for you.
 When you request too much, your jobs may not match to as many available
 "slots" as they could otherwise, and your overall throughput will suffer
 in that case as well.</br>
</br>
3. <b>Determining Run Time.</b> Depending on how long each of your jobs
 are (determined by examining when the job began executing and when it
 completed), you can send your jobs to even more computers than are in the
 CHTC Pool (where your jobs will run, by default). Refer to the table below
 for tips on how to send your jobs to the rest of the UW Grid and to the
 national <a href="http://www.opensciencegrid.org/">Open Science Grid</a>. 
</P>
	<a name="resource"></a>
<h2>C. Getting the Right Resources</h2>
<P>
Be sure to always add or modify the following lines in your submit files, as appropriate,
 and after running a few tests.
<table class="gtable">
<tr><th>Submit file entry</th><th>Resources your jobs will run on</th></tr>
<tr>
<td>request_cpus = <i>cpus</i></td>
<td>
Matches each job to a computer "slot" with at least this many CPU cores.
</td>
</tr>
<tr>
<td>request_disk = <i>kilobytes</i></td>
<td>
Matches each job to a slot with at least this much disk space, in units of KB.<br>
</td>
</tr>
<tr>
<td>request_memory = <i>megabytes</i></td>
<td>
Matches each job to a slot with at least this much memory (RAM), in units of MB.<br>
</td>
</tr>
<tr>
<td>+WantFlocking = true</td>
<td>
Also send jobs to other HTCondor Pools on campus (UW Grid)<br>
Good for jobs that are less than ~8 hours, or checkpoint at least that frequently.
</td>
</tr>
<tr>
<td>+WantGlideIn = true</td>
<td>
Also send jobs to the Open Science Grid (OSG).<br>
Good for jobs that are less than ~8 hours (or checkpoint at least that frequently), 
and have been tested for portability. (Contact Us for more details).
</td>
</tr>
</table>

<p>Learn more about sending jobs to the UW Grid and OSG in our 
<a href="/scaling-htc.shtml">Scaling Beyond Local HTC Capacity</a> guide.</p>

	<a name="homework"></a>
<h2>D. Now, time for a little homework</h2>
<P>
To get the most of the CHTC,
you will want to have a good understanding of how HTCondor works.
<b>We HIGHLY recommend browsing the latest 
<a href="https://agenda.hep.wisc.edu/event/1325/other-view?view=standard#20180521.detailed">HTCondor User Tutorial</a>
 from the international HTCondor Week conference.</b>
</br></br>
<a href="guides.shtml">Our full set of CHTC online guides is available here.</a> Remember to <a href="get-help.shtml">Get Help</a> whenever you have questions or issues. That's what CHTC staff are here for.
</br></br>
The full HTCondor manual is comprehensive and lengthy, and Googling "HTCondor examples"
may lead you to examples that really only work on another campus's HTCondor system. You 
can always dig into more details as you become more experienced, but the below pages 
of the manual may be a good place to start if you like manuals:
<P>
<ul>
<li><a href="https://htcondor.readthedocs.io/en/latest/users-manual/running-a-job-steps.html">Road-map for Running Jobs</a>
</li>
<li><a href="https://htcondor.readthedocs.io/en/latest/users-manual/submitting-a-job.html">Submitting a Job</a>
</li>
<li><a href="https://htcondor.readthedocs.io/en/latest/man-pages/condor_submit.html"><code>condor_submit</code> manual page</a>
</li>
<li><a href="https://htcondor.readthedocs.io/en/latest/users-manual/managing-a-job.html">Managing a Job</a>
</li>
<li><a href="https://htcondor.readthedocs.io/en/latest/man-pages/condor_q.html"><code>condor_q</code> manual page</a>
</li>
<li><a href="https://htcondor.readthedocs.io/en/latest/users-manual/matchmaking-with-classads.html">Matchmaking with ClassAds</a>
</li>
</ul>
</P>
<h1>Now you are ready for some real work</h1>
<p> 
Ok, you have the basics! 
This should be enough background to get you started using the CHTC for the
<i>real</i> problems you came to us for. 
Remember, we are here to help. 
Don't hesitate to contact us at
<a href="chtc@cs.wisc.edu">chtc@cs.wisc.edu</a> with questions.
</p>