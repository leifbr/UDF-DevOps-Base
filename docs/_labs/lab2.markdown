---
name: Lab2
title: Lab2 - Jinja2 Templates
description: Demonstrates the ability of Jinja2 to create dynamic AS3 declarations.  While this lab uses the F5-CLI, the same logic applies when using Jinaj2 with Ansible. 
layout: page
tags: 
    - f5-cli
    - AS3
    - jinja2
---
## Overview

In this lab, we will leverage [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/), a very powerful template engine, to generate dynamic AS3 declarations.  A few advantages dynamic template offer you and your team:

* Code reuse 
* simplified logic
* makes it easier to add new types of deployments

Ideally, in a production environment you will want to use a template engine, like Jinja2.

## Prerequisites

1. Access to the F5 Unified Demo Framework (UDF)
2. Chrome browser
3. Finished the Lab0 setup process in the [UDF DevOps Base documentation][UDF DevOps Base documentation]

## Environment

In this lab we will deploy an HTTP application on the F5 BIG-IP.  This lab will
leverage the following components:

* [F5 BIG-IP][F5 BIG-IP]
* [F5 Automation Toolchain][F5 Automation Toolchain]
* [F5 CLI][F5 CLI]
* [Jinja2 CLI][Jinja2 CLI]
* [InSpec][InSpec]

## Setup
Ensure you havce the latest version of the lab from GitHub then open the lab2 folder:

    cd ~/projects/UDF-DevOps-Base
    git pull
    cd labs/lab2

## Create AS3 Declaration

1. Create a YAML data file called lab2a.yml in the lab2 directory:
    ~/projects/UDF-DevOps-Base/labs/lab2/lab2a.yml

        virtualAddresses: 10.1.20.20
        serverAddresses: 10.1.10.5

2. Open the lab2a.as3.j2 template file and examine it.
    
* it should look familiar to the declaration from lab1 
* simple declaration

3. Create the AS3 declaration using Jinja2:

        jinja2 lab2a.as3.j2 lab2a.yml > lab2a.as3.json

4. Open the new AS3 declaration in VS Code and examine it:

* similar to the declaration from lab1 

        diff lab2a.as3.json ../lab1/http.as3.json

* static assignment of virtual server and pool members

Congratulations, you have created your Jinja2 template and AS3 declaration!  However, you have undoubtably noticed that this declaration does not scale or work well in a production environment.  Let's fix that!

## Looping over Variables

To make our Jinja2 template a little more useful, we are going to introduce [Jinja filters](https://jinja.palletsprojects.com/en/2.11.x/templates/#filters) so we can add multiple virtual addresses and multiple pool members.

1. Create a new yaml data file called lab2b.yml:

        virtualAddresses:
          - 10.1.20.20
        serverAddresses:
          - 10.1.10.5
          - 10.1.20.10

2. Create the AS3 declaration using Jinja2

        jinja2 lab2b.as3.j2 lab2b.yml > lab2b.as3.json

3. Open the AS3 declaration and examine it:

* Virtual addresses in the declaration now support the correct data type based on the AS3 schema
* Pool members in the declaration now support the correct data type based on the AS3 schema
* Pool members can now be dynamically added and removed

## Multiple Applications
Now we will extend the Jinja2 template to support multiple applications under the demo tenant.

1. Create a new yaml data file called lab2c.yml:

        apps:
          - name: test1.f5demos.com
            virtualAddresses:
              - 10.1.20.20
            virtualPort: 80
            servicePort: 80
            serverAddresses:
              - 10.1.10.5
              - 10.1.10.10
          - name: test2.f5demos.com
            virtualAddresses:
              - 10.1.20.20
            virtualPort: 8080
            servicePort: 8080
            serverAddresses:
              - 10.1.10.5
              - 10.1.10.10

2. Create the AS3 declaration using Jinja2

        jinja2 lab2c.as3.j2 lab2c.yml > lab2c.as3.json

3. Open the AS3 declaration and examine it:

* You should see multiple applications under the demo tenant
        

## Issue AS3 Declaration to BIG-IP

1. Set the BIG-IP password as an environment variable:

    > **NOTE**: The BIG-IP password can be found on the BIG-IP1 and BIG-IP2 documentation pages inside the UDF deployment

        export bigip_pwd=replaceme

2. login to the BIG-IP (using the F5-CLI container)

    > **Note**: If the F5-CLI container is not running, start is with the 'docker start f5-cli' command 
        
        docker exec -it f5-cli f5 login --authentication-provider bigip --host 10.1.1.6 --user admin --password $bigip_pwd

3. Verify the Application Service 3 Extension is installed

        docker exec -it f5-cli f5 bigip extension as3 verify

4. Issue AS3 Declaration

        docker exec -it f5-cli f5 bigip extension as3 create --declaration /f5-cli/projects/UDF-DevOps-Base/labs/lab2/lab2c.as3.json

## Testing

For testing we will use Chef InSpec_.
This tool is commonly used in automated deployments and offers
a wide variaty of both infrastructure and application testing options.

Test that the deployment was successful:

  > **NOTE**: If you are still in the F5 CLI container, you will need to either exit or open a new terminal

    cd ~/projects/UDF-DevOps-Base/labs/lab2
    inspec exec test/app

## Cleanup

To cleanup the lab we need to remove the AS3 declaration deployed to the BIG-IP.  Run the following command in the f5-cli docker container

    docker exec -it f5-cli f5 bigip extension as3 delete --auto-approve


[F5 CLI]: https://clouddocs.f5.com/sdk/f5-cli/
[UDF DevOps Base documentation]: https://udf-devops-base.readthedocs.io/en/latest/
[F5 BIG-IP]: https://www.f5.com/products/big-ip-services/virtual-editions
[F5 Automation Toolchain]: https://www.f5.com/products/automation-and-orchestration
[InSpec]: https://www.inspec.io/
[Jinja2 CLI]: https://pypi.org/project/jinja2-cli/