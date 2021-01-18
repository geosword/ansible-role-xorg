# molecule-scaffold
A scaffold for new ansible roles tested with molecule

The main idea is making starting a new role easy including all various scripts and things I've created to make my life easier.

Scenario ```molecule/default/``` includes sample molecule files for docker (molecule.yml) and aws (molecule-ec2.yml) and some other standard files which might be needed for developing a role.

Read the comments in ```verify.yml``` and ```tasks/verify_*.yml``` to fully understand how to use it, and it's limitations

# Environment Variables
Use can use ```ANSIBLE_SKIP_TAGS molecule <command>``` to skip tags in your playbooks. Usefull when you want test a subset of changes without waiting for the whole role / prepare

# ec2 driver and aws
The ec2 driver requires boto, boto3 and a working installation of awscli (Not strictly true, but if the cli works, the python modules will too)
See ```.molecule/molecule/ec2scenario``` for an example scenario for the ec2 driver

## CICD
While you might be happy using your own aws credentials when developing / testing locally, adding your credenitals to a repo is a very bad idea. Instead create a new IAM user (no console access) and assign it to a group which has this inline policy attached:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "MoleculeMainlyAllowingStandingUpInstances",
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:TerminateInstances"
            ],
            "Resource": "arn:aws:ec2:*:*:*"
        },
        {
            "Sid": "MoleculeAllowToDynamicallyCreateAKeypairBasedOnWhatsInYourDotssh",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeKeyPairs",
                "ec2:CreateKeyPair",
                "ec2:DeleteKeyPair"
            ],
            "Resource": "*"
        },
        {
            "Sid": "MoleculeAllowMoleculeToCloneDatabasesForTestingPurposes",
            "Effect": "Allow",
            "Action": [
                "rds:RestoreDBClusterToPointInTime",
                "rds:CreateDBInstance",
                "rds:DeleteDBInstance",
                "rds:DeleteDBCluster"
            ],
            "Resource": [
                "arn:aws:rds:*:*:*:*"
            ]
        },
        {
            "Sid": "MoleculeAllowMoleculeAddRemoveSecurityGroups",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:security-group/*"
            ]
        },
        {
            "Sid": "MoleculeTopLevelStuffThatHasNoResourceOrNeedsResourceStar",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeSecurityGroups",
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup",
                "ec2:DescribeSubnets",
                "ec2:CreateTags",
                "ec2:DescribeTags"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

That will result in a user which has just enough privileges to stand up ec2 instances & associated operations, clone RDS instances, and tear it all down again.
