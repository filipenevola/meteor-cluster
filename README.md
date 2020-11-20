# meteor-cluster

Meteor Package enabling users to create a Worker Pool on the server to handle heavy jobs.
It can run synchronous and asynchronous tasks.

# TaskQueue
  `TaskQueue` is a Mongodb backed job queue. It enables to add, update, remove jobs consistently between processes.<br><br>
  `TaskQueue.addTask({ taskType: String, data: Object, priority: Integer, _id: String })`
  - `taskType` is mandatory
  - `data` is mandatory but you can pass an empty object
  - `priority` is mandatory, default is set to 1
  - `_id` is optional

# Cluster
  `Cluster` is an isomorphic class to handle both the Worker and the Master<br/><br/>
  on the Master it :
  - verifies if jobs are in the queue
  - verifies if workers are available, or create them
  - dispatches jobs to the workers
  - closes the workers when no jobs are available

  on the Worker it :
  - starts the job
  - removes the task from the queue once the job is done

  ## prototype

  `constructor(taskMap: Object, { port: Integer, maxAvailableWorkers: Integer, refreshRate: Integer })`
  - `maxAvailableWorkers`: maximum number of child process (cores), default set to maximum
  - `port`: server port for child process servers, default set to `3008`
  - `refreshRate`: Worker pool refresh rate (in ms), default set to `1000`
  - `taskMap`: a map of functions associated to a `taskType`

  `Cluster.isMaster()`: `true` if this process is the master<br/>

  `Cluster.maxWorkers()`: returns the maximum number of workers available at the same time<br/>

  `setRefreshRate(refreshRate: Integer)`: change the refresh rate on the master

  if the Master process crashes or restarts, all the unfinished jobs will be restarted from the beginning.<br/>
  Each job is logged when started / finished with the format : `${timestamp}:task:${taskType}:${taskId}`<br/>

# basic usage

```
  import { Meteor } from 'meteor/meteor'
  import { Cluster, TaskQueue } from 'meteor/nschwarz:cluster'

  const taskMap = {
    'SYNC': (job) => console.log("this is a synchrone task"),
    'ASYNC': (job) => new Promise((resolve, reject) => Meteor.setTimeout(() => {
      console.log("this is an asynchrone task")
      resolve()
    }, job.data.timeout))
  }

  const cluster = new Cluster(taskMap)
  Meteor.startup(() => {
    if (Cluster.isMaster()) {
      TaskQueue.addTask({ taskType: 'SYNC', data: {}})
      TaskQueue.addTask({ taskType: 'ASYNC', data: { timeout: 5000 }, priority: 6 })    
    }
  })
```
