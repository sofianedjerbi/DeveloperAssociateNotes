# CloudFormation

## CloudFormation

You can't change a template, you should replace with **a new version**.  
CloudFront add **stack tag / resource tag**.

## Resource

They are **AWS components**.  
Type format: `service-provider::service-name::data-type-name`  
Type example: `AWS::EC2::Instance`

## Parameters

They are a way to **provite inputs** in templates, for reusability.  
Should be used if **something could be changed in the future**.  
They are **not just strings**. They can be int, values from SSM...

To use a parameter you should use `!Ref <name>`, also works with resources.

## Pseudo Parameters

Predefined parameters.  
Example: `AWS::AccountId`, `AWS::Region`



