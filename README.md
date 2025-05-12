## cloud-run-job-shell






<!-- TOC -->

- [cloud-run-job-shell](#cloud-run-job-shell)
    - [參考資料](#%E5%8F%83%E8%80%83%E8%B3%87%E6%96%99)
    - [setting GCP project & job runtime env](#setting-gcp-project--job-runtime-env)
    - [build docker image](#build-docker-image)
    - [登入 GCP container Registry](#%E7%99%BB%E5%85%A5-gcp-container-registry)
    - [push GCP container Registry](#push-gcp-container-registry)
    - [建立 job](#%E5%BB%BA%E7%AB%8B-job)
    - [更新 job](#%E6%9B%B4%E6%96%B0-job)
    - [執行 job](#%E5%9F%B7%E8%A1%8C-job)
    - [執行 job 並等待執行完成](#%E5%9F%B7%E8%A1%8C-job-%E4%B8%A6%E7%AD%89%E5%BE%85%E5%9F%B7%E8%A1%8C%E5%AE%8C%E6%88%90)

<!-- /TOC -->

---

### 參考資料
> - ref: https://cloud.google.com/run/docs/create-jobs
> - ref: https://cloud.google.com/run/docs/quickstarts#build-and-create-a-job



### setting GCP project & job runtime env
```
JOB_NAME="job-quickstart"
GCP_REGION="asia-east1"
GCP_PROJECT_ID="${GCP_PROJECT_ID}"
GCP_CONTAINER_REPO="${GCP_CONTAINER_REPO}"
IMAGE_URL="${GCP_REGION}-docker.pkg.dev/${GCP_PROJECT_ID}/${GCP_CONTAINER_REPO}/${JOB_NAME}"

NETWORK="${VPC_NETWORK}"
SUBNET="${VPC_SUBNET}"
SERVICE_ACCOUNT="${SERVICE_ACCOUNT_NAME}@${GCP_PROJECT_ID}.iam.gserviceaccount.com"
GCP_KEY_FILE="${GCP_KEY_FILE_PATH}"
```

### build docker image
```
docker build --platform linux/amd64 -f Dockerfile -t ${JOB_NAME} .
```

### 登入 GCP container Registry
```
cat "${GCP_KEY_FILE}" | docker login -u _json_key --password-stdin https://${GCP_REGION}-docker.pkg.dev
```

### push GCP container Registry
```
docker tag ${JOB_NAME} ${IMAGE_URL}:latest
docker push ${IMAGE_URL}:latest
```

### 建立 job
```
gcloud run jobs create ${JOB_NAME} \
    --image=${IMAGE_URL} \
    --project=${GCP_PROJECT_ID} \
    --region=${GCP_REGION} \
    --network=${NETWORK} \
    --subnet=${SUBNET} \
    --cpu=1 \
    --memory=512Mi \
    --tasks=1 \
    --max-retries=3 \
    --task-timeout=600 \
    --service-account=${SERVICE_ACCOUNT} \
    --set-env-vars SLEEP_MS=10000 \
    --set-env-vars FAIL_RATE=0.5



```

### 更新 job

```
gcloud run jobs deploy ${JOB_NAME} \
    --image=${IMAGE_URL} \
    --project=${GCP_PROJECT_ID} \
    --region=${GCP_REGION} \
    --network=${NETWORK} \
    --subnet=${SUBNET} \
    --cpu=1 \
    --memory=512Mi \
    --tasks=3 \
    --max-retries=3 \
    --task-timeout=600 \
    --service-account=${SERVICE_ACCOUNT} \
    --set-env-vars SLEEP_MS=10000 \
    --set-env-vars FAIL_RATE=0.3
```


### 執行 job
```
gcloud run jobs execute ${JOB_NAME} --region=${GCP_REGION}
```

### 執行 job 並等待執行完成
```
gcloud run jobs execute ${JOB_NAME} --region=${GCP_REGION} --wait
```





### Environment variables for jobs
For Cloud Run jobs, the following environment variables are set:


 **Name**               | **Description**                                                                                                      | **Example**     
------------------------|----------------------------------------------------------------------------------------------------------------------|-----------------
 CLOUD_RUN_JOB          | The name of the Cloud Run job being run.                                                                             | hello-world     
 CLOUD_RUN_EXECUTION    | The name of the Cloud Run execution being run.                                                                       | hello-world-abc 
 CLOUD_RUN_TASK_INDEX   | The index of this task. Starts at 0 for the first task and increments by 1 for every successive task, up to the maximum number of tasks minus 1. If you set --parallelism to greater than 1, tasks might not follow the index order. For example, it would be possible for task 2 to start before task 1. | 0               
 CLOUD_RUN_TASK_ATTEMPT | The number of times this task has been retried. Starts at 0 for the first attempt and increments by 1 for every successive retry, up to the maximum retries value.                                                                                                                                        | 0               
 CLOUD_RUN_TASK_COUNT   | The number of tasks defined in the --tasks parameter.                                                                | 1               








