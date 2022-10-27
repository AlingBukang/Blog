### Create Cloudwatch alert and publish to Slack     
&nbsp;


**Required:**
- Slack Webhook URL
- AWS SNS
- AWS Lambda
- AWS Cloudwatch
  
_Your AWS account should have admin/write permissions to these services._

&nbsp;

Amazon SNS can be used to send notifications to HTTP(S) endpoints, such as Webhook URLs. When confirming the HTTP(S) subscription, some webhooks expect JSON key-value pairs that Amazon SNS does not support. For example, Slack Webhooks expects a JSON request with a message string corresponding to a “text” key. To transform the Amazon SNS message body JSON document for the webhook endpoint to process, use an AWS Lambda function.   

&nbsp;

**Summary of Steps:**
1. Create an SNS Topic
2. Create a Lambda Function
3. Add an SNS topic trigger to the Lambda function
4. Create a Cloudwatch metric filter
5. Create Metric Alert and send notification to the SNS topic

&nbsp;

**Create SNS Topic**
1. In AWS console, navigate to SNS → Topics then, click Create topic.
2. Choose Standard as we don’t care about the order of the notifications to be sent. 
3. Specify the topic name then click Create topic.
4. Verify the Topic was created.

&nbsp;

**Create a Lambda Function**
1. In AWS console, navigate to Lambda → Functions then, click Create function.
2. Choose Author from scratch as we want to write our own script. 
3. Specify a function name.
4. For Runtime, choose Python.  
   _There are other supported languages such as Node, Java, Ruby, etc. Choose the one that you are most comfortable coding._
5. Specify an existing role or leave it to default. If left to default, a new execution role with lambda execution role permissions policy will be created and attached to this Lambda.
6. Create the custom script that will transform SNS message body for the webhook endpoint to process. Slack Incoming Webhooks expect a JSON request with a message string corresponding to a "text" key.

```python 
#!/usr/bin/python3.8
import urllib3 import json
http = urllib3.PoolManager()
def
lambda_handler(event, context):
slack_webhook_url = "https://hooks.slack.com/services/....."
sns_message = {
    "channel": "#test-alerts",
    "username": "alexism",
    "text": event['Records'][0]['Sns']['Message'],
    "icon_emoji": ""
}
encoded_msg = json.dumps(sns_message).encode('utf-8')
resp = http.request('POST', slack_webhook_url, body=encoded_msg) print({
    "message": event['Records'][0]['Sns']['Message'],
    "status_code": resp.status,
    "response": resp.data
})
```

7. Navigate to the lambda Code source then overwrite the pre-created sample script with the python script.
8. Test the function to make sure it works prior deploy.
   - Choose the Test dropdown list. Then, choose Configure test event. 
    - In the Configure test event dialog box, choose Create new event. 
   - For Event template, choose SNS Topic Notification.
   - For Event name, enter a name for the test event.
   - Choose Save.
   - After it's saved, choose Test. Then, review the Execution result.
   - The test invocation should return a 200 status code, otherwise, check the webhook URL is correct.

&nbsp;

**Add an SNS topic trigger to the Lambda function**
1. On the lambda console, navigate to Configuration → Triggers → click Add trigger.
2. Select SNS as trigger source then choose the SNS topic previously created.
3. Click Add.
Lambda will add the necessary permissions for AWS SNS to invoke the Lambda function from this trigger.

&nbsp;

**Create a Cloudwatch Metric Filter**
1. In AWS console, navigate to Cloudwatch → Log groups.
2. Choose the target Medworld log group which is /ecs/medworld-web-definition.
3. Click Actions dropdown → Create metric filter.
4. For Filter pattern, specify the filter pattern to use. Refer to Filter and pattern syntax.
5. Test the filter pattern, under Test Pattern, enter one or more log events to use to test the pattern. Each log event must be within one line, because line breaks are used to separate log events in the Log event messages box.
6. Click Next, then specify the filter name.
7. Under Metric details, for Metric namespace, enter a name for the CloudWatch namespace where the metric will be published. If this
namespace doesn't already exist, be sure that Create new is selected.
8. Specify a name for the new metric.
9. For Metric value, enter 1. This increments the metric by 1 for each log event that includes one of the filter keywords.
10. Click Next, review the details then click Create metric filter.

&nbsp;

**Create a Cloudwatch Alarm on a static threshold** 
1. Continuing from previous step, click the Metric filters tab.
2. Click the metric filter checkbox then Create alarm.
3. The Specify metric and conditions page appears. 
- For Statistic, choose Sum.
- For Period, choose the evaluation period for the alarm. When evaluating the alarm, each period is aggregated into one data point.
- For Conditions, specify the following: 
    - Static threshold
    - For Whenever metric is, specify whether the metric must be greater than, less than, or equal to the threshold. Under than..., specify the threshold value.
    - For Datapoints to alarm, specify how many evaluation periods (data points) must be in the ALARM state to trigger the alarm. If the two values here match, you create an alarm that goes to ALARM state if that many consecutive periods are breaching.
4. Choose Next for the Notification.
5. Select the SNS topic to notify when the alarm is in ALARM state.
6. Choose Next, then enter a name for the alarm and click Next again.
7. Under Preview and create, verify the information and conditions, then choose Create alarm.  

&nbsp;

&nbsp;


[/TODO] Create Terraform version