# Create resources in AWS for a hosted cluster in Hypershift

This repo deploys a stack that creates/updates a custom resource that triggers a Go-based AWS Lambda function using AWS CDK. The Lambda function creates the infra and IAM resources in AWS that are required for creating a hosted cluster in hypershift. 
The stack's output JSON contains the ARNs that can be used to create a hosted cluster using the `hypershift create cluster` command.

The documentation here describes the resources that get created for infra and IAM: 
https://hypershift-docs.netlify.app/how-to/create-infra-iam-separately/

#### Lambda
Implemented as a GO Lambda function - The code is located in the `/hc-resources-lambda` folder. This function internally calls functions from hypershift's `github.com/openshift/hypershift/cmd/infra/aws` package to create the Infra and IAM resources in AWS.

This lambda is meant to work with a handwritten CF template `samples/lambda-cr-cfn.template`. The lambda function implements the cfn-response module to construct the responses accordingly to work with the CF template.

#### CF Template

See `samples/lambda-cr-cfn.template`.

#### To deploy a stack:
1. Prepare the deployment package for the lambda.
    - [Create a zip file](https://docs.aws.amazon.com/lambda/latest/dg/golang-package.html#golang-package-mac-linux) for the golang executable.
    - [Upload the zipfile](https://s3.console.aws.amazon.com/s3/upload/vnambiar-hypershift?region=us-east-1) to an S3 bucket.
2. In the CF template `samples/lambda-cr-cfn.template`, update the values for the name of the [S3 bucket and zipfile](https://github.com/stolostron/create-hc-resources/blob/lambda-cfn-response/samples/lambda-cr-cfn.template#L57-L58) holding the lambda deployment package from step 1.
3. On the AWS CloudFormation page, create a new stack using the CF template from the previous step providing values for the parameters.

#### Output:
```
{
  "infraOutput": {
      // Infra output ARNs from hypershift's create infra aws
  },
  "iamOutput": {
      // IAM output ARNs from hypershift's create iam aws
  }
}
```

#### Useful commands
*Run from within the `/hc-resources-cdk` folder*

`cdk synth` causes the resources defined in the app (lambda function in this case) to be translated into an AWS CloudFormation template
`cdk bootstrap` bootstrap cdk components into your AWS account/region
`cdk deploy` deploys this stack to your default AWS account/region
