---
title: "AWS CLI"
tags:
- AWS
---


## AWS Configuration 설정 확인하기 
`aws configure`로 구성한 credential 확인하려면 아래와 같이 할 수 있다.

```sh
vi ~/.aws/credential

[default]
aws_access_key_id={ACCESS_KEY}
aws_secret_access_key={SECRET_ACCESS_KEY}

vi ~/.aws/config

[default]
region=us-west-2
output=json
```

- https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html


## AWS S3 파일 옮기기

```sh
aws s3 mv s3://{from_path} s3://{to_path} --recursive
```

- web console 에서도 가능하다
![](https://user-images.githubusercontent.com/2231510/206854577-026b629c-6c0d-4e25-a8bd-a47fda6dab95.png)
