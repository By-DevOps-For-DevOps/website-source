---
title: CloudFormation
date: 2017-01-03 14:45:56
git: https://github.com/microservices-today/website-source
---
### Overview
This post explains AWS CloudFormation concepts. 
AWS CloudFormation provides an easy way to create and manage a collection of AWS resources. This allows us to version control our AWS infrastructure. You only pay for the resources you create.

![CloudFormation Components](/images/cloudformation/CloudFormation_Components.png)

### CloudFormation Template

We use CloudFormation Templates to create service or application architectures and have CloudFormation use those templates for quick and reliable provisioning of the services or applications.
These are formatted text files in JSON or YAML. These templates describe the resources that you want to provision in your AWS CloudFormation stacks. 

![Parameters](/images/cloudformation/Template1.png)
![Resources](/images/cloudformation/Template2.png)
![Outputs](/images/cloudformation/Template3.png)
![Template YAML](/images/cloudformation/Template_YAML.png)

### AWS CloudFromation Designer

AWS CloudFormation Designer is a graphic tool for creating, viewing, and modifying AWS CloudFormation templates.

The following figure illustrates the Designer panes and its main components.
![Designer Panes](/images/cloudformation/Designer.png)

 a. Canvas Pane: Displays template resources as diagram.
 b. Resource types pane: Lists all of the template resources that you can add to your template.
 c. JSON Editor: Here you specify the details of your template, such as resource properties or template parameters
 d. Errors Pane: Errors pane displays validation errors.
 e. Full screen and Split screen buttons: Buttons to select different views of Designer.
 f. Fit to window button: A button that resizes the canvas pane to fit your template's diagram.
 g. Toolbar: Provides quick access to commands for common actions, such as opening and saving templates.


### CloudFormation Framework

Framework, creates, updates, deletes stack 
The AWS CloudFormation framework creates, updates and deletes CloudFormation Stacks. It provisions and configures resources by making calls to AWS resources that are described in your template. If stack creation fails, AWS CloudFormation rolls back your changes by deleting the resources that it created.

##### Getting Started

a. Pick a template 
	You will need a template that specifies the resources that you want in your stack. You can create them either using Designer, or you can upload your template file created locally to S3 Bucket.
b. Make sure your template is valid and have all the required dependent resources required for your stack.
c. Create Stack
	- Sign in to the AWS Management Console and open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation/.
	- If this is a new AWS CloudFormation account, click Create New Stack. Otherwise, click Create Stack.
	- In the Template section, you can either design your template or choose a template (local file or S3 URL).
	- In the Specify Details section, enter the stack name. The stack name cannot contain spaces.
	- In the Parameters Section, specify all the required parameters.
	- Click Next
	- If required, you can add some tags to identify your stack.
	- Review the information for the stack. When you're satisfied with the settings, click Create.
d. Monitor the progress of the stack creation.
	To view the events for the stack, select your stack and in the Stack Details pane, click the Events tab.
e. Use your stack resources.
	When the stack has a status of CREATE_COMPLETE, AWS CloudFormation has finished creating the stack, and you can start using its resources.
f. Clean Up
	To delete the stack and its resources, from the AWS CloudFormation console, select the stack and click Delete.
	In the same way you monitored the creation of the stack, you can monitor its deletion by using the Event tab.



