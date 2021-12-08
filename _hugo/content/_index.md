+++
title = "Google cloud functions"
outputs = ["Reveal"]
+++

# Google cloud functions
- Demo repo here: https://github.com/joshcarp/cloudfunctionsdemo
---
# Google cloud functions
- Serverless
- Docker-less

---
# Supported languages
- Node.js 8 & Node.js 10
- Python
- Go 1.11 & Go 1.13

---
# Code structure
```
.
├── function.go --> function with http.HandlerFunc
├── go.mod
└── shared/
    └── shared.go
```
---
# http.Handlerfunc

```go
type HandlerFunc func(http.ResponseWriter, *http.Request)
```

example:

```go
func ServeHTTP(w http.ResponseWriter, r *http.Request) {
  w.Write([]byte("Hello, world"))
}
```

---
# Deploying from github actions
Need: github service account with `Cloud Functions Admin` permissions

- go to [cloud console iam admin service accounts](https://console.cloud.google.com/iam-admin/serviceaccounts?project=gopper)
- click `Create Service account` 
- give service account `Cloud Functions Admin` permissions

---
# Deploying from github actions
- download access key
- Put access key in `GCLOUD_SERVICE_KEY` secret in repo

---
# Github actions
```yaml
name: cloud-function-deploy

on:
  push:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Setup - gcloud / gsutil
        uses: GoogleCloudPlatform/github-actions/setup-gcloud@master
        with:
          service_account_key: ${{ secrets.GCLOUD_SERVICE_KEY }}
          export_default_credentials: true
      - name: Set default project
        run: gcloud config set core/project <PROJECT NAME>
      - name: Deploy
        run: |
          gcloud functions deploy <FUNCTION NAME> \
                 --entry-point=ServeHTTP \
                 --runtime=go113 \
                 --trigger-http \
                 --allow-unauthenticated
      - name: Set permissions
        run: |-
          gcloud functions add-iam-policy-binding <FUNCTION NAME>\
            --member="allUsers" \
            --role="roles/cloudfunctions.invoker"
```
more options [here](https://cloud.google.com/sdk/gcloud/reference/functions/deploy)

---
# Push to master

- Should be deployed at `<region>-<projectid>cloudfunctions.net/<function name>`

```bash
curl <region>-<projectid>cloudfunctions.net/<function name>
> Hello, World!
```

---
# Create a gcs bucket: via cloud console
- go to https://console.cloud.google.com/storage/browser
- click `CREATE BUCKET`


---
# Authenticating locally
- `export GOOGLE_APPLICATION_CREDENTIALS=<path to json key>`
- Google cloud service account with:
 - `Storage Admin` permissions if creating buckets programmatically
 - `Storage Object Admin` for only read/write access to one bucket (can't create new buckets)

---
# Uploading files

```go
func upload(bucket string, object string, r io.Reader) error {
	ctx := context.Background()
	client, err := storage.NewClient(ctx)
	if err != nil {
		return fmt.Errorf("storage.NewClient: %v", err)
	}
	defer client.Close()

	wc := client.Bucket(bucket).Object(object).NewWriter(ctx)
	if _, err = io.Copy(wc, r); err != nil {
		return fmt.Errorf("io.Copy: %v", err)
	}
	if err := wc.Close(); err != nil {
		return fmt.Errorf("Writer.Close: %v", err)
	}
	return nil
}
```

---
# Downloading files
```go
func download(bucket, object string) (io.Reader, error) {
	ctx := context.Background()
	client, err := storage.NewClient(ctx)
	if err != nil {
		return nil, fmt.Errorf("storage.NewClient: %v", err)
	}
	defer client.Close()
	return client.Bucket(bucket).Object(object).NewReader(ctx)
}
```
---
# Bonus create a gcs bucket in go
```go
func NewBucket(bucketName string)error{
	ctx := context.Background()
	client, err := storage.NewClient(ctx)
	if err != nil {
		return err
	}

	bucket := client.Bucket(bucketName)
	ctx, cancel := context.WithTimeout(ctx, time.Second*10)
	defer cancel()
	if err := bucket.Create(ctx, projectID, nil); err != nil {
		return err
	}
	return nil
}
```

# Authenticating locally vs Cloud functions
- Locally the service account needs `Storage Admin`/ `Storage Object Admin`
- On google cloud functions all functions already have access to the bucket

# Demo repo
-  https://github.com/joshcarp/cloudfunctionsdemo