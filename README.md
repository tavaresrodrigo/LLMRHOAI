# Uploading and Deploying Models in OpenShiftAI

## Model Download

### Hosting Options
- **External Hosting:** Models can be hosted on platforms like Hugging Face.
- **Internal Hosting:** Models can be stored in an internal repository or Artifactory. If an internal process is in place, no firewall configuration is needed.

### Example: Hugging Face Model Download
In this guide, we will use the model from Hugging Face: [Mistral-7B-v0.3](https://huggingface.co/mistralai/Mistral-7B-v0.3). 

- Navigate to the model's page.
- Click on **Files and Versions** to manually download.
- **Important:** You must agree to share your contact details to access this model.

### Programmatic Download (Recommended)
To automate downloading, use the following Python script:

```python
import os

# Example model repository URL
git_repo = f"https://{os.getenv('HF_USER')}:{os.getenv('HF_TOKEN')}@huggingface.co/mistralai/Mistral-7B-Instruct-v0.3"
model_dir = os.path.basename(git_repo)

if not os.path.exists(model_dir):
    !git clone $git_repo
else:
    %cd $model_dir
    !git fetch --all && git reset --hard origin/main
    %cd ..

print(f"Model Directory: {model_dir}")
```

---

## Upload to S3

### S3 Configuration
Ensure the following environment variables are configured:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_S3_ENDPOINT`
- `AWS_DEFAULT_REGION`
- `AWS_S3_BUCKET`

### Upload Script
Use the following Python script to upload the model to an S3 bucket:

```python
import boto3
import botocore
import os

def upload_directory_to_s3(local_directory, s3_prefix):
    for root, dirs, files in os.walk(local_directory):
        for filename in files:
            file_path = os.path.join(root, filename)
            relative_path = os.path.relpath(file_path, local_directory)
            if ".git" in relative_path:
                print(f"Skipping {relative_path}")
                continue
            s3_key = os.path.join(s3_prefix, relative_path)
            print(f"Uploading {file_path} -> {s3_key}")
            bucket.upload_file(file_path, s3_key)


def list_objects(prefix):
    for obj in bucket.objects.filter(Prefix=prefix).all():
        print(obj.key)

# Initialize S3 session
session = boto3.session.Session(
    aws_access_key_id=os.getenv('AWS_ACCESS_KEY_ID'),
    aws_secret_access_key=os.getenv('AWS_SECRET_ACCESS_KEY')
)

s3_resource = session.resource(
    's3',
    config=botocore.client.Config(signature_version='s3v4'),
    endpoint_url=os.getenv('AWS_S3_ENDPOINT'),
    region_name=os.getenv('AWS_DEFAULT_REGION')
)

bucket = s3_resource.Bucket(os.getenv('AWS_S3_BUCKET'))

# Upload the model
upload_directory_to_s3(model_dir, f"models/{model_dir}")

# Verify uploaded files
list_objects("models")
```

---

Next steps will cover deploying the uploaded model in OpenShiftAI.

