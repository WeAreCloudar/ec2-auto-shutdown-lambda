EC2 Auto Shutdown
=================
This lambda will check all the running EC2 instances and take the following actions:

- Do nothing if:
     - the instance is part of an autoscaling group
     - the instance is tagged with the tag defined in `EXCLUDE_TAG`
     - the instance has been active (determined by CPUUtilization and the `THRESHOLD` variable) in the last `MAIL_AFTER_DAYS` days
- Send an e-mail if the instance has been active between `MAIL_AFTER_DAYS` ago and `SHUTDOWN_AFTER_DAYS` ago
- Stop the instance if it has not been active between now and `SHUTDOWN_AFTER_DAYS` ago.

Dry Run
-------
By default `DRY_RUN` is set to `True`, to prevent accidental shutdown of instances. You should run this the first time in dry run mode, and look at the cloudwatch logs to determine wich instances will be stopped. Make sure to exclude instances with the `EXCLUDE_TAG` if necessary.

Installation
------------
You should deploy this as a scheduled lambda with the following IAM policy.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeTags",
        "ec2:StopInstances",
        "cloudwatch:GetMetricStatistics",
        "ses:SendEmail"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```


to use SES, `MAIL_FROM` should be a verified address. If the account is still in the SES sandbox, `MAIL_TO` should also be verified.

Configuration
-------------
Please use the constants in the beginning of the lambda function for the configuration

- `DRY_RUN`: See above
- `MAIL_AFTER_DAYS`: The number of days an instance should be inactive before sending an e-mail
- `SHUTDOWN_AFTER_DAYS`: The number of days an instance should be inactive before sending an e-mail
- `THRESHOLD`: If the (maximum) CPUUtilization is below this value, the instance will be considered inactive
- `EXCLUDE_TAG`:  If this tag is present on an instance, no action will be taken
- `MAIL_FROM`: `MAIL_TO` and `MAIL_TEXT` used to configure the sender, receiver and contents of the e-mail.
- `ASG_TAG`:  This should never change.
