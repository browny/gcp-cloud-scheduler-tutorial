# gcp-cloud-scheduler-tutorial

[![Open in Cloud
Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/home/dashboard?cloudshell=true&cloudshell_git_repo=https://github.com/browny/gcp-cloud-scheduler-tutorial&cloudshell_tutorial=README.md)

## Scheduling compute instances with Cloud Scheduler

### 1. Config project

<walkthrough-project-setup></walkthrough-project-setup>


## Set up the Compute Engine instance

```bash
gcloud compute instances create dev-instance \
  --network default \
  --zone us-west1-b \
  --labels=env=dev
```


## Set up the Cloud Functions functions with Pub/Sub

- Create start-instance and stop-instance event

    ```bash
    gcloud pubsub topics create start-instance-event
    ```

    ```bash
    gcloud pubsub topics create stop-instance-event
    ```

- Get code

    ```bash
    git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples.git
    ```

    ```bash
    cd nodejs-docs-samples/functions/scheduleinstance/
    ```

- Create the start and stop functions

    ```bash
	gcloud functions deploy startInstancePubSub \
      --trigger-topic start-instance-event \
      --runtime nodejs8
    ```

    ```bash
	gcloud functions deploy stopInstancePubSub \
      --trigger-topic stop-instance-event \
      --runtime nodejs8
    ```

- (Optional) Verify the functions work

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

- Create the jobs

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

- (Optional) Verify the jobs work

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
