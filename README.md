# gcp-cloud-scheduler-tutorial

[![Open in Cloud
Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/home/dashboard?cloudshell=true&cloudshell_git_repo=https://github.com/browny/gcp-cloud-scheduler-tutorial&cloudshell_tutorial=README.md)

## Scheduling compute instances with Cloud Scheduler

This tutorial demos how to use Cloud Scheduler to schedule Compute Engine instances to start and
stop at specified or periodical time.

The main componentes used in this turial includes Compute Engine instances (with labels), Cloud
Pub/Sub, Cloud Functions and Cloud Scheduler. The event flow looks like as below: 

    Cloud Scheduler -> Cloud Pub/Sub -> Cloud Functions -> Start/Stop Compute Engine instances


### 1. Config project

Select the project which you have permissions to access Compute Engine, Cloud Pub/Sub, Cloud
Functions and Cloud Scheduler. Remember to enable these APIs first.

<walkthrough-project-setup></walkthrough-project-setup>


## Set up the Compute Engine instance

Create a sample instance with label `env=dev` which will be used as filter when start/stop action
executed by Cloud Functions.

```bash
gcloud compute instances create dev-instance \
  --network default \
  --zone us-west1-b \
  --labels=env=dev
```


## Set up the Cloud Functions functions with Pub/Sub

Create 2 Cloud Pub/Sub topics which will be used as the triggers of following Cloud Functions.

### 1. Create start-instance and stop-instance event

```bash
gcloud pubsub topics create start-instance-event
```

```bash
gcloud pubsub topics create stop-instance-event
```

Create 2 Cloud Funtions, one for starting instances, the other for stopping instances. They are
triggered by the message of above 2 Cloud Pub/Sub topics.

### 2. Get code

```bash
git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples.git
```

```bash
cd nodejs-docs-samples/functions/scheduleinstance/
```

### 3. Create the start and stop functions

```bash
gcloud functions deploy startInstancePubSub \
  --trigger-topic start-instance-event \
  --runtime nodejs8 \
  --allow-unauthenticated
```

```bash
gcloud functions deploy stopInstancePubSub \
  --trigger-topic stop-instance-event \
  --runtime nodejs8 \
  --allow-unauthenticated
```

Once the functions created, we can manually trigger it to verify the instance being stopped or
started.

### 4. (Optional) Verify the functions work

```bash
gcloud functions call stopInstancePubSub \
--data '{"data":"eyJ6b25lIjoidXMtd2VzdDEtYiIsICJsYWJlbCI6ImVudj1kZXYifQo="}'
```

```bash
gcloud compute instances describe dev-instance \
--zone us-west1-b \
| grep status
```


## Set up the Cloud Scheduler jobs to call Pub/Sub

Now we have 2 Cloud Functions with corresponding Cloud Pub/Sub topics as their triggers ready. Then
we will create 2 Cloud Scheduler jobs, one for starting the instance at 9AM every weekday (Mon. to
Fri.), the other for stopping the instance at 10AM every weekday. 

### 1. Create the jobs (notice: change below `--schedule` parameter to `0 9 * * 1-5` / `0 10 * * 1-5`)

```bash
gcloud beta scheduler jobs create pubsub startup-dev-instances \
  --schedule '0 9 * * 1-5' \
  --topic start-instance-event \
  --message-body '{"zone":"us-west1-b", "label":"env=dev"}' \
  --time-zone 'Asia/Taipei'
```

```bash
gcloud beta scheduler jobs create pubsub shutdown-dev-instances \
  --schedule '0 10 * * 1-5' \
  --topic stop-instance-event \
  --message-body '{"zone":"us-west1-b", "label":"env=dev"}' \
  --time-zone 'Asia/Taipei'
```

Once the jobs created, we don't need to wait until the scheduled time, we can manaully run the jobs
to verify the following actions being ran accordingly.

### 2. (Optional) Verify the jobs work

```bash
gcloud beta scheduler jobs run shutdown-dev-instances
```

```bash
gcloud compute instances describe dev-instance \
--zone us-west1-b \
| grep status
```

```bash
gcloud beta scheduler jobs run startup-dev-instances
```

```bash
gcloud compute instances describe dev-instance \
--zone us-west1-b \
| grep status
```
	
## Clean up

- Delete the Cloud Scheduler jobs
- Delete the Pub/Sub topics
- Delete the Cloud Functions functions
- Delete the Compute Engine instance
