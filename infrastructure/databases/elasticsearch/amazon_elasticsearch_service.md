# Elasticsearch

## Amazon Elasticsearch Service
General information for Amazon's Elasticsearch Service can be found within the [official documentation](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html)

For general help to upgrade Amazon Elasticsearch click [here](https://docs.aws.amazon.com/de_de/elasticsearch-service/latest/developerguide/es-version-migration.html) 


The following steps are required to __Upgrade__ from Elastic 6.x to 7.x:

### Prerequisites for an upgrade

- Important: a major upgrade from 6.x to 7.x will delete everything and the cluster will be rebuild
- Installed AWS SDK for console
- Configured SDK via ~/.aws/config
- Authenticated user (in config) need access to elasticsearch service
- All requests (GET, PUT, POST) has to be signed by AWS. Therefore it is recommended to use a Python script with the necessary tools like requests, boto 3 and AWS4Auth.
    ```bash
    pip install requests
    pip install requests-aws4auth
    pip install boto3
    ```

### Backup existing Elastic (create snapshot)

All steps are described in detail in the [official AWS documentation](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains-snapshots.html) and should be considered.


The Variable `host = '{ELASTICSEARCH_URL}/'` in the python scripts below has to be set on the **OLD** elasticsearch server, because we want to backup it.

##### 1. Create S3 Bucket to store snapshots
See [Creating a bucket](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html#create-bucket-intro) in the official AWS documentation

##### 2. Create IAM-Role for Bucket and Elastic access
See [Manual Snapshot Prerequisites](https://docs.aws.amazon.com/de_de/elasticsearch-service/latest/developerguide/es-managedomains-snapshots.html) in the official AWS documentation

##### 3. Create a snapshot repository to create a "reference" location for snapshots

The snapshot repository is the registration for elasticsearch to store and restore snapshots. The following script can be used to fulfil this operation. In the following sections, the header will be used for all scripts and only differnt parts will be described. So the first 9 lines are required for all operations and payload, request type etc. differs for each operation. The variables like `{AWS_REGION}` in the code-sections are required to change and are just placeholders for clarification.

```python
import boto3
import requests
from requests_aws4auth import AWS4Auth

host = '{ELASTICSEARCH_URL}' # include https:// and trailing /
region = '{AWS_REGION}' # e.g. us-west-1
service = 'es'
credentials = boto3.Session().get_credentials()
awsauth = AWS4Auth(credentials.access_key, credentials.secret_key, region, service, session_token=credentials.token)

# Register repository

path = '_snapshot/upgrade_repo' # the Elasticsearch API endpoint
url = host + path

payload = {
  "type": "s3",
  "settings": {
    "bucket": "{BUCKKET_NAME}",
    # "endpoint": "s3.amazonaws.com", # for us-east-1
    "region": "{AWS_REGION}", # for all other regions
    "role_arn": "{SNAPSHOT_TO_S3_ROLE}"
  }
}

headers = {"Content-Type": "application/json"}

r = requests.put(url, auth=awsauth, json=payload, headers=headers)

print(r.status_code)
print(r.text)
```

##### 4. Trigger the manual snapshot creation
```python
# Take snapshot
path = '_snapshot/my-snapshot-repo/my-snapshot'
url = host + path
r = requests.put(url, auth=awsauth)
print(r.text)
```

##### 5 Export all Dashboards, Index Patterns, Visualizations
- Log in to Kibana
- Click on `Management`
- Click on `Saved Objects`
- Mark everything necessary and click on `Export x objects` on top right
- This will trigger the download of a json file `export.json`



### Restore existing Snapshot

The Variable `host = '{ELASTICSEARCH_URL}/'` in the python scripts has to be set on the **NEW** elasticsearch server, because we want to restore the snapshop on it.

##### 1. Upgrade existing Terraform script and raise Version to x.x

##### 2. Create a snapshot repository to create a "reference" location for existing snapshots
The steps are the same like in the backup repo registration, but now with the new elasticsearch url
```python
path = '_snapshot/upgrade_repo' # the Elasticsearch API endpoint
url = host + path
payload = {
  "type": "s3",
  "settings": {
    "bucket": "{BUCKKET_NAME}",
    # "endpoint": "s3.amazonaws.com", # for us-east-1
    "region": "{AWS_REGION}", # for all other regions
    "role_arn": "{SNAPSHOT_TO_S3_ROLE}"
  }
}
headers = {"Content-Type": "application/json"}
r = requests.put(url, auth=awsauth, json=payload, headers=headers)

print(r.status_code)
print(r.text)
```

##### 3. Restore snapshot from repo (excluding `.kibana*`)
The restore can take quite a while, so watch the progress in AWS Console -> Cloudwatch or on the elasticsearch dashboard itself.

```python
path = '_snapshot/{REPO_NAME}/{SNAPHOT_NAME}/_restore'
url = host + path
payload = {
  "indices": [ "{INDEX_1}", "{INDEX_2}" ],
  "include_global_state": False
}
headers = {"Content-Type": "application/json"}
r = requests.post(url, auth=awsauth, json=payload, headers=headers)

print(r.status_code)
print(r.text)
```



##### 4. Import all Dashboards, Index Patterns, Visualizations
- Log in to Kibana
- Click on `Management`
- Click on `Saved Objects`
- Mark everything necessary and click on `Export x objects` on top right
- This will trigger the download of a json file `export.json` 




### Other helpful commands for the backup & restore process

##### Get repository content
```python
path = '_snapshot/{REPO_NAME}'
url = host + path
headers = {"Content-Type": "application/json"}
r = requests.get(url, auth=awsauth, headers=headers)
print(r.text)
```

##### List all snapshots
```python
path = '_snapshot/{REPO_NAME}/_all?pretty'
url = host + path
headers = {"Content-Type": "application/json"}
r = requests.get(url, auth=awsauth, headers=headers)
print(r.text)
```

##### Delete everything
```python
path = '*'
url = host + path
r = requests.delete(url, auth=awsauth)
print(r.text)
```