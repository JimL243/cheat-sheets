**Read Write Delete to Only specified Bucket Access**

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
				"s3:DeleteObject",
				"s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::bucketname",
                "arn:aws:s3:::bucketname/*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "NotResource": [
                "arn:aws:s3:::bucketname",
                "arn:aws:s3:::bucketname/*"
            ]
        }
    ]
}
```


https://aws.amazon.com/blogs/security/how-to-restrict-amazon-s3-bucket-access-to-a-specific-iam-role/

https://saturncloud.io/blog/how-to-restrict-user-access-to-a-single-s3-bucket-using-amazon-iam/

https://objectivefs.com/howto/how-to-restrict-s3-bucket-policy-to-only-one-aws-s3-bucket

**Anonymous User Write Access to bucket**
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "allow-anon-put",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "s3:PutObject",
      "Resource": [
        "arn:aws:s3:::bucketname/*",
        "arn:aws:s3:::bucketname"
      ]
    }
  ]
}
```




