# Moqui framework and Apache OFBiz Job (Based upon data model supported in [Job Manager](https://github.com/hotwax/job-manager)) Comparison

## Enitites:

### moqui.service.job.ServiceJob
Primary entity for storing job information.

- jobName (Primary key)
- description
- serviceName
- transactionTimeout
- topic - On completion send a notification to this topic
- localOnly - If Y this will be run local only. By default runs on any server in a cluster listening for async distributed services (if an async distributed executor is configured).
- cronExpression - An extended cron expression like Unix crontab but with extended syntax options (L, W, etc) similar to Quartz Scheduler. See: http://cron-parser.com http://www.quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/tutorial-lesson-06.html
- fromDate - Only run scheduled after this date/time. Ignored for ad-hoc/explicit runs.
- thruDate - Only run scheduled before this date/time. Ignored for ad-hoc/explicit runs.
- repeatCount - If specified only run this many times. Must specify a cronExpression for the job to repeat. When this count is reached thruDate will be set to now and paused set to Y. Ignored for ad-hoc/explicit runs.
- paused - If Y this job is inactive and won't be run on a schedule even if cronExpression is not null. Ignored for ad-hoc/explicit runs.
- expireLockTime - Ignore lock and run anyway after this many minutes. This should generally be much greater than the longest time the service is expected to run. This is the mechanism for recovering jobs after a run failed in a way that did not clean up the ServiceJobRunLock record. Defaults to 24 hours (1440 minutes) to make sure jobs get recovered.
- minRetryTime - Minimum time between retries after an error (based on most recent ServiceJobRun record), in minutes
- priority - Job execution priority, lower numbers run first among jobs that need to be run regardless of scheduled start time


### moqui.service.job.ServiceJobParameter
Stores Parameters for specific job 

- jobName (Primary key)
- parameterName (Primary key)
- parameterValue
- lastUpdatedStamp


### moqui.service.job.ServiceJobRun
- jobRunId (Primary key)
- jobName
- userId - The user that initiated the job run
- parameters
- results
- messages
- hasError
- errors
- hostAddress
- hostName
- runThread
- startTime
- endTime
- lastUpdatedStamp


### moqui.service.job.ServiceJobRunLock

- jobName
- jobRunId - If not null this is the currently running job instance.
- lastRunTime
- lastUpdatedStamp


### moqui.service.job.ServiceJobUser
Associates Job with user to identify sending notifications
- jobName (Primary key)
- userId (Primary key)
- receiveNotifications If Y this user will receive notifications
- lastUpdatedStamp

### Job Sandbox
- jobId
- jobName
- runTime
- priority
- poolId
- statusId
- parentJobId
- previousJobId
- serviceName
- loaderName
- maxRetry
- currentRetryCount
- authUserLoginId
- runAsUser
- runtimeDataId
- tempExprId
- currentRecurrenceCount
- maxRecurrenceCount
- runByInstanceId
- startDateTime
- finishDateTime
- cancelDateTime
- jobResult
- recurrenceTimeZone
- systemJobEnumId
- productStoreId
- shopifyShopId
- createdDate
- createdByUserLogin
- lastModifiedByUserLogin



## Inferences

In OFBiz, we have single entity Job Sandbox for storing jobs which may be draft(custom), pending, running and already executed while in Moqui we have ServiceJob for scheduling the jobs while ServiceJobRun for running & executed jobs information. I think it would be much simpler querying the scheduled jobs and history as being in separate entities.

We could use paused field to identify draft jobs.

In OFBiz, runtime parameters are stored in RunTimeData in xml format while there is entry for each parameter in ServiceJobParameter entity. I think it would be much simpler to handle data now.

There is no run time field, but there is fromDate and thrueDate for handling the schedule for job. We could set the fromDate with date time for job to start and thru date to discontinue. 
For skip operation fromDate could be modified to later date, we could use library like [cron-parser](https://www.npmjs.com/package/cron-parser) for it.
For cancel operation thruDate could be modified to now.


We will need field systemJobEnumId in ServiceJob to identify similar jobs. We could continue to use jobName to get ServiceJobParameter and ServiceJobRun.

We will need fields productStoreId and shopifyShopId in ServiceJob to identify jobs for specific stores.

Instead of temporal expression, [cron expressions](https://www.ibm.com/docs/en/db2oc?topic=task-unix-cron-format) are used in Moqui, we could use libraries like [cron-time-generator](https://www.npmjs.com/package/cron-time-generator) that will help easily generate cron format.

Couldn't find any field related to timezone similar to OFBiz's recurrenceTimeZone.

Couldn't find any field related to timezone similar to OFBiz's recurrenceTimeZone.

moqui.security.UserPreference entity could be used for storing the pinned jobs by user
