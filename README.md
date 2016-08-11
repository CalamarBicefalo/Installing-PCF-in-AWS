# Installing-PCF-in-AWS
Guide detailing how to install a PCF instance on top of a raw account of AWS


1. Base installation with [cfawsinit](https://github.com/mandarjog/cfawsinit)
1. Installing [AWS services broker](https://docs.pivotal.io/aws-services/index.html)
1. Installing [Data Dog tile]() for monitoring the infrastructure
1. Installing [PCF Metrics]() for monitoring individual apps



## Preparing AWS
At this point it is assumed a AWS account is ready to be used.

Before getting started, setting up Cloud Trail will allow us to monitor API calls, it is useful to enable 
Cloud Trail before proceeding any further.

An IAM user for pcf needs to be created, this will provide us with the access key and secret access key.
Additionally, we will need to generate a key pair in EC2.
Since the user is going to perform cloudformation, it is easier to run the process with admin privileges.

The installation process requires that an SSL certificate for the domain is
[uploaded to AWS](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs_manage.html#UploadSignedCert).
It is important to keep the certificate identifier since it will be required later
(or it can be retrieved via the [api](http://docs.aws.amazon.com/cli/latest/reference/iam/list-server-certificates.html).


## Executing CloudFormation (using cfawsinit)
In order to set up PCF, the first step is to make sure the AWS infrastructure is ready,
for this task we will use [AWS cloudformation](https://aws.amazon.com/cloudformation/) and
we will run the [provided cloudformation](/pcf_1_7_cloudformation_singlefile.json) template together with further installation steps in an 
automatic way using [cfawsinit](https://github.com/mandarjog/cfawsinit).

To get cfawsinit ready to get started, we need to provide it with a yaml file
specifying the connection parameters among others, in particular:
```yml
region: aws region (e.g. eu-west-1)
email: a reference email address (e.g. example@example.com)
ssh_private_key_path: the path to the private key that was set up in EC2 (e.g. ~/keys/mynewkey.pem)
ssh_key_name: the name of the above key as specified in EC2 (e.g. my-key)
domain: domain of the platform (e.g. example.com)
ssl_cert_arn: an ID of a certificate uploaded to AWS (e.g. arn:aws:iam::375783000519:server-certificate/mycertificate)
PIVNET_TOKEN: token from network.pivotal.io (can be seen https://network.pivotal.io/users/dashboard/edit-profile e.g. AAAA-h6BBBBBCotwXFi)
ops-manager:
    version: latest
    beta-ok: false
elastic-runtime:
    version: latest
    beta-ok: false
    cloudformation-template: pcf_1_7_cloudformation_singlefile.json
skip_cert_verify: true
```
A template [is provided](/awsdeploy-template.yml) for your convenience.

cfawsinit will proceed with the cloudformation procedure in first instance and will install 
and configure Ops Manager afterwards. No tiles will be set up with the current version.

As explained in [Installation Docs](http://docs.pivotal.io/pivotalcf/1-7/customizing/requirements.html#general) it is necessary to point a DNS domain, ideally a wildcard domain, at the load balancer. On AWS, the name/IP address of the load balancer is unknown until it is created. Therefore only after the cloudformation script created ELB you can log in to your DNS registrar and point *.domain to your ELB using a CNAME record as described [here](http://docs.pivotal.io/pivotalcf/1-7/customizing/cloudform-er-config.html#cname).
