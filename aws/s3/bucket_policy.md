Bucket Policy
==========

Bucket policies override ACL's _most_ of the time. This isn't always true though. There's a default ACL that creates a _DENY_ to everyone outside your account. If you add any allows to the ACL then this will start to poke unexpected holes in your policy. My general strategy has been to allow the default ACL to exist and all other rules are funneled through the policy. This makes the bucket policy the single source of truth.

The only exception I've ever made to this philosophy was when I had to dynamically update policies. Usually if you're doing buckets right though, this isn't a thing. Just assign one bucket to one function, this steals partly from the Linux methodology of _"Do one thing and do it well."_

Note: As a general rule I avoid altering the ACL's. If you have to, there's probably a better way of doing what you're attempting to do.

## Principals

Principals are specifying who the _ALLOW_ or _DENY_ apply to.

Warning: One thing to keep in mind is if you use "\*" then everything, including resources outside your account will have access.

## Example

Below is an example of a well balanced policy.

```json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<account number>:root"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<bucket name>/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                }
            }
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<account number>:role/<role name>"
            },
            "Action": [
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::<bucket name>/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<account number>:role/<role name>"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::<bucket name>"
        }
    ]
}
```

In the first policy I'm allowing anything in my account to utilize `s3:PutObject`. Note the `/*`, this indicates that you can use this function on the bucket and it's folders. I also specify that anything utilizing PutObject must use a KMS key (So everything in the bucket is encrypted).

The second policy allows anyone using a specific role to utilize `s3:GetObject` and `s3:DeleteObject`. This role is part of an `STS:AssumeRole` trust policy. The third policy is much of the same.
