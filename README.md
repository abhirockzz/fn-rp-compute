# Using Resource Principal Authentication provider with Oracle Functions

When a function you've deployed to Oracle Functions is running, it can access other Oracle Cloud
Infrastructure resources like Object Storage, Compute, Networking etc. You can use the OCI SDK for the specific service in your function business logic to achieve this, but the challenge is that in order to authenticate, the service SDK requires Oracle Cloud Infrastructure private keys. This is an insecure in terms of the overall architecute standpoint and cumbersome from a developer experience perspective.

Instead, you can use the Resource Principal authentication provider included in the Oracle Cloud Infrastructure SDK. To enable a function to access another Oracle Cloud Infrastructure resource, you have to include the function in a dynamic group, and then create a policy to grant the dynamic group access to that resource.

This example will take you through how to use Oracle Functions to list Compute Instances in a specific compartment within your OCI tenacy, using the Resource Principal authentication provider. Let's start by configurion Oracle Functions and deploying our function.

## Oracle Functions

Before you begin, please ensure that you have configure the Oracle Functions development environment. 

### Setup Fn CLI

Download (or update) the [Fn CLI](https://github.com/fnproject/cli)

`curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh`

### Switch to correct context**

- `fn use context <your context name>`
- Check using `fn ls apps`

### Deploy to Oracle Functions

Clone the Git repo and change to the correct directory

	git clone https://github.com/abhirockzz/fn-rp-compute
	cd fn-rp-compute

Create an application using the console or CLI.

	fn create app fn-rp-compute-app --annotation oracle.com/oci/subnetIds='SUBNET_OCIDs
        
        //example
        fn create app fn-rp-compute-app --annotation oracle.com/oci/subnetIds='["ocid1.subnet.oc1.phx.aaaaaaaaghmsma7mpqhqdhbgnby25u2zo4wqlrrcskvu7jg56dryxtfoobar"]' 

Deploy the application

        fn -v deploy --app fn-rp-compute-app

### Check the function OCID

To find the OCID of the function you just deployed, run the following command:

        fn inspect fn fn-rp-compute-app listinstances id

You should get the OCID - please copy it (except the `double quotes`) since you'll be using it in the next step.

## Functions Resource Principal configuration

### Dynamic Group

Add a dynamic group (e.g. `functions-dynamic-group`) with a rule to allow the function you just deployed to be added to the group

        resource.id='<FUNCTION_OCID>'
        
        //example
        resource.id='ocid1.fnfunc.oc1.phx.aaaaaaaaabzi5xfoobarm2gfwisjpnnrrx4c5j2gk6dp26qoinb647foobar'

### IAM Policy

Add a policy (e.g. `functions-list-instances-policy`) to allow listing instances in the specified compartment

        allow dynamic-group <DYNAMIC_GROUP_NAME> to read instances in compartment <COMPARTMENT_NAME>
        
        //example
        allow dynamic-group functions-dynamic-group to read instances in compartment mycompartment


## Test

Pass in the compartment OCID

        echo -n '<COMPARTMENT_OCID>' | fn invoke fn-rp-compute-app listinstances

        //example
        echo -n 'ocid1.compartment.oc1..foobaraaokbzj2jn3hf5kfoobarl2dq7u54p3tsmxrjd7s3uu7x23tfoobar' | fn invoke fn-rp-compute-app listinstances

You should get back a list of names of the compute instances

        ["dev-instance-1", "stage-instance-1", "prod-instance-1"]