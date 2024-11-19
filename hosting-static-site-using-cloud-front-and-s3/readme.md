# Setting Up AWS CloudFront as an HTTPS Endpoint for S3
## Introduction
In this hands-on lab, we'll be setting up a CloudFront distribution in front of an S3 bucket website and securing it via HTTPS provided by CloudFront. AWS CloudFront is a versatile caching service (CDN) that helps with lowering latency when accessing objects over the internet. Additionally, it can act as an HTTPS termination point for your cached website — thus, providing a secure way of distributing your content over the internet.

# Solution
Log in to the AWS Management Console using the credentials provided on the lab instructions page. Make sure you're using the `us-east-1` Region.

Ensure there is an available bucket under Amazon S3>Buckets.

In a different tab, navigate to the EC2 dashboard, find the running instance, and then connect by doing the following:

1. Navigate to EC2> Instances (running).
2. Click on the checkbox next to the instance name.
3. In the upper right corner, select Connect.
4. Choose Session Manager, and select Connect.
5. Once connected, assume the `cloud_user` identity by running: `sudo su cloud_user`.
6. Run `cd` to get to the `cloud_user` home.
7. Print the working directory: `pwd`.
8. Confirm you are cloud_user: `whoami`.

> **Note:** When copying and pasting code into Vim from the lab guide, first enter `:set paste` (and then `i` to enter insert mode) to avoid adding unnecessary spaces and hashes. To save and quit the file, press `Escape` followed by `:wq`. To exit the file without saving, press `Escape` followed by `:q!`.

> **Note**: If the EC2 instance is not showing the ability to connect via Session Manager, simply restart the instance to restart the SSM Agent.
## Create a S3 bucket
1. Use a unique name
2. Unselect block all public access and check the 1st two properties.
3. Acknowledge and save changes to create a bucket
4. Go to the properties section of bucket and edit static web hosting properties
5. Enable static web hosting and add two file name index.html and erro.html as index doc and error doc
6. Now get to the permission section and edit bucket policy
7. Add the following policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```
8. Upload a index.html to the root folder of the bucket. And we are good to go.
## Upload Content to S3 Bucket
1. Verify pre-existing files:
```sh
ls
```
2. Upload the `index.html` file to the bucket (replacing `<BUCKET_NAME_PROVIDED>` with the S3 bucket name listed on the lab page):
```sh
aws s3 cp index.html s3://<BUCKET_NAME_PROVIDED>
```
```sh
aws s3 cp index.html s3://s3websitebucket-755260906016
```
## Create CloudFront OAI
1. Generate a UUID by running `uuidgen` in the console. Copy and save the UUID in a separate text file; use that for the next step.
2. Create a CloudFront origin access identity (replacing `<YOUR_UNIQUE_UUIDGEN_HERE>` with the uuidgen):
```sh
aws cloudfront create-cloud-front-origin-access-identity --cloud-front-origin-access-identity-config CallerReference=<YOUR_UNIQUE_UUIDGEN_HERE>,Comment=MyOAI
```
```sh
aws cloudfront create-cloud-front-origin-access-identity --cloud-front-origin-access-identity-config CallerReference=4d8099c8-e6cd-44f9-8f76-d6793a0ff15a,Comment=MyOAI
```
Response:
```json
{
    "CloudFrontOriginAccessIdentity": {
        "CloudFrontOriginAccessIdentityConfig": {
            "Comment": "MyOAI",
            "CallerReference": "4d8099c8-e6cd-44f9-8f76-d6793a0ff15a"
        },
        "S3CanonicalUserId": "4789268f00a309df0c217f9937b02067a60c97f282c31c9eb5087a34c25db7ac8fecf0f350cf5821239d5656e9d6f3e9",
        "Id": "E22CM7K19WW245"
    },
    "ETag": "E58SNVQ15VZ63",
    "Location": "https://cloudfront.amazonaws.com/2020-05-31/origin-access-identity/cloudfront/E22CM7K19WW245"
}
```

3. Copy the `Id` from the output and paste it into a text file, as we'll need it for the next step.
4. Also, save the `OAI Id` to an environment variable:
```sh
export OAI_ID=<Id value here>
```
```sh
export OAI_ID=E22CM7K19WW245
```
## Modify S3 Policy File in Directory and Execute It Against S3 Bucket
1. Run the next command to substitute your `OAI` into the `policy_cloudfront.json` document.
2. `sed "s/<OAI-ID>/$(echo $OAI_ID)/" policy_cloudfront.json` to test the changes.
3. Run `sed -i "s/<OAI-ID>/$(echo $OAI_ID)/" policy_cloudfront.json` to save the changes.
4. Execute this policy against the S3 website bucket (NOTE: The `<BUCKET_NAME>` is the S3 Website BucketName found in the credentials section.):
```sh
aws s3api put-bucket-policy --bucket <BUCKET_NAME> --policy file://policy_cloudfront.json
```
```sh
aws s3api put-bucket-policy --bucket s3websitebucket-755260906016 --policy file://policy_cloudfront.json
```
5. Navigate to the Amazon S3 Console, click on the bucket, and go to the Permissions tab to confirm the bucket policy has been updated.
## Create CloudFront Distribution
1. Create a CloudFront distribution:
```sh
aws cloudfront create-distribution --origin-domain-name <BUCKET_NAME>.s3.amazonaws.com --default-root-object index.html
```
```sh
aws cloudfront create-distribution --origin-domain-name s3websitebucket-755260906016.s3.amazonaws.com --default-root-object index.html
```
Response:
```json

{
    "Distribution": {
        "Status": "InProgress",
        "DomainName": "d2cn6bws26xdpi.cloudfront.net",
        "InProgressInvalidationBatches": 0,
        "DistributionConfig": {
            "Comment": "",
            "CacheBehaviors": {
                "Quantity": 0
            },
            "IsIPV6Enabled": true,
            "Logging": {
                "Bucket": "",
                "Prefix": "",
                "Enabled": false,
                "IncludeCookies": false
            },
            "WebACLId": "",
            "Origins": {
                "Items": [
                    {
                        "OriginPath": "",
                        "DomainName": "s3websitebucket-755260906016.s3.amazonaws.com",
                        "ConnectionAttempts": 3,
                        "CustomHeaders": {
                            "Quantity": 0
                        },
                        "ConnectionTimeout": 10,
                        "S3OriginConfig": {
                            "OriginAccessIdentity": ""
                        },
                        "Id": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099"
                    }
                ],
                "Quantity": 1
            },
            "DefaultRootObject": "index.html",
            "PriceClass": "PriceClass_All",
            "Enabled": true,
            "DefaultCacheBehavior": {
                "FieldLevelEncryptionId": "",
                "TargetOriginId": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099",
                "LambdaFunctionAssociations": {
                    "Quantity": 0
                },
                "TrustedSigners": {
                    "Enabled": false,
                    "Quantity": 0
                },
                "ViewerProtocolPolicy": "allow-all",
                "ForwardedValues": {
                    "Headers": {
                        "Quantity": 0
                    },
                    "Cookies": {
                        "Forward": "none"
                    },
                    "QueryStringCacheKeys": {
                        "Quantity": 0
                    },
                    "QueryString": false
                },
                "MaxTTL": 31536000,
                "SmoothStreaming": false,
                "DefaultTTL": 86400,
                "AllowedMethods": {
                    "Items": [
                        "HEAD",
                        "GET"
                    ],
                    "CachedMethods": {
                        "Items": [
                            "HEAD",
                            "GET"
                        ],
                        "Quantity": 2
                    },
                    "Quantity": 2
                },
                "MinTTL": 0,
                "Compress": false
            },
            "CallerReference": "cli-1732010921-118821",
            "ViewerCertificate": {
                "CloudFrontDefaultCertificate": true,
                "SSLSupportMethod": "vip",
                "MinimumProtocolVersion": "TLSv1",
                "CertificateSource": "cloudfront"
            },
            "CustomErrorResponses": {
                "Quantity": 0
            },
            "OriginGroups": {
                "Quantity": 0
            },
            "HttpVersion": "http2",
            "Restrictions": {
                "GeoRestriction": {
                    "RestrictionType": "none",
                    "Quantity": 0
                }
            },
            "Aliases": {
                "Quantity": 0
            }
        },
        "ActiveTrustedSigners": {
            "Enabled": false,
            "Quantity": 0
        },
        "LastModifiedTime": "2024-11-19T10:08:41.574Z",
        "Id": "E1UOOOPI28P7FY",
        "ARN": "arn:aws:cloudfront::755260906016:distribution/E1UOOOPI28P7FY"
    },
    "ETag": "EL6TYA1XAP3L0",
    "Location": "https://cloudfront.amazonaws.com/2020-05-31/distribution/E1UOOOPI28P7FY"
}
```
2. Copy the `ETag` and Id of the CloudFront distribution from within the returned `JSON` string and paste them into a text file, as we'll need them later.
3. Export the Id to an environment variable:
```sh
export CF_ID=<Id value here>
```
```sh
export CF_ID=E1UOOOPI28P7FY
```
4. Export the ETag to an environment variable:
```sh
export CF_ETAG=<Id value here>
```
```sh
export CF_ETAG=EL6TYA1XAP3L0
```
## Get and Update the CloudFront Distribution Config
1. Get the CloudFront distribution config and filter out the distribution-specific part:
```sh
aws cloudfront get-distribution-config --id $CF_ID | jq -r .DistributionConfig > dist-config.json

cat dist-config.json
```
```json
{
  "Comment": "",
  "CacheBehaviors": {
    "Quantity": 0
  },
  "IsIPV6Enabled": true,
  "Logging": {
    "Bucket": "",
    "Prefix": "",
    "Enabled": false,
    "IncludeCookies": false
  },
  "WebACLId": "",
  "Origins": {
    "Items": [
      {
        "OriginPath": "",
        "DomainName": "s3websitebucket-755260906016.s3.amazonaws.com",
        "ConnectionAttempts": 3,
        "CustomHeaders": {
          "Quantity": 0
        },
        "ConnectionTimeout": 10,
        "S3OriginConfig": {
          "OriginAccessIdentity": ""
        },
        "Id": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099"
      }
    ],
    "Quantity": 1
  },
  "DefaultRootObject": "index.html",
  "PriceClass": "PriceClass_All",
  "Enabled": true,
  "DefaultCacheBehavior": {
    "FieldLevelEncryptionId": "",
    "TargetOriginId": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099",
    "LambdaFunctionAssociations": {
      "Quantity": 0
    },
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "ViewerProtocolPolicy": "allow-all",
    "ForwardedValues": {
      "Headers": {
        "Quantity": 0
      },
      "Cookies": {
        "Forward": "none"
      },
      "QueryStringCacheKeys": {
        "Quantity": 0
      },
      "QueryString": false
    },
    "MaxTTL": 31536000,
    "SmoothStreaming": false,
    "DefaultTTL": 86400,
    "AllowedMethods": {
      "Items": [
        "HEAD",
        "GET"
      ],
      "CachedMethods": {
        "Items": [
          "HEAD",
          "GET"
        ],
        "Quantity": 2
      },
      "Quantity": 2
    },
    "MinTTL": 0,
    "Compress": false
  },
  "CallerReference": "cli-1732010921-118821",
  "ViewerCertificate": {
    "CloudFrontDefaultCertificate": true,
    "SSLSupportMethod": "vip",
    "MinimumProtocolVersion": "TLSv1",
    "CertificateSource": "cloudfront"
  },
  "CustomErrorResponses": {
    "Quantity": 0
  },
  "OriginGroups": {
    "Quantity": 0
  },
  "HttpVersion": "http2",
  "Restrictions": {
    "GeoRestriction": {
      "RestrictionType": "none",
      "Quantity": 0
    }
  },
  "Aliases": {
    "Quantity": 0
  }
}
```
2. List the files:
```sh
ls
```
3. Open the newly created `dist-config.json` file:

vim dist-config.json
4. Modify the following properties in the file (NOTE: You will need the OAI_Id.):
```sh
"OriginAccessIdentity": "origin-access-identity/cloudfront/<OAI_ID>"

"OriginAccessIdentity": "origin-access-identity/cloudfront/E22CM7K19WW245"
"PriceClass": "PriceClass_100"
"ViewerProtocolPolicy": "redirect-to-https"
```
5. Save and exit the file:
```sh
:wq
```
## Update CloudFront Distribution with the Modified `dist-config.json` File
1. Update the CloudFront distribution with the `dist-config.json` file:
```sh
aws cloudfront update-distribution --id $CF_ID --distribution-config file://dist-config.json --if-match $CF_ETAG
```
Response:
```json
{
    "Distribution": {
        "Status": "InProgress",
        "DomainName": "d2cn6bws26xdpi.cloudfront.net",
        "InProgressInvalidationBatches": 0,
        "DistributionConfig": {
            "Comment": "",
            "CacheBehaviors": {
                "Quantity": 0
            },
            "IsIPV6Enabled": true,
            "Logging": {
                "Bucket": "",
                "Prefix": "",
                "Enabled": false,
                "IncludeCookies": false
            },
            "WebACLId": "",
            "Origins": {
                "Items": [
                    {
                        "OriginPath": "",
                        "DomainName": "s3websitebucket-755260906016.s3.amazonaws.com",
                        "ConnectionAttempts": 3,
                        "CustomHeaders": {
                            "Quantity": 0
                        },
                        "ConnectionTimeout": 10,
                        "S3OriginConfig": {
                            "OriginAccessIdentity": "origin-access-identity/cloudfront/E22CM7K19WW245"
                        },
                        "Id": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099"
                    }
                ],
                "Quantity": 1
            },
            "DefaultRootObject": "index.html",
            "PriceClass": "PriceClass_100",
            "Enabled": true,
            "DefaultCacheBehavior": {
                "FieldLevelEncryptionId": "",
                "TargetOriginId": "s3websitebucket-755260906016.s3.amazonaws.com-1732010921-313099",
                "LambdaFunctionAssociations": {
                    "Quantity": 0
                },
                "TrustedSigners": {
                    "Enabled": false,
                    "Quantity": 0
                },
                "ViewerProtocolPolicy": "redirect-to-https",
                "ForwardedValues": {
                    "Headers": {
                        "Quantity": 0
                    },
                    "Cookies": {
                        "Forward": "none"
                    },
                    "QueryStringCacheKeys": {
                        "Quantity": 0
                    },
                    "QueryString": false
                },
                "MaxTTL": 31536000,
                "SmoothStreaming": false,
                "DefaultTTL": 86400,
                "AllowedMethods": {
                    "Items": [
                        "HEAD",
                        "GET"
                    ],
                    "CachedMethods": {
                        "Items": [
                            "HEAD",
                            "GET"
                        ],
                        "Quantity": 2
                    },
                    "Quantity": 2
                },
                "MinTTL": 0,
                "Compress": false
            },
            "CallerReference": "cli-1732010921-118821",
            "ViewerCertificate": {
                "CloudFrontDefaultCertificate": true,
                "SSLSupportMethod": "vip",
                "MinimumProtocolVersion": "TLSv1",
                "CertificateSource": "cloudfront"
            },
            "CustomErrorResponses": {
                "Quantity": 0
            },
            "OriginGroups": {
                "Quantity": 0
            },
            "HttpVersion": "http2",
            "Restrictions": {
                "GeoRestriction": {
                    "RestrictionType": "none",
                    "Quantity": 0
                }
            },
            "Aliases": {
                "Quantity": 0
            }
        },
        "ActiveTrustedSigners": {
            "Enabled": false,
            "Quantity": 0
        },
        "LastModifiedTime": "2024-11-19T10:21:13.680Z",
        "Id": "E1UOOOPI28P7FY",
        "ARN": "arn:aws:cloudfront::755260906016:distribution/E1UOOOPI28P7FY"
    },
    "ETag": "E39F3T387EDH4J"
}
```
Hit `d2cn6bws26xdpi.cloudfront.net` to see the site.
## Test and Verify
1. In the AWS console, navigate to CloudFront Console.
2. Click on the only distribution listed to view the details; CloudFront distributions take about 20 minutes to propagate, so give it some time to complete before testing.
3. Click the Origins tab.
4. Select the radio button for the origin listed, and click Edit.
5. Review the settings, then click Cancel to exit.
6. Click the Behaviors tab and confirm the Viewer protocol policy is Redirect HTTP to HTTPs.
7. Click the General tab and copy the domain name.
8. Once it's deployed, paste the copied domain name to a new browser tab to to verify our CloudFront distribution is serving the S3 static website properly.
## Conclusion
Congratulations on successfully completing this hands-on lab!