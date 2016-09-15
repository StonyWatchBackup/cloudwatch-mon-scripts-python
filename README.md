cloudwatch-mon-scripts-python
=============================

Linux monitoring scripts for CloudWatch.



Docker
------

This software is packaged as a [Docker image](https://hub.docker.com/r/pebbletech/cloudwatch-mon-scripts/) for deployment.

Docker Requirements
-------------------
  * Docker 1.5+


Docker Installation
-------------------
1. Create systemd unit:

    ### docker-cloudwatch.service
    ```
    [Unit]
    Description=someapp cloudwatch stats
    After=docker.service
    Requires=docker.service

    [Service]
    TimeoutStartSec=0
    ExecStartPre=-/usr/bin/docker kill cloudwatch-mon-scripts
    ExecStartPre=-/usr/bin/docker rm cloudwatch-mon-scripts
    ExecStartPre=-/usr/bin/docker pull pebbletech/cloudwatch-mon-scripts
    ExecStart=/bin/sh -c '/usr/bin/docker run -e ASG=$(source /etc/profile; fleetctl list-machines -l | grep $(cat /etc/machine-id) | sed 's/.*service=//g') pebbletech/cloudwatch-mon-scripts'
    ```

2. Start systemd unit

    ```
    systemctl start $PWD/docker-cloudwatch.service
    ```


Source Requirements
-------------------

- Python >= 2.6
- Boto >= 2.33.0


Source Installation
-------------------

Optionally create a virtual environment and activate it. Then just run
`pip install cloudwatchmon`. Install the scripts in /usr/local/bin folder.

For script usage, run:

    mon-put-instance-stats.py --help


Examples
--------

To perform a simple test run without posting data to Amazon CloudWatch

    mon-put-instance-stats.py --mem-util --verify --verbose

Report memory and disk space utilization to Amazon CloudWatch

    mon-put-instance-stats.py --mem-util --disk-space-util --disk-path=/

To get utilization statistics for the last 12 hours

    mon-get-instance-stats.py --recent-hours=12


Configuration
-------------

To allow an EC2 instance to read and post metric data to Amazon CloudWatch,
this IAM policy is required:

    {
      "Statement": [
        {
          "Action": [
            "cloudwatch:ListMetrics",
            "cloudwatch:GetMetricStatistics",
            "cloudwatch:PutMetricData",
            "autoscaling:DescribeAutoScalingInstances"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ]
    }

If the policy is configured via an IAM role that is assigned to the EC2
server this script runs on, you're done.

Otherwise you can configure the policy for a user account and export
the credentials before running the script:

    export AWS_ACCESS_KEY_ID=[Your AWS Access Key ID]
    export AWS_SECRET_ACCESS_KEY=[Your AWS Secret Access Key]

Third option is to create a _~/.boto_ file with this content:

    [Credentials]
    aws_access_key_id = Your AWS Access Key ID
    aws_secret_access_key = Your AWS Secret Access Key


Copyright
---------

Copyright 2015 Oliver Siegmar

Based on Perl-Version of CloudWatch Monitoring Scripts for Linux -
Copyright 2013 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
