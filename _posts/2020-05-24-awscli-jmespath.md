---
layout: post
title: "Get most recent S3 object using AWSCLI"
date: 2020-05-24
description: How to effectively use AWSCLI with JMESPath
toc: true
share: true
tags:
 - awscli
 - jmespath
---

# Overview
If you are working in cloud and that too specifically in AWS you will be familiar with the [AWS Commandline Interface](https://aws.amazon.com/cli/). It is quite powerful and very useful to quickly accomplish something and also script it at the same time. It can return results in either text or [json](https://www.json.org/json-en.html) format. Json is useful when you need to programmatically interpret the results. 

AWSCLI provides a `--query` option to filter the command results based on [JMESPath](https://jmespath.org/). If you haven't already take a moment to go through AWS documentation on [Controlling command output from the AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html). 

The blog post [Advanced AWS CLI JMESPath Query Tricks](https://opensourceconnections.com/blog/2015/07/27/advanced-aws-cli-jmespath-query/) talks about it really well. 

# Problem
A few months ago I faced a problem, where I had to programmaticaly fetch the most recent object from an S3 bucket given the bucket prefix. First, I had to figure out how to get metadata about s3 objects which includes last modified timestamp. Then I had to figure out how to sort the objects based on the timestamp. I knew JMESPath had `sort` and `sort_by` methods. Having never used them before, it took me some time to figure out the correct command. Here, I have attempted to explain the thoughtflow and process I went though to figure out the solution

Let's solve this one step at a time. 

## The base command
This is the raw AWSCLI command to fetch all s3 objects matching the given prefix
```
aws s3api list-objects --bucket <your bucket name>  --prefix "<object prefix>"
```
This provides result as shown in [AWSCLI reference](https://docs.aws.amazon.com/cli/latest/reference/s3api/list-objects.html#output). Shown an example here as well for convenience:
```
{
    "Contents": [
        {
            "Key": "SomePrefix/MyObject1"
            "LastModified": "2019-02-01T00:00:04.000Z",
            "ETag": "\"445175bab9cfc988fbe495a2a49b65d5\"",
            "Size": 7889,
            "StorageClass": "STANDARD",
            "Owner": {
                "DisplayName": "my_owner-name",
                "ID": "9e6b02405e2933107e2d7489d7a22ca772ae1eecf4677a27517b9ec21d248d9b"
            }
        },
       ....
    ]
}
```


## JMESPath Query
Now all we have to do is sort the list under `Contents` based on `LastModified` timestamp. So we add `--query 'sort_by(Contents, &LastModified)` to the command. You can find examples for `sort_by` method at https://jmespath.org/examples.html#using-functions. This gives us a list of objects in a sorted order. The command now becomes
```
aws s3api list-objects --bucket <your bucket name>  --prefix "<object prefix>"  --query 'sort_by(Contents, &LastModified)
```
But we only want the most recently modified key. Now that we have a sorted list, the most recently modified key will be the last item in the list. So we pipe the output of the previous query and get the last item using `[-1]`. Since we are only interested in the name of the s3 key we can refine this further to get only the key by `[-1].Key`.

Finally, we have our command!

```
aws s3api list-objects --bucket <your bucket name>  --prefix "<object prefix>"  --query 'sort_by(Contents, &LastModified) | [-1].Key' --output text
```
# Conclusion
I have used this a number of times in various contexts. Hope this will be useful to you too. 
