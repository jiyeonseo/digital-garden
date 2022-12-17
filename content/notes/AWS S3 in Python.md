---
title: "AWS S3 in Python"
tags:
- aws
- python
---

## generate_presigned_url 
- S3는 기본적으로 private이지만, presigned_url을 통해 권한이 없는 사람에게도 object를 받을 수 있도록 임시로 허용할 수 있다. 
- 만료시간을 주어 access를 조절할 수 있다. 
- presigned_url 만들 시에는 만든 **사용자의 권한**에 따라 제한된다.
	- IAM instance profile : 최대 6시간 사용 가능한 presigned_url 생성 가능 
	- AWS Security Token Service : 최대 36시간 
	- IAM user : 최대 7일 (AWS Signature Version 4 사용시)

```python
import logging
import boto3
from botocore.exceptions import ClientError

def create_presigned_url(bucket_name, object_name, expiration=3600):
    """Generate a presigned URL to share an S3 object

    :param bucket_name: string
    :param object_name: string
    :param expiration: Time in seconds for the presigned URL to remain valid
    :return: Presigned URL as string. If error, returns None.
    """

    # Generate a presigned URL for the S3 object
    s3_client = boto3.client('s3')
    try:
        response = s3_client.generate_presigned_url('get_object',
                                                    Params={'Bucket': bucket_name,
                                                            'Key': object_name},
                                                    ExpiresIn=expiration)
    except ClientError as e:
        logging.error(e)
        return None

    # The response contains the presigned URL
    # https://{bucket_name}.s3.amazonaws.com/{key}?AWSAccessKeyId={aws_access_key}&Signature={signature}&Expires={expire_unixtimestamp}
    return response
```


## Sample Code 
```python
import boto3

S3_ACCESS_KEY = 's3_access_key'
S3_SECRET_KEY = 's3_secret_key'

class S3Client():

    def __init__(self, aws_access_key=S3_ACCESS_KEY, aws_secret_key=S3_SECRET_KEY):
        self.aws_access_key = aws_access_key
        self.aws_secret_key = aws_secret_key
        self.client = boto3.client(
            's3',
            aws_access_key_id=self.aws_access_key,
            aws_secret_access_key=self.aws_secret_key)

    def list_folder_contents(self, bucket_name, folder_name=None, exclude_self=True):
        if folder_name:
            folder_name = os.path.join(folder_name, '') # ensure it ends in a slash
        else:
            folder_name = '' # non-prefixed -- all folders

        objects = []
        incomplete = True
        continuation_token = None

        while incomplete:
            if continuation_token:
                response = self.client.list_objects_v2(
                    Bucket=bucket_name,
                    Prefix=folder_name,
                    ContinuationToken=continuation_token,
                )
            else:
                response = self.client.list_objects_v2(
                    Bucket=bucket_name,
                    Prefix=folder_name,
                )

            objects += response.get('Contents', [])
            if response.get('isTruncated', False):
                continuation_token = response['NextContinuationToken']
            else:
                incomplete = False

        if exclude_self:
            contents = [obj['Key'] for obj in objects if obj['Key'] != folder_name]
        else:
            contents = [obj['Key'] for obj in objects]

        return contents

    def move_object(self, source_bucket_name=None, source_name=None, target_name=None, target_bucket_name=None):
        """
        Moving an object on S3 requires two steps:
        1) copy to destination
        2) delete from source
        :param source_bucket_name: (str) name of bucket to copy from
        :param source_name: (str) object key to copy from
        :param target_name: (str) object key to copy to
        :param target_bucket_name: (str) name of bucket to copy to. If None, use the source_bucket_name
        :return None:
        """

        if target_bucket_name is None:
            target_bucket_name = source_bucket_name

        response = self.client.copy_object(
            Bucket=target_bucket_name,
            Key=target_name,
            CopySource={
                'Bucket': source_bucket_name,
                'Key': source_name
            }
        )

        if response.get('CopyObjectResult', False):
            # Assume it worked
            response = self.client.delete_object(
                Bucket=source_bucket_name,
                Key=source_name
            )


    def upload_file(self, source_name, target_name, bucket_name):
        """
        Uploads the source to the target in the bucket
        :params source_name: (str) name of file to upload
        :params target_name: (str) name of object on S3 (include any folder or path)
        :params bucket_name: (str) bucket to receive file
        :returns: None
        """
        self.client.upload_file(
            source_name,
            bucket_name,
            target_name
        )

    def download_file(self, source_name, target_name, bucket_name):
        """
        Downloads the source object from the bucket into the target file. Note that the target_name paths should already exist.
        :params source_name: (str) object key to download
        :params target_name: (str) destination for download -- all paths to the base file must already exist
        :params bucket_name: (str) name of bucket to download from
        :returns: None
        """
        self.client.download_file(
            bucket_name,
            source_name,
            target_name
        )
	def generate_presigned_url(self, bucket_name, key, expiration=3600):
        return self.client.generate_presigned_url(
            ClientMethod="get_object",
            Params={
                "Bucket": bucket_name,
                "Key": key,
            },
            ExpiresIn=expiration, # seconds
        )
```

## Preferences
- [Example: An S3 proxy client written in Python](https://gist.github.com/tamouse/b5c725082743f663fb531fa4add4b189)
- [Using presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)