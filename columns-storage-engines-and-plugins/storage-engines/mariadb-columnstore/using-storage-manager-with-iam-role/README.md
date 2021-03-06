# Using Storage Manager with IAM Role

## AWS IAM Role Configuration

We have added a new feature in Columnstore 5.5.2 that allows you to use AWS IAM roles in order to connect to S3 buckets without explicitly entering credentials into the <strong>storagemanager.cnf</strong> config file.

You will need to modify the IAM role of your Amazon EC2 instance to allow for this. Please follow the AWS [documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) before beginning this process.

It is important to note that you must update the AWS S3 endpoint based on your chosen region otherwise you might face delays in propagation as discussed [here](https://forums.aws.amazon.com/thread.jspa?messageID=552808) and [here](https://forums.aws.amazon.com/thread.jspa?messageID=801522).

For a complete list of AWS service endpoints, please visit the AWS  [reference guide](https://docs.aws.amazon.com/general/latest/gr/rande.html).

### Sample configuration

Edit your <strong>Storage Manager</strong> configuration file located at <em>/etc/columnstore/storagemanager.cnf</em> in order to look similar to the example below (replacing those in the [S3] section with your own custom variables):

```sql
[ObjectStorage]
service = S3
object_size = 5M
metadata_path = /var/lib/columnstore/storagemanager/metadata
journal_path = /var/lib/columnstore/storagemanager/journal
max_concurrent_downloads = 21
max_concurrent_uploads = 21
common_prefix_depth = 3

[S3]
ec2_iam_mode=enabled
bucket = my_mcs_bucket
region = us-west-2
endpoint = s3.us-west-2.amazonaws.com

[LocalStorage]
path = /var/lib/columnstore/storagemanager/fake-cloud
fake_latency = n
max_latency = 50000

[Cache]
cache_size = 2g
path = /var/lib/columnstore/storagemanager/cache
```

<em><strong>Note</strong>: This is an AWS only feature. For other deployment methods, see the example [here](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/sample-storagemanagercnf/).</em>