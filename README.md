# Installing-PCF-in-AWS
Guide detailing how to install a PCF instance on top of a raw account of AWS
NOTE: A more detailed, and more granular installation process can be found 
in the [official documentation](http://docs.pivotal.io/pivotalcf/1-7/customizing/cloudform.html).

1. Base installation with [cfawsinit](https://github.com/mandarjog/cfawsinit)
1. Installing [AWS services broker](https://docs.pivotal.io/aws-services/index.html)
1. Installing [ELK tile]() for log gathering and querying
1. Installing [PCF Metrics]() for monitoring individual apps


## Preparing AWS
At this point it is assumed an AWS account is ready to be used.

Before getting started, setting up Cloud Trail will allow us to monitor API calls, it is useful to enable 
Cloud Trail before proceeding any further.

An IAM user for pcf needs to be created, this will provide us with the access key and secret access key.
Additionally, we will need to generate a key pair in EC2.
Since the user is going to perform cloudformation, it is easier to run the process with admin privileges.

Verify that EC2 instance limit is sufficient. The default is 20 instances which is barely enough
to run minimal PCF across 3 availability zones (AZ).


## Setting up the main domain
Before getting started it is important to get a domain with a valid certificate.
Self-signed certificates, although feasible, will cause issues at different points,
so it is strongly recommended to get a valid certificate.

In this documentation we assume that the domain is obtained from AWS Route53
and the certificate is registered in AWS Certificate Manager.
 
### Registering the certificate
Once the domain is ready, it is important to register several domains and subdomains
that will be required further during the installation process. Assuming your domain
is `example.com` you should register:

- `*.login.system.example.com`
- `*.uaa.system.example.com`
- `*.system.example.com`
- `*.apps.example.com`

Further details can be found in [the official documentation](http://docs.pivotal.io/spring-cloud-services/prerequisites.html)

It is useful to make note of the certificate identifier since it will be required in the next step
(it can alternatively be read from the Certificate Manager at any time).

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
ssl_cert_arn: an ID of a certificate created with CM (e.g. 	arn:aws:acm:eu-east-1:6333456:certificate/333-9447-4751-ae11-cd555d)
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

As explained in [Installation Docs](http://docs.pivotal.io/pivotalcf/1-7/customizing/requirements.html#general) 
it is necessary to point a DNS domain, ideally a wildcard domain, at the load balancer. On AWS, the name/IP address 
of the load balancer is unknown until it is created. Therefore only after the cloudformation script created ELB you 
can log in to Route53 and point *.domain to your ELB using a CNAME record as described 
[here](http://docs.pivotal.io/pivotalcf/1-7/customizing/cloudform-er-config.html#cname).

Note that the instance hosting Ops Manager has very generous permissions (e.g. IAM)
and it is publicly available on the internet by default.
In order to mitigate the risks you might enclose it in a private subnet or secure it by other means. 

### Troubleshooting

- **Errors while running smoke tests/errands:** tests are run from a virtual machine created in the private network.
  They involve communication with PCF API via the *cf* CLI. Make sure that the ELB is available from the private subnet
  (this may require to temporarily attach a NAT gateway to the private subnet). NOTE: The cloudformation script does this
  on purpose so you can decide how to enable that connectivity.
  
## Installing AWS services broker
Follow [the official documentation](https://docs.pivotal.io/aws-services/installation.html).
Please keep in mind the official documentation requires a user with, among others, IAM permission.
This permission is used to create new users and attach policies to it. That happens as part of service 
instantiation for some components such as dynamoDB.

## Installing PCF Metrics
Follow [the official documentation](http://docs.pivotal.io/pcf-metrics/).

## Installing ELK tile
Follow [the official documentation](https://docs.pivotal.io/logsearch/installing.html).
Once your ELK is ready, you might want to configure your ERT and orther tiles to ship
logs to logstash as described in the previous link, you might want to ship application
logs into logstash as described in [this documentation page](https://docs.run.pivotal.io/devguide/services/log-management.html)