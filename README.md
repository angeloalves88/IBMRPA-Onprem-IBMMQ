# Installing IBM RPA On-premises 
# using the IBM MQ queue provider

IBM Robotic Process Automation - version 21.0.5

IBM Message Queue - version 9.2.5

Configuration performed on an IBM Cloud Classic VM

<p align="center">
   <img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DEVELOPMENT&color=RED&style=for-the-badge"/>
 <!--  <img src="http://img.shields.io/static/v1?label=STATUS&message=CONCLUIDO&color=GREEN&style=for-the-badge"/>-->
</p>

### Topics 

:small_blue_diamond: [Prerequisites](#prerequisites)

:small_blue_diamond: [Install IBM MQ](#install-ibm-mq)

:small_blue_diamond: [Configure IBM MQ](#configure-ibm-mq)

:small_blue_diamond: [MQ environment information](#mq-environment-information)

:small_blue_diamond: [Installing IBM RPA On-premises](#installing-ibm-rpa-on-premises)

:small_blue_diamond: [Testing access to IBM MQ](#testing-access-to-ibm-mq)


## Prerequisites

- Define the MQ access user
	- Local User on Windows
		- Create a new user
		- Add to MQM group
	- LDAP user
		- Follow this procedure: https://www.ibm.com/docs/en/ibm-mq/9.2?topic=mq-creating-setting-up-windows-domain-accounts
		
- In this tutorial, we will use a local account, if you use an LDAP account, inform your user instead of UserMQ

|USERNAME|PASSWORD|
| -------- |-------- |
|UserMQ|ibmrpa2022|


![image](https://user-images.githubusercontent.com/46223364/193288830-e1fee344-58f3-45bc-a8c3-db3287ac677d.png)


## Install IBM MQ

When searching for IBM Robotic Process Automation on Passport Advantage, you will have the IBM MQ file for Windows available for download.

Extract the files and start the Typical installation of IBM MQ (next, next, and finish).

At the end of the installation, another "Prepare IBM MQ Wizard" window will load to configure the access account to IBM MQ. In this Wizard, you will be asked to choose a user, local, or LDAP account.

- Enter 'No' (local user, our scenario)
- Enter 'Yes' for an LDAP account, and you will be asked for the domain, username, and password, to be configured in the IBM MQ service

This message will be displayed at the end if everything goes well with the user.
![image](https://user-images.githubusercontent.com/46223364/194894688-f5884c2e-4a16-4cc0-8516-b92c761e022d.png)


## Configure IBM MQ
<h5>[WINDOWS PROMPT]</h5>

The default port of IBM MQ is 1414 in the tutorial we will use 1515.

> **This script is different from the one documented

```

* Create the Queue Manager (QM)
	crtmqm RPA.QM

* Start QM
	STRMQM RPA.QM
	
* Access IBM MQ Prompt 
	RUNMQSC RPA.QM

* Stop Listeners
	STOP LISTENER('SYSTEM.DEFAULT.LISTENER.TCP') IGNSTATE(YES)

* Test Queue
	DEFINE QLOCAL('RPA.QUEUE.1') REPLACE
	DEFINE QLOCAL('RPA.DEAD.LETTER.QUEUE') REPLACE

* Use a different dead letter queue, for undeliverable messages
	ALTER QMGR DEADQ('RPA.DEAD.LETTER.QUEUE')

* Authentication Settings
	DEFINE AUTHINFO('RPA.AUTHINFO') AUTHTYPE(IDPWOS) CHCKCLNT(REQDADM) CHCKLOCL(OPTIONAL) ADOPTCTX(YES) REPLACE
	ALTER QMGR CONNAUTH('RPA.AUTHINFO')
	REFRESH SECURITY(*) TYPE(CONNAUTH)

* Create the channel
	DEFINE CHANNEL('RPA.CHANNEL') CHLTYPE(SVRCONN) MCAUSER('UserMq') REPLACE

* Channel authentication method configuration
	SET CHLAUTH('RPA.CHANNEL') TYPE(ADDRESSMAP) ADDRESS('*') USERSRC(CHANNEL) CHCKCLNT(REQUIRED) ACTION(REPLACE)
	ALTER QMGR CHLAUTH(DISABLED)

* Enabling listener on port 1515
	DEFINE LISTENER('RPA.LISTENER.TCP') TRPTYPE(TCP) PORT(1515) CONTROL(QMGR) REPLACE
	START LISTENER('RPA.LISTENER.TCP') IGNSTATE(YES)
  
  	exit
```

Assigning user access 
[WINDOWS PROMPT] 

```
* Configure user permission
	setmqaut -m RPA.QM -t qmgr -p UserMq +connect +inq
	setmqaut -m RPA.QM -n RPA.** -t queue -p UserMq +put +get +browse +inq +chg
	setmqaut -m RPA.QM -n SYSTEM.DEFAULT.** -t queue -p UserMq +dsp +chg

* Stop the QM
	ENDMQM RPA.QM
	
* Start the QM
	STRMQM RPA.QM
```

Test the queue with Put
[WINDOWS PROMPT] 

```

	set MQSERVER=RPA.CHANNEL/TCP/localhost(1515)
	set MQSAMP_USER_ID=UserMq
	echo %MQSERVER% %MQSAMP_USER_ID%

	cd C:\Program Files\IBM\MQ\tools\c\Samples\Bin64

	amqsputc RPA.QUEUE.1 RPA.QM
	Enter user password: ibmrpa2022
  	Enter messages to queue
  
```

![image](https://user-images.githubusercontent.com/46223364/193287723-83366818-db02-4a3c-a431-fe80031344b6.png)


## MQ environment information

The MQ access information

|Key|Value|
| -------- |-------- |
|Address|FQDN - Fully Qualified Domain Name|
|Port|1515|
|Queue Manager|RPA.QM|
|Channel|RPA.CHANNEL|
|User Id|UserMq|
|Password|ibmrpa2022|


## Installing IBM RPA On-premises

Perform the installation normally following the documentation

> https://www.ibm.com/docs/en/rpa/21.0?topic=server-install-by-using-installer

On the screen to choose the Message Provider to choose the IBM Message Queue

![image](https://user-images.githubusercontent.com/46223364/193291905-ec9a072c-fa9f-4068-ae7d-e463821b0af2.png)

Fill in the environment data and finish the installation

![image](https://user-images.githubusercontent.com/46223364/193291774-f96730ac-8f38-476d-b749-cf28247fde55.png)

> The installation validates the connection with MQ, if there is no message, everything is ok

## Testing access to IBM MQ

<h3>Connect IBM MQ command</h3>

Example script

```
  defVar --name success --type Boolean
  defVar --name conMQ --type QueueConnection
  defVar --name queue1 --type MessageQueue
  defVar --name quantity --type Numeric
  defVar --name item --type QueueMessage
  defVar --name successConnection --type Boolean
  connectIbmMQ --queueprovider "MQ-Local" --fromconfiguration  successConnection=success conMQ=value
  getQueue --connection ${conMQ} --name "RPA.QUEUE.1" success=success queue1=value
  enqueue --collection "${queue1}" --isserver  --priority 1 --value "test-123"
  count --collection "${queue1}" quantity=value
  return
```
![image](https://user-images.githubusercontent.com/46223364/194963124-c6e5d8d4-4ead-4b08-ab23-fbd1e0e6c849.png)


<h3>IBM RPA Systemic Provider</h3>

Create a queue in the Control center
![image](https://user-images.githubusercontent.com/46223364/193294275-dceea3af-1702-498d-9e08-46085261be2a.png)

Example script

```
  defVar --name successConnect --type Boolean
  defVar --name conMQ --type QueueConnection
  defVar --name queue1 --type MessageQueue
  defVar --name qtt --type Numeric
  defVar --name successGetQueue --type Boolean
  connectSystemMQ successConnect=success conMQ=value
  getQueue --connection ${conMQ} --fromconfiguration  --queue test1 successGetQueue=success queue1=value
  enqueue --collection "${queue1}" --isserver  --value t1
  count --collection "${queue1}" qtt=value
  return
```

![image](https://user-images.githubusercontent.com/46223364/193294522-e708fa9f-ef32-4baf-966d-3c8e459520aa.png)

IBM MQ Explorer

> We can visualize the queues created

![image](https://user-images.githubusercontent.com/46223364/193295168-0c367cce-0837-4abb-9954-4193d50bc166.png)

Finally, create a simple script and do a scheduling test to test access to the machine's queue.

Enjoy Yourself


... 

If you need to consult the IBM MQ log, see the file below

 - C:\ProgramData\IBM\MQ\qmgrs\RPA!QM\errors\AMQERR01.LOG
 
Support material:

- Create a Queue Manager	https://www.ibm.com/docs/en/ibm-mq/9.2?topic=interface-creating-queue-manager-called-qm1
- Configuring IBM MQ		https://www.ibm.com/docs/en/rpa/21.0?topic=server-optional-configuring-mq

