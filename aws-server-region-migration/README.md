# AWS Instance Migration Between Regions

This is a tutorial to replicate one server from one region to another. 

<span style="color:#ffc107">**WARNING:**</span>
This tutorial can incur in charges, thus ensure that all resources are cleaned up after completing it (delete/terminate all EBS Volumes, Elastic Network Interface, and all EC2 instances that could possible be created).

Before getting started, install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) if not already installed. Make sure to follow the [Configuration Basics](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) after instalation.

This tutorial was based on the video: [AWS Hands on lab - AWS MGN - Replicate server from one region to another](https://youtu.be/enz7kAB-ST4)

## 1. Create the stack

Create a key pair to enter with SSH connection into the machines. Copy the `KeyMaterial` to a file named `KurentoKey1.pem`. The parameter `--output text` makes it easier to copy, since the json format contains the new line character `\n` in the key material.

```
aws ec2 create-key-pair \
--key-name KurentoKey1 \
--region us-east-1
--output text
```
<details><summary>Response JSON example:</summary>

```json
{
    "KeyFingerprint": "0d:b5:9a:fb:30:d2:5a:db:90:1c:2c:07:f6:a3:46:2e:61:f5:df:f0",
    "KeyMaterial": "-----BEGIN RSA PRIVATE KEY-----\nMIIEogIBAAKCAQEArPMfo9StjWRNvMLKDgs1VYMm7hQJEpMfJss2OYxROdOWMTWU\nKawNy5F0SKS2+x+8HU7VOI+2zIr5g1qL9Yl8G4hO8r27QhhvzVesTt/wuUf4CUAY\nxQeN06DcrcrXc/EI0FlsjJnFWscVRp3z1nl+BMaoWWVKEkstlTPJAcBCYLiZBRnv\nkPr/K7oOSFP7ZTUcB856pfZMDb5sJYHTa9OYNB+kyTyI9k/WVK7Mk+2U8YFqFRAu\n4wR93TESS6wTdEAslZXKPN44xDUnoiPcCp+wzGcUGVlCHva/0IkVNvNMxu/Y1NjF\nUyozBFG2rwu2+KZkZZJrrgwULBsxvh/9oaL07wIDAQABAoIBACBhTHUx5mRoeR10\nIrxKYOvnqCw+2AwAO37Z4QcZPEPlV2NTMrAypEqOqwTGwfN2V5PtJIJ4dbMJ+fkb\nxMRtvyywcoeD+kE/rf72AS6rQriNvuSMeZa5+VW78lUfewMcB5hqhaY1S/vY9iTI\ngdSP3oYqY26JRjrylFAw59tKEaNl3VDMMt9idFuPcrrpX6cOp+3CdkrwFiUCrzrh\nGUTifGSzYh6wUnjVqIMPqLdd9Wzr9ijFGMMD2kv+YLayksUK4AFEONl1qOYWhZf7\nGUUkJTleCLfkCxLwqRqzvbaUEK//VAc+Gnf+4WKAFnNTmLNYThPxweIA8Rof6MR1\nD+62R6ECgYEA4kEV+hiLvmfcetdgOkCnev/G5GmlmosHCrO2lc4JrS9jpPCWJs66\n9RkoyB6EBsdq0NWJdiSo78YkAIBqNQivp7H75MWJp/ExAORnJM7KoqDjzfNNZF5D\n7fh26QwIKUScgF7Mgxt83uITPY6S7cVrKc7wnCsUki4hkCnaJxzmqYkCgYEAw6//\nCrf1VGsoAyOWMGFNbl164TD1FAx79ItIYRXYPuiH8p34sClU116nnOkcAjDnFwAd\nk8/phv/gyBSL0lO+0KntuZ3q/Kd26M5/Tt4VBDUKzdlVtEvocGGWkO8AhMZ3ezVd\n97s3deeLRjB+izb3RC47ihTfKEv8KG+OT54EpLcCgYBhYuSDxub3qRr04RmxWTz9\nq3S/wl2evLLmP16a8pmlqt04FLp8r8U3VIICSWhIxrNKem91o+f3dRDwClYsx7Vb\n+DdVTFWpLR8LpERlSoFcKOaFMnGgfxa8KpN8Ukp9AORgOO3MjRtdkG/5shG6OJyc\n3U8h2UU8epDFzc3xwfXjCQKBgA7bnhHCRe5S9IbIfO7PdIGct2fBv9n12LOIn8Y/\nUlv0a94QAIHfoYF4vmE7kdTYwbMNXzGJ58FITFjktnkRwrs1K8ecJetpC65Bf4kN\nc6sOG/PlPIyj9tIRls0KWI+8QfYo5ymYHW3mVrzNkc4gLkYO/JZPX2I/4rVvQu7o\noJGhAoGAd2mqyvYLzdqUZcZc2+W7cAdJnW4Z16wOh93em86zo1lSN7m4e+7SQ6Kq\n1CAGmR+s994aTEfCnPmdYV+ZU7rUbTy82Jc0xJgwpbiYFb9WwYBL2T2uBsLV3tSW\n/pPDc6cLEfKzCW0SZ4A8j/7On2EzGk5J1yKas+NSsRZoRJ4tSCU=\n-----END RSA PRIVATE KEY-----",
    "KeyName": "KurentoKey1",
    "KeyPairId": "key-0cedc87624b680911"
}
```
</details>
<details><summary>Response TEXT example:</summary>

```
e9:3c:25:79:2d:45:3e:c3:d0:3a:d3:c4:45:64:e2:ba:41:68:0d:2b     -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAjj+h82HrpmzUzAAD4XM/UqH8ns8ofjmMmSczhxq3xW4oQJR/
j/+D3Gd9foNYWvZa+Ymj+g97BzO0XijZoqWJ8MhrPv5Ml1Et0O6ylfjcDwTuZjQf
yaY0gS3wmmNSBpX0lkZvAe5Y1rEUCLsrBJfqh95pt8ImyN7jyYTGxIC6ZF4YxaAw
5qtaeyHKjLJRGwox8L8dunfIGlj8vJlhd6v7yVogshbDQE15Prwx29kRQJ3X3/aN
9bhKR8VeF4zrf/3aFVhCwpCE1gwMpO5eykRnXRO+7f1sw+o2aM+Yxa1d6S/iSv3V
+Nv4USRFVTmmZoWroS4dON5NvFhmA9aev9cWUQIDAQABAoIBAEvvahvLlQllpX81
Lt0mMZKCCKIaQiqHvCdxxs8Dk0CgmnpHC4WqSBDbaiGkKgF863m0YUW3B90DW/C7
69oodmeEBcQ96lMIseWL1ue6TnbyEfWbM7DoubSP+pXgGUSMvmGOPeoQ+3m4U9KJ
X+B23Gslbtf6m8u1OHI2IAl8EoRK6auJyMVGCnwAPoIh4zTkErEFZ4589+yGIvvf
j3S5tZT0CBRUnJz4YrCSRjbKesEbiPAAw9AoMl1GTYV9xiUHDOBkz1JRWVId00kW
V95/wU3uqPEGvKmH/08CJI2wgyzoVQoCEYAieC6BGq6Mep4tTTNtKbX/mvpFibhG
M3knggUCgYEA8QC2QQmj8WdZiTmCtFzLbbCfeiUfPDMt5GF/Ji9cidmSxaIg7uDs
iULj9g2bpbfwkqnUo/8t6Xb+58ibD+1i9lS3zJJKFauuR8SukY/p+HYUKgm/Vm5o
FLWY0GDwVJhtOr7WTorttJ0fPlLIcqqHpPuhHDxhPWiMaBdTl6e/qdcCgYEAlxm4
Kz5s2oiZfF3Qt6xBxRHAnFjkj/Xnuixa084Bd+ceI91INaOIe7eyoYZN+bMlISAU
d0F2qyZXHUsfdmkoYUlxZApZ8fQko5UUf1tqhpG07iE8T7ccT5ZqqMETlJ05RTEH
nxAuHS1a435J02kK5uBdlgEgHjlZEITRNXoFTBcCgYEAuV460LuBheRgUdQSXHgj
YDNB9skmxT74RXlYOx6ipsTax3h0rEgEp27zuAWvej+IosZN7/YOckw8tDrwScfl
HmL7HDZJFXH/kuQNAZkX9SudRthIe0pgc81ZxK1LLUYwfcbbP35MZ2eS6HT0CH5x
5sxEl1s5z+niVQ3cFgHIwfECgYANmSjz61hMioKVqLPB8+SvYZud9noLYqwKGzfJ
W/7R1aDmxWFcQm1kBPI0iyu1TyQGSSbOXLvbR0YLwtkABRK3Pq7PvAbAOaKafi4s
EAQUPES2MZOF8QrBNt4+dbHXbBbdXT48WG5t/cjyNElcl1V91a9Wcp97WNnlHK7f
Sy3y3QKBgQDEGKVlabeeIBx2yxuW/yDPF/sNUnFKRKqAiOwWrzqoFi6rgclXri4E
OExvCpkU7bqcitFmbTsd/HdB5mVISeWKUx0XoR4IgCxghhctstsqK4lNm1TijJ3M
o+39/SPhKypo01bfwHgZ4CTv4u6zrKWMiCB8qYxkfq0ObV/AzO3PRQ==
-----END RSA PRIVATE KEY-----   KurentoKey1      key-00c53317c9772174f
```
</details><br>

Create the kurento stack with a template URL
```
aws cloudformation create-stack \
--stack-name KurentoStack \
--template-url https://s3-eu-west-1.amazonaws.com/aws.kurento.org/KMS-Coturn-cfn-6.16.0.yaml \
--parameters ParameterKey=InstanceType,ParameterValue=t2.micro ParameterKey=KeyName,ParameterValue=KurentoKey1 \
--region us-east-1
```
<details><summary>Response example:</summary>

```json
{
    "StackId": "arn:aws:cloudformation:us-east-1:820726337684:stack/KurentoStack/b89d31f0-2839-11ed-9f0b-121a174b4a23"
}
```
</details><br>

This will create an instance that will be running a KMS. To test this, use the kurento client and provide the instance's public IP address in the following URL:
http://localhost:8443/index.html?ws_uri=ws://PUBLIC_IP:8888/kurento

To start the kurento client, simply run the folling commands on the `kurento-client-js/` folder to create the kurento client in a docker container.
```
docker build . -t kurento-client-js
docker run -d kurento-client-js -p 8443:8443
```

## 2. Initialize AWS MGN

```
aws mgn initialize-service --region eu-central-1
```

The next step creates a replication settings template in Frankfurt. 
Before executing it, make sure that there is no template already created in this particular region.
```
aws mgn create-replication-configuration-template \
--associate-default-security-group \
--bandwidth-throttling 0 \
--create-public-ip \
--data-plane-routing PUBLIC_IP \
--default-large-staging-disk-type GP3 \
--ebs-encryption DEFAULT \
--replication-server-instance-type t3.small \
--replication-servers-security-groups-ids '[]' \
--staging-area-subnet-id subnet-05eaea2abd73853f0 \
--staging-area-tags Name=ReplicationServer \
--no-use-dedicated-replication-server \
--region eu-central-1
```


## 3. Generate the required AWS credentials

```
aws iam create-user --user-name MGNUser
```
<details><summary>Response example:</summary>

```json
{
    "User": {
        "Path": "/",
        "UserName": "MGNUser",
        "UserId": "AIDA36FY2CCKO4I5IBXPQ",
        "Arn": "arn:aws:iam::820726337684:user/MGNUser",
        "CreateDate": "2022-08-30T13:19:21+00:00"
    }
}
```
</details><br>

```
aws iam attach-user-policy \
--user-name MGNUser \
--policy-arn arn:aws:iam::aws:policy/AWSApplicationMigrationAgentPolicy
```

```
aws iam create-access-key --user-name MGNUser
```
<details><summary>Response example:</summary>

```json
{
    "AccessKey": {
        "UserName": "MGNUser",
        "AccessKeyId": "AKIA36FY2CCKIQABIY5H",
        "Status": "Active",
        "SecretAccessKey": "0Xb0JaHv/fzFnJCV6zPM23jpKyBoOxpERmrhuqnR",
        "CreateDate": "2022-08-30T13:22:27+00:00"
    }
}
```
</details><br>


## 4. Install replication agent on source server

Connect to the instance created by the stack by SSH. To do so, use the previously created key saved in `KurentoKey1.pem`.

```
ssh -i "KurentoKey1.pem" ubuntu@<PUBLIC_IP>
```

Run this command if there is a rejection saying that the private key file is unproteced. This ensures the key is not publicly viewable.
```
chmod 400 KurentoKey1.pem
```

After connecting successfully into the instance, run the following commands to create the agent.

```
wget -O ./aws-replication-installer-init.py https://aws-application-migration-service-eu-central-1.s3.eu-central-1.amazonaws.com/latest/linux/aws-replication-installer-init.py
```

In this command, do not forget to change the access key and the secret according to the MGNUser credentials.
```
sudo python3 aws-replication-installer-init.py \
--region eu-central-1 \
--aws-access-key-id AKIA36FY2CCKIQABIY5H \
--aws-secret-access-key 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 \
--no-prompt
```

After these steps, the agent should be downloaded, installed and the server should be synced with AWS MGN.
Go to [Application Migration Service > Source servers](https://eu-central-1.console.aws.amazon.com/mgn/home?region=eu-central-1#/sourceServers), and wait for the Migration life cycle to be *Ready for testing* (or continue the optional step 5, since it does not depend on this).



## 5. Edit MGN Launch Settings and Template (optional)

Be aware that the default EC2 Launch Template has a c4.large as instance type. To reduce the costs, reducing this instance to a t2.micro is recommended.

Since we are in a different region, Frankfurt, we will need to create another key pair to authenticate into the replicated instances that will be created. 

Save the Key Material in `KurentoKey2.pem`.
```
aws ec2 create-key-pair \
--key-name KurentoKey2 \
--region eu-central-1 \
--output text
```

The following step requires the ID of the source server, which can be seen in the previous location provided. 

This ensures that AWS MGN will not change the intance type specified later in the launch template. 

Do not confuse the source server name with the ID. 
```
aws mgn update-launch-configuration \
--source-server-id s-13a5b49c3810fa449 \
--target-instance-type-right-sizing-method NONE \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "bootMode": "LEGACY_BIOS",
    "copyPrivateIp": false,
    "copyTags": false,
    "ec2LaunchTemplateID": "lt-098c8790e8e45c7f3",
    "launchDisposition": "STARTED",
    "licensing": {
        "osByol": true
    },
    "name": "Launch Configuration for Source Server s-13a5b49c3810fa449",
    "sourceServerID": "s-13a5b49c3810fa449",
    "targetInstanceTypeRightSizingMethod": "NONE"
}
```
</details><br>

TODO: create security group sg-02ace4f6edf5431df

Now, we can finally create the EC2 Launch Template.

Here we can use the `--client-token` parameter, where we can put any string with less than 64 chars. It is only meant to ensure [indempotency in AWS](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/Run_Instance_Idempotency.html). For this case it is not used.

The `--launch-template-id` is provided in the last step, but can also be found in the [EC2 > Launch templates](https://eu-central-1.console.aws.amazon.com/ec2/v2/home?region=eu-central-1#LaunchTemplates). Replace the ID in command below with it.
```
aws ec2 create-launch-template-version \
--launch-template-id lt-098c8790e8e45c7f3 \
--source-version 2 \
--launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"AssociatePublicIpAddress":true,"Groups":["sg-02ace4f6edf5431df"]}],"InstanceType":"t2.micro","KeyName":"KurentoKey2"}' \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "LaunchTemplateVersion": {
        "LaunchTemplateId": "lt-098c8790e8e45c7f3",
        "LaunchTemplateName": "created-and-used-by-application-migration-service-s-13a5b49c3810fa449-223108-004711",
        "VersionNumber": 3,
        "CreateTime": "2022-08-31T11:13:59+00:00",
        "CreatedBy": "arn:aws:iam::820726337684:user/admin",
        "DefaultVersion": false,
        "LaunchTemplateData": {
            "BlockDeviceMappings": [
                {
                    "DeviceName": "/dev/xvda",
                    "Ebs": {
                        "Iops": 400,
                        "VolumeSize": 8,
                        "VolumeType": "io1"
                    }
                }
            ],
            "NetworkInterfaces": [
                {
                    "AssociatePublicIpAddress": true,
                    "DeviceIndex": 0,
                    "Groups": [
                        "sg-02ace4f6edf5431df"
                    ]
                }
            ],
            "InstanceType": "t2.micro",
            "KeyName": "KurentoKey2",
            "TagSpecifications": [
                {
                    "ResourceType": "instance",
                    "Tags": [
                        {
                            "Key": "AWSApplicationMigrationServiceSourceServerID",
                            "Value": "s-13a5b49c3810fa449"
                        },
                        {
                            "Key": "Name",
                            "Value": "ip-172-31-22-187"
                        },
                        {
                            "Key": "AWSApplicationMigrationServiceManaged",
                            "Value": "mgn.amazonaws.com"
                        }
                    ]
                },
                {
                    "ResourceType": "volume",
                    "Tags": [
                        {
                            "Key": "AWSApplicationMigrationServiceSourceServerID",
                            "Value": "s-13a5b49c3810fa449"
                        },
                        {
                            "Key": "Name",
                            "Value": "ip-172-31-22-187"
                        },
                        {
                            "Key": "AWSApplicationMigrationServiceManaged",
                            "Value": "mgn.amazonaws.com"
                        }
                    ]
                }
            ]
        }
    }
}
```
</details><br>

Finally, we need to change the default version to the version of the template just created. 
Normally, in a fresh AWS account, the version should be 3, but it also provisioned in the last response.
```
aws ec2 modify-launch-template \
--launch-template-id lt-098c8790e8e45c7f3 \
--default-version 3 \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "LaunchTemplate": {
        "LaunchTemplateId": "lt-098c8790e8e45c7f3",
        "LaunchTemplateName": "created-and-used-by-application-migration-service-s-13a5b49c3810fa449-223108-004711",
        "CreateTime": "1970-01-01T00:00:00+00:00",
        "CreatedBy": "arn:aws:sts::820726337684:assumed-role/AWSServiceRoleForApplicationMigrationService/701cc13a-3de7-4d90-96cc-3943a41085ff",
        "DefaultVersionNumber": 3,
        "LatestVersionNumber": 3
    }
}
```
</details><br>

## 6. Launch Test Instance

Launches a test instance for the specific source server. This will change the migration lifecycle to *Test in progress* and will launch a **AWS Application Migration Service Conversion Server** to handle the instance migration conversion. Once the conversion is completed, a test instance should be available.
```
aws mgn start-test \
--source-server-ids s-13a5b49c3810fa449 \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "job": {
        "arn": "arn:aws:mgn:eu-central-1:820726337684:job/mgnjob-10cba81b2d2b1167a",
        "creationDateTime": "2022-08-31T14:33:26.983505+00:00",
        "initiatedBy": "START_TEST",
        "jobID": "mgnjob-10cba81b2d2b1167a",
        "participatingServers": [
            {
                "launchStatus": "PENDING",
                "sourceServerID": "s-13a5b49c3810fa449"
            }
        ],
        "status": "PENDING",
        "tags": {
            "resourceArn": "arn:aws:mgn:eu-central-1:820726337684:job/mgnjob-10cba81b2d2b1167a"
        },
        "type": "LAUNCH"
    }
}
```
</details><br>

Test this instance with the kurento client, as previously. Do not forget to change the public IP with the one from the test instance:
http://localhost:8443/index.html?ws_uri=ws://PUBLIC_IP:8888/kurento


## 7. Launch Cutover Instance

If everything is working, mark the migration as *Ready for cutover* with the following command.

```
aws mgn change-server-life-cycle-state \
--life-cycle state=READY_FOR_CUTOVER \
--source-server-id s-13a5b49c3810fa449 \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "arn": "arn:aws:mgn:eu-central-1:820726337684:source-server/s-13a5b49c3810fa449",
    "dataReplicationInfo": {
        "dataReplicationInitiation": {
            "nextAttemptDateTime": "2022-08-31T17:08:00+00:00",
            "startDateTime": "2022-08-31T00:48:19.613446+00:00",
            "steps": [
                {
                    "name": "WAIT",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "CREATE_SECURITY_GROUP",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "LAUNCH_REPLICATION_SERVER",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "BOOT_REPLICATION_SERVER",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "AUTHENTICATE_WITH_SERVICE",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "DOWNLOAD_REPLICATION_SOFTWARE",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "CREATE_STAGING_DISKS",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "ATTACH_STAGING_DISKS",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "PAIR_REPLICATION_SERVER_WITH_AGENT",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "CONNECT_AGENT_TO_REPLICATION_SERVER",
                    "status": "SUCCEEDED"
                },
                {
                    "name": "START_DATA_TRANSFER",
                    "status": "SUCCEEDED"
                }
            ]
        },
        "dataReplicationState": "CONTINUOUS",
        "lagDuration": "P0D",
        "lastSnapshotDateTime": "2022-08-31T16:42:49.446799+00:00",
        "replicatedDisks": [
            {
                "backloggedStorageBytes": 0,
                "deviceName": "/dev/xvda",
                "replicatedStorageBytes": 8589934592,
                "rescannedStorageBytes": 8589934592,
                "totalStorageBytes": 8589934592
            }
        ]
    },
    "isArchived": false,
    "launchedInstance": {
        "ec2InstanceID": "i-00dc9c253fa179036",
        "firstBoot": "SUCCEEDED",
        "jobID": "mgnjob-113f80bf2819a725e"
    },
    "lifeCycle": {
        "addedToServiceDateTime": "2022-08-31T00:47:10.686735+00:00",
        "elapsedReplicationDuration": "PT16H20M38.697305S",
        "firstByteDateTime": "2022-08-31T00:50:51.450542+00:00",
        "lastCutover": {
            "finalized": {},
            "initiated": {},
            "reverted": {}
        },
        "lastSeenByServiceDateTime": "2022-08-31T17:06:11.954571+00:00",
        "lastTest": {
            "finalized": {},
            "initiated": {
                "apiCallDateTime": "2022-08-31T15:42:30.904212+00:00",
                "jobID": "mgnjob-113f80bf2819a725e"
            },
            "reverted": {}
        },
        "state": "READY_FOR_CUTOVER"
    },
    "replicationType": "AGENT_BASED",
    "sourceProperties": {
        "cpus": [
            {
                "cores": 1,
                "modelName": "Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz"
            }
        ],
        "disks": [
            {
                "bytes": 8589934592,
                "deviceName": "/dev/xvda"
            }
        ],
        "identificationHints": {
            "awsInstanceID": "i-0982f5a2ad5c1a85c",
            "fqdn": "ip-172-31-22-187.ec2.internal",
            "hostname": "ip-172-31-22-187"
        },
        "lastUpdatedDateTime": "2022-08-31T16:48:56.355953+00:00",
        "networkInterfaces": [
            {
                "ips": [
                    "172.31.22.187"
                ],
                "isPrimary": true,
                "macAddress": "0A-40-F1-E8-B2-2D"
            }
        ],
        "os": {
            "fullString": "Linux version 4.4.0-1087-aws (buildd@lcy01-amd64-010) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) ) #98-Ubuntu SMP Wed Jun 26 05:50:53 UTC 2019"
        },
        "ramBytes": 1038729216,
        "recommendedInstanceType": "c4.large"
    },
    "sourceServerID": "s-13a5b49c3810fa449",
    "tags": {
        "resourceArn": "arn:aws:mgn:eu-central-1:820726337684:source-server/s-13a5b49c3810fa449"
    }
}
```
</details><br>

Only afterwards it is possible to launch the cutover instances. Executing this will set the source server's migration lifecycle with *Cutover in progress*. Then, it will terminate the test instance and start the cutover instance.
```
aws mgn start-cutover \
--source-server-ids s-13a5b49c3810fa449 \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "job": {
        "arn": "arn:aws:mgn:eu-central-1:820726337684:job/mgnjob-11d8c23ab1e3c9dc2",
        "creationDateTime": "2022-08-31T17:10:13.411702+00:00",
        "initiatedBy": "START_CUTOVER",
        "jobID": "mgnjob-11d8c23ab1e3c9dc2",
        "participatingServers": [
            {
                "launchStatus": "PENDING",
                "sourceServerID": "s-13a5b49c3810fa449"
            }
        ],
        "status": "PENDING",
        "tags": {
            "resourceArn": "arn:aws:mgn:eu-central-1:820726337684:job/mgnjob-11d8c23ab1e3c9dc2"
        },
        "type": "LAUNCH"
    }
}
```
</details><br>

After the cutover instance is up and running, test it with the cutover instance's public IP.

Now it is possible to finalize the cutover.
```
aws mgn finalize-cutover \
--source-server-id s-13a5b49c3810fa449 \
--region eu-central-1
```
<details><summary>Response example:</summary>

```json
{
    "arn": "arn:aws:mgn:eu-central-1:820726337684:source-server/s-13a5b49c3810fa449",
    "dataReplicationInfo": {
        "dataReplicationInitiation": {
            "nextAttemptDateTime": "2022-08-31T18:07:00+00:00",
            "startDateTime": "2022-08-31T18:06:24.362299+00:00",
            "steps": [
                {
                    "name": "WAIT",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "CREATE_SECURITY_GROUP",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "LAUNCH_REPLICATION_SERVER",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "BOOT_REPLICATION_SERVER",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "AUTHENTICATE_WITH_SERVICE",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "DOWNLOAD_REPLICATION_SOFTWARE",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "CREATE_STAGING_DISKS",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "ATTACH_STAGING_DISKS",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "PAIR_REPLICATION_SERVER_WITH_AGENT",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "CONNECT_AGENT_TO_REPLICATION_SERVER",
                    "status": "NOT_STARTED"
                },
                {
                    "name": "START_DATA_TRANSFER",
                    "status": "NOT_STARTED"
                }
            ]
        },
        "dataReplicationState": "DISCONNECTED",
        "replicatedDisks": [
            {
                "backloggedStorageBytes": 0,
                "deviceName": "/dev/xvda",
                "replicatedStorageBytes": 8589934592,
                "rescannedStorageBytes": 8589934592,
                "totalStorageBytes": 8589934592
            }
        ]
    },
    "isArchived": false,
    "launchedInstance": {
        "ec2InstanceID": "i-07e0559b366d0e404",
        "firstBoot": "SUCCEEDED",
        "jobID": "mgnjob-11d8c23ab1e3c9dc2"
    },
    "lifeCycle": {
        "addedToServiceDateTime": "2022-08-31T00:47:10.686735+00:00",
        "elapsedReplicationDuration": "PT17H19M13.655655S",
        "firstByteDateTime": "2022-08-31T00:50:51.450542+00:00",
        "lastCutover": {
            "finalized": {
                "apiCallDateTime": "2022-08-31T18:06:24.412320+00:00"
            },
            "initiated": {
                "apiCallDateTime": "2022-08-31T17:10:13.492019+00:00",
                "jobID": "mgnjob-11d8c23ab1e3c9dc2"
            },
            "reverted": {}
        },
        "lastSeenByServiceDateTime": "2022-08-31T18:05:29.535757+00:00",
        "lastTest": {
            "finalized": {},
            "initiated": {
                "apiCallDateTime": "2022-08-31T15:42:30.904212+00:00",
                "jobID": "mgnjob-113f80bf2819a725e"
            },
            "reverted": {}
        },
        "state": "CUTOVER"
    },
    "replicationType": "AGENT_BASED",
    "sourceProperties": {
        "cpus": [
            {
                "cores": 1,
                "modelName": "Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz"
            }
        ],
        "disks": [
            {
                "bytes": 8589934592,
                "deviceName": "/dev/xvda"
            }
        ],
        "identificationHints": {
            "awsInstanceID": "i-0982f5a2ad5c1a85c",
            "fqdn": "ip-172-31-22-187.ec2.internal",
            "hostname": "ip-172-31-22-187"
        },
        "lastUpdatedDateTime": "2022-08-31T17:48:58.323423+00:00",
        "networkInterfaces": [
            {
                "ips": [
                    "172.31.22.187"
                ],
                "isPrimary": true,
                "macAddress": "0A-40-F1-E8-B2-2D"
            }
        ],
        "os": {
            "fullString": "Linux version 4.4.0-1087-aws (buildd@lcy01-amd64-010) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) ) #98-Ubuntu SMP Wed Jun 26 05:50:53 UTC 2019"
        },
        "ramBytes": 1038729216,
        "recommendedInstanceType": "c4.large"
    },
    "sourceServerID": "s-13a5b49c3810fa449",
    "tags": {
        "resourceArn": "arn:aws:mgn:eu-central-1:820726337684:source-server/s-13a5b49c3810fa449"
    }
}
```
</details><br>

The lifecycle should now be set to *Cutover complete*, thus, the migration can be considered completed.
