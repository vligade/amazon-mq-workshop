# Lab 6: Tighten up security with access control

In this lab you will tighten up the security configuration with access control policies, so that you control which user can send/receive messages from which queues/topics.

1\. Navigate to the Amazon MQ Brokers page.
<details><summary>Screenshot step 1</summary><p>

![Amazon MQ workshop lab 6 step 1](/images/amazon-mq-broker-overview.png)

</p></details><p/>


2\. Click on the name of the broker, you created in lab 1.
<details><summary>Screenshot step 2</summary><p>

![Amazon MQ workshop lab 6 step 2](/images/security-set-up-Step2.png)

</p></details><p/>


3\. Go to the bottom of the page and click **Create user**.
<details><summary>Screenshot step 3</summary><p>

![Amazon MQ workshop lab 6 step 3](/images/security-set-up-Step3.png)

</p></details><p/>


4a\. Create two users. One with the name and group **user1** and a second one with the name and group **user2**.
<details><summary>Screenshot step 4a</summary><p>

![Amazon MQ workshop lab 6 step 4a](/images/security-set-up-Step4a.png)

</p></details><p/>


4b\. Update the user we have created during the broker set-up and apply the group **admin**.
<details><summary>Screenshot step 4b</summary><p>

![Amazon MQ workshop lab 6 step 4b](/images/security-set-up-Step4b.png)

</p></details><p/>


5\. To apply these changes imediately, you have to reboot your broker. Go to the top of the page and choose **Actions** and select **Reboot broker**.
<details><summary>Screenshot step 5</summary><p>

![Amazon MQ workshop lab 6 step 5](/images/security-set-up-Step5.png)

</p></details><p/>


6\. It takes a few minutes until the broker finishes the reboot and the status of these **Pending modifications** are not **Creation** anymore.
<details><summary>Screenshot step 6</summary><p>

![Amazon MQ workshop lab 6 step 6](/images/security-set-up-Step6.png)

</p></details><p/>


7\. Go to the top of the page and click **Edit**.
<details><summary>Screenshot step 7</summary><p>

![Amazon MQ workshop lab 6 step 7](/images/security-set-up-Step7.png)

</p></details><p/>


8\. To view and edit the latest configuration, just click the **View** link.
<details><summary>Screenshot step 8</summary><p>

![Amazon MQ workshop lab 6 step 8](/images/security-set-up-Step8.png)

</p></details><p/>


9\. On this page, you see the XML configuration of Active MQ. Click **Edit configuration** button in the top right corner.
<details><summary>Screenshot step 9</summary><p>

![Amazon MQ workshop lab 6 step 9](/images/security-set-up-Step9.png)

</p></details><p/>


10\. Scroll a bit down to the element **\<authorizationPlugin\>** and make the following changes. Afterwards click **Save** and confirm, to store the configuration changes.

```
    <authorizationPlugin>
      <map>
        <authorizationMap>
          <authorizationEntries>
            <authorizationEntry admin="admin" queue="&gt;" read="admin" write="admin"/>
            <authorizationEntry admin="admin" topic="&gt;" read="admin" write="admin"/>
            <authorizationEntry admin="admin,user1" queue="queue.user1" read="user1" write="user1"/>
            <authorizationEntry admin="admin,user2" read="user2" topic="topic.user2" write="user2"/>
            <authorizationEntry admin="admin,user1,user2" read="admin,user1,user2" topic="ActiveMQ.Advisory.&gt;" write="admin,user1,user2"/>
          </authorizationEntries>
          <tempDestinationAuthorizationEntry>
            <tempDestinationAuthorizationEntry admin="tempDestinationAdmins" read="tempDestinationAdmins" write="tempDestinationAdmins"/>
          </tempDestinationAuthorizationEntry>
        </authorizationMap>
      </map>
    </authorizationPlugin>
```


<details><summary>Screenshot step 10</summary><p>

![Amazon MQ workshop lab 6 step 10](/images/security-set-up-Step10.png)

</p></details><p/>


11\. Close this browser-tab and go back to the Edit broker page. From the **Revision** drop down select the new revision you created, and click **Schedule modifications**. On the next page, select **Immediately** and click on **Apply**.
<details><summary>Screenshot step 11</summary><p>

![Amazon MQ workshop lab 6 step 11](/images/security-set-up-Step11.png)

</p></details><p/>


12\. After the broker is again in the status **Running**, run the following command in an SSH session on your EC2 instance (first replace the parameter **<....>** with the value you have chosen):

```
java -jar amazon-mq-client.jar -url $url -user user2 -password <user 2 password> -mode sender -type queue -destination queue.user1 -name user1
```

You should see a log output like the following one:

```
[ActiveMQ Task-1] INFO org.apache.activemq.transport.failover.FailoverTransport - Successfully connected to ssl://b-4e4bfd69-7b83-4a27-9faf-4684cfa80443-2.mq.eu-central-1.amazonaws.com:61617
Error: User user2 is not authorized to write to: queue://queue.user1
```

This indicates, the **user2** is not authorized to write into this queue. Try out what happens, if you read messages from this queue or read/write from/to another one.
When you only change the user from **user2** to **user1** and read from **queue.user1**, you should see the following log output which indicates, that you can successfully write to **queue.user1**.

```
[ActiveMQ Task-1] INFO org.apache.activemq.transport.failover.FailoverTransport - Successfully connected to ssl://b-4e4bfd69-7b83-4a27-9faf-4684cfa80443-2.mq.eu-central-1.amazonaws.com:61617
12.04.2018 18:49:43.511 - Sender: sent '[queue://workshop.user1] [user1] Message number 1'
12.04.2018 18:49:43.543 - Sender: sent '[queue://workshop.user1] [user1] Message number 2'
12.04.2018 18:49:43.567 - Sender: sent '[queue://workshop.user1] [user1] Message number 3'
...
```


13\. Stop the sender by holding **CTRL + c** in the SSH session.