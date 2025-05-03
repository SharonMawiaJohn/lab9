# lab9
Guided Lab: Encrypting Data at Rest by Using AWS Encryption Options
Lab overview and objectives
In this lab, you review the default data encryption and AWS Key Management Service (AMS KMS) encryption used to encrypt data at rest. You review the default encryption of the objects stored in Amazon Simple Storage Service (Amazon S3). You create an AWS KMS key and use it to encrypt objects stored in Amazon Elastic Block Store (Amazon EBS) volumes. You also observe how AWS CloudTrail provides an audit log of AWS KMS key usage and how disabling the key affects data access. 

After completing this lab, you should be able to do the following:

Review the default encryption provided by Amazon S3.

Access the encrypted Amazon S3 object.

Create an AWS KMS customer managed key to encrypt and decrypt data at rest.

Create and attach an encrypted data volume on an existing EC2 instance.

Disable and re-enable an AWS KMS key and observe the effects on data access.

Monitor encryption key usage by using CloudTrail event history.

Review key rotation.

Duration
This lab requires approximately 30 minutes to complete.

AWS service restrictions
In this lab environment, access to AWS services and service actions might be restricted to the ones that are needed to complete the lab instructions. You might encounter errors if you attempt to access other services or perform actions beyond those that this lab describes.

Scenario
In this lab, you start with the following architectural components: 

An S3 bucket to store images 

An EC2 instance 

An AWS KMS key to be used for encryption in the lab

The following diagram shows these components.

Architecture at the beginning of the lab showing components created when the lab is started.

By the end of this lab, you will have created the architecture in the following diagram. This diagram shows some of the actions that you take and the communication that occurs between Amazon EC2, Amazon EBS, AWS KMS, and CloudTrail during the lab.

Architecture at the end of the lab. The diagram description includes more information.

Diagram description: A user is connected to AWS Cloud resources through the internet. The S3 bucket named ImageBucket contains an object encrypted with server-side encryption with Amazon S3 managed keys (SSE-S3). The encrypted volume is attached to the EC2 instance named LabInstance. Requests to encrypt or decrypt EBS volumes are sent to AWS KMS, where the customer managed key named MyKMSKey is stored. CloudTrail logs AWS KMS key usage as events.

 

Step	Explanation
1	User attaches encrypted EBS volume to the EC2 instance. Instance receives an encrypted data key from the volume.
2	EC2 instance sends request to AWS KMS to decrypt the encrypted data key.
3	AWS KMS validates the users and role permissions and sends the plain data key back to the EC2 instance.
4	The EC2 instance uses the key received for decryption and stores it in memory for the encryption and decryption operation.
 

Accessing the AWS Management Console
 

At the top of these instructions, choose  Start Lab.

The lab session starts.

A timer displays at the top of the page and shows the time remaining in the session.

 Tip: To refresh the session length at any time, choose  Start Lab again before the timer reaches 0:00.

Before you continue, wait until the circle icon to the right of the AWS  link in the upper-left corner turns green. When the lab environment is ready, the AWS Details panel will also display.

To connect to the AWS Management Console, choose the AWS link in the upper-left corner above the terminal window.

A new browser tab opens and connects you to the console.

 Tip: If a new browser tab does not open, a banner or icon is usually at the top of your browser with a message indicating that your browser is preventing the site from opening pop-up windows. Choose the banner or icon, and then choose Allow pop-ups.

 

Task 1: Reviewing default encryption for objects in an S3 bucket
In this task, you upload an image file to an S3 bucket and review the default encryption provided by Amazon S3. 

Download the file named clock.png to your computer.

  Next, you locate the S3 bucket and analyze its encryption settings.

At the top of AWS Management Console, in the search box, search for and choose S3 to open the Amazon S3 console.

In the navigation pane, choose Buckets.

Choose the link for the bucket name that contains imagebucket.

Choose the Properties tab.

  In the Default encryption section, notice the setting Server-side encryption is automatically applied to new objects stored in this bucket.

   Note: Amazon S3 now applies server-side encryption with Amazon S3 managed keys (SSE-S3) as the base level of encryption for every bucket in Amazon S3.

  Next, you upload a file to the bucket.

At the top of the page, choose the Objects tab.

Choose Upload.

Choose Add files.

Browse to and select the clock.png file that you downloaded to your computer.

Expand the Permissions option.

Choose Grant public-read access.

  The following warning message appears: Granting public-read access is not recommended.

To acknowledge the warning, select I understand the risk of granting public-read access to the specified objects.

At the bottom of the page, choose Upload.

After the upload completes, in the upper-right corner, choose Close.

  The clock.png file is now listed as an object in the bucket.

To open the file properties, choose the clock.png file.

In the Server-side encryption settings section, notice the following settings: 

This encryption setting is enabled, and the following message is displayed: Server-side encryption protects data at rest.

For Encryption type, the following message is displayed: Server-side encryption with Amazon S3 managed keys (SSE-S3).

To open the image, in the Object overview section, choose the Object URL. 

  You can use this method to access an Amazon S3 object from the public internet.

  The clock image opens in a browser tab. 

   Note: The object is encrypted, but while it displays, it has to be decrypted. The service handles this decryption transparently.

In this task, you reviewed the encryption settings on the bucket. You then uploaded an object to the bucket and accessed the object by using a public link for the object. Amazon S3 transparently decrypted the object before displaying it in the browser.

Task 2: Creating an AWS KMS key
In this task, you create a customer managed AWS KMS key. Later in the lab, you use the AWS KMS key that you create to generate, encrypt, and decrypt data keys. The data keys will be shared with Amazon EC2. The data keys are used to encrypt the actual data stored on EBS volumes.

In the AWS Management Console, in the search box, search for KMS and choose Key Management Service to open the AWS KMS console.

In the navigation pane, choose Customer managed keys.

   Tip: To open the navigation pane, choose the  menu icon in the upper-left corner. 

Choose Create key.

For Key type, choose Symmetric.

   Note: Symmetric keys never leave AWS KMS unencrypted. The key that you create will be a 256-bit secret key.

Choose Next.  

For Alias, enter MyKMSKey.

Choose Next. 

On the Define key administrative permissions page, for Key administrators, search for and select the voclabs role.

   Note: Key administrators are users or roles that manage access to the encryption key. You are currently signed in to the console with the voclabs role. This user role is displayed in the upper-right corner of the console to the left of the AWS Region. 

Choose Next.

On the Define key usage permissions page, for Key users, search for and select the voclabs role again.

   Note: Key users are the users or roles that can use the key to encrypt and decrypt data. 

Choose Next.

At the bottom of the Review page, choose Finish.

  Your key displays in the list of customer managed keys.

   Note: If you attempted this lab within the past 7 days, you might see an older key or older keys in this list with the status Pending deletion. The minimum amount of time that must transpire between when you request that a key be deleted and when it's actually deleted is 7 days. 

In this task, you created an AWS KMS key, which you use in the next task.

Task 3: Creating and attaching encrypted data volume on an EC2 instance
In this task, you create an encrypted EBS volume by using the KMS key that you created in the previous task and attach it to your EC2 instance. When you attach the encrypted volume, the EC2 instance retrieves the data key from AWS KMS and uses it to decrypt the data on the EBS volume. In later tasks, you examine the CloudTrail event history to observe the calls made to the AWS KMS service.

First, you observe the root volume.

At the top of AWS Management Console, in the search box, search for and choose EC2 to open the Amazon EC2 console.

In the navigation pane, choose Instances.

  The LabInstance is listed. Note the Availability Zone for this EC2 instance. You need this information in a later step.

Choose the Instance ID link for the LabInstance.

Choose the Storage tab.

  In the Block devices section, notice that the volume that is attached indicates it is not encrypted. This is the root volume, which contains the guest operating system installation. It was created when the the EC2 instance was created.

Next, you create the new encrypted volume.

From the navigation pane, choose Volumes.

Choose Create volume.

On the Create volume page, configure the following settings:

For Volume type, keep the default.

For Size (GiB), enter 1.

For Availability Zone, choose the Availability Zone where the EC2 instance exists. You checked for this information in a previous step.

For Encryption, select  Encrypt this volume.

For KMS key, choose MyKMSKey. Notice the details about the key that are displayed.

Choose Create volume.

  The volume is created and will have a Volume state of Available soon.

  Next, you attach the new volume.

Choose the link for the volume that you created.

Choose Actions, and then choose Attach volume.

On the Attach volume page, for Instance, choose your EC2 instance from the dropdown list.

  A Device name is allocated and displayed similar to /dev/sdf.

Choose Attach volume.

  The volume is attached to the instance.

Open a new browser tab, navigate to the the Amazon EC2 console, and choose Instances.

Select  LabInstance.

Choose the Storage tab.

  Two volumes are attached to the instance:

The root volume (8 GiB) is not encrypted.

The data volume (1 GiB) that you attached is encrypted and has a KMS key ID.

   Note: You might need to refresh the page to see the latest information for the attached volume.

   Note: Observe the second diagram to understand what happens when you attach an encrypted EBS volume to the instance.

In this task, you created a new encrypted data volume and attached it to the EC2 instance so that the data on it is now more secure.

Task 4: Disabling the encryption key and observing the effects
In this task, you temporarily disable the AWS KMS key that you previously used to encrypt the EBS volume. You then observe the effects that disabling the key has on accessing encrypted data.

First, you disable the AWS KMS key that you used to encrypt the EBS volume.

Navigate to the AWS KMS console.

In the navigation pane, choose Customer managed keys.

Select  MyKMSKey, and then choose Key actions > Disable.

In the Disable key window that appears, to confirm the action, select  Confirm that you want to disable this key, and then choose Disable key.

  Next, you detach the data volume from EC2 instance. It is necessary for this test to detach the instance. The encrypted data key is stored in memory and used to encrypt and decrypt all data going to and from the attached EBS volume. Amazon EBS retains the encrypted data key for later use in case the data key in memory is no longer available.

Navigate to the Amazon EC2 console.

In the navigation pane, choose Volumes.

Choose the link for the volume that you created.

Choose Actions > Detach volume.

In the Detach window that appears, choose Detach.

Next to Actions, choose the Refresh button to refresh the page.

Choose Actions > Attach volume.

For Instance, choose your instance from the dropdown list.

Choose Attach volume.

  A message similar to the following appears: Volume vol-072d7cf3173a13151 cannot be attached. The encrypted volume was unable to access the KMS key.

  The operation fails as expected because the AWS KMS key used to encrypt the data volume is now disabled and can no longer be used to decrypt the data.

  Next, you analyze the CloudTrail event history.

In the AWS Management Console, in the search box, search for and choose CloudTrail to open the CloudTrail console.

In the navigation pane, choose Event history.

   Tip: To open the navigation pane, choose the  menu icon in the upper-left corner.

   Note: CloudTrail provides an audit log of API calls that are made in the AWS account. The event history provides access to events from the last 90 days of account activity.

  The recent events are displayed.

Choose the link for the DisableKey event name.

  In the Event record section, review the details of the event. You should see information similar to the following record:

  DisableKey event record. Highlighted information includes principalId, eventName, keyId, and EBS object name.

  In the Details section, examine the event details. The Event time show that you disabled the AWS KMS key a few minutes ago.

  Next, you examine the AttachVolume event, which occurred just after the DisableKey event.

In the navigation pane, choose Event history.

Choose the link for the AttachVolume event name.

  In the Event record section, review the details of the event. You should see information similar to the following record:

  AttachVolume event record. Highlighted information includes principalId, eventName, keyId, and EBS object name.

  The details show that the request to attach the volume to the instance was unsuccessful. 

  This happened because the attach volume process noticed that the data volume was encrypted with the MyKMSKey, so Amazon EC2 contacted AWS KMS to provide the plaintext data key so it could decrypt the volume. AWS KMS denied the request because the AWS KMS key that was used to encrypt the data key that the EBS volume was encrypted with was disabled.

  Next, you re-enable the AWS KMS key and re-attach the EBS volume.

To re-enable the MyKMSKey, return to the AWS KMS console.

Select  MyKMSKey, and then choose Key actions > Enable.

To attach the volume again, return to the Amazon EC2 console.

In the navigation pane, choose Volumes.

Select your volume, and then choose Actions > Attach volume.

On the Attach volume page, for Instance, choose your instance, and then choose Attach volume.

  The volume is attached successfully.

Task 5: Analyzing AWS KMS activity by using CloudTrail
In this task, you access the CloudTrail event history to find events that are related to your encryption operations. The CloudTrail audit log functionality provides an important security feature, and it's a good idea to monitor how AWS KMS keys are used in your account.

First, you access the CloudTrail event history.

Navigate to the CloudTrail console. 

In the navigation pane, choose Event history.

  In the Event history list, notice the column named Event source.

  Each time an API call to an AWS service occurs within the Region that you have selected, if that service reports such events to CloudTrail, then the event and the AWS service that reported the event are listed.

  Next, you filter the event history to display only events that the AWS KMS service reported. 

Under Lookup attributes, from the dropdown list that currently displays Read-only, choose Event source.

In the search box that currently reads Enter an event source, enter kms, and then choose kms.amazonaws.com.

  Next, you analyze a few key CloudTrail events.

Choose the link for the CreateGrant event name.

  In the Event record section, review the details of the event. The details are similar to the following image. In this image, the essential details of the event record are highlighted.

  CreateGrant event record. Highlighted information includes principalId, eventName, keyId, and EBS object name.

  Analysis: This event allows AWS principals to use AWS KMS keys in cryptographic operations. The EC2 instance sends a CreateGrant request so that it can decrypt the data key.

Return to the Event history list, and choose the link for the Decrypt event name. If there are multiple Decrypt events, pick any one of them.

   Note: If you do not see the Decrypt event, wait for a few minutes.

  In the Event record section, review the details of the event. The details are similar to the following image. In this image, the essential details of the event record are highlighted.

  Decrypt event record. Highlighted information includes principalId, eventName, keyId, and EBS object name.

  Analysis: Decrypt follows a successful CreateGrant event. The record shows the details of the AWS KMS key named MyKMSKey that the EC2 instance uses to decrypt the encrypted key, which is eventually used to decrypt data on the EBS volume.

Return to the Event history list, choose the links for the following events, and observe the event details.

For the GenerateDataKeyWithoutPlaintext, an event occurred when a request to generate a key to encrypt the EBS volume data was sent to AWS KMS.

You might also look at events such as RetireGrant.

CloudTrail logs all AWS KMS API activity. By evaluating these log entries, you might be able to determine the past usage of a particular AWS KMS key. If you want to be able to analyze events that occurred more than 90 days ago, you can create a CloudTrail trail.

Task 6: Reviewing key rotation
You might be required to rotate your KMS keys due to business or contract rules or government regulations. AWS KMS supports automatic key rotation only for symmetric encryption KMS keys with key material that AWS KMS creates. Automatic rotation is optional for customer managed KMS keys.

In this task, you review the feature to enable automatic rotation for the key you created in this lab. However, you cannot see it in effect as the rotation would be effective after a year, once enabled.

In the AWS Management Console, choose Key Management Service to open the AWS KMS console.

From the navigation pane on the left, choose Customer managed keys  .

From the list, choose MyKMSKey, the key you created for this lab.

Choose Key rotation.

On the Key rotation menu, select Automatically rotate this KMS key every year.

Choose Save.

A message is displayed at the top indicating Key rotation updated.

Your key is now scheduled for rotation.

Conclusion
Congratulations! You now have successfully done the following:

Reviewed the default encryption provided by Amazon S3

Accessed the encrypted Amazon S3 object

Created an AWS KMS customer managed key to encrypt and decrypt data at rest

Created and attached an encrypted data volume on an existing EC2 instance

Disabled and re-enabled an AWS KMS key and observe the effects on data access.

Monitored encryption key usage by using CloudTrail event history.

Reviewed key rotation.

Submitting your work
To record your progress, at the top of these instructions, choose Submit.

When prompted, choose Yes.

After a couple of minutes, the Grades panel appears and shows you how many points you earned for each task. If the results don't display after a couple of minutes, choose Grades at the top of these instructions.

 Tip: You can submit your work multiple times. After you change your work, choose Submit again. Your last submission is recorded for this lab.

To find detailed feedback about your work, choose Submission Report.

Lab complete
Congratulations! You have completed the lab.

At the top of this page, choose  End Lab, and then choose Yes to confirm that you want to end the lab.

The following message appears: Ended AWS Lab Successfully

© 2023, Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.