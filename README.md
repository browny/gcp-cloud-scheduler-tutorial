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
