# AWS SDK Extra

The AWS SDK + a handful of extra convenience methods.

```js
// require aws-sdk-extra, instead of the official aws-sdk
const aws = require(`@serverless/aws-sdk-extra`)

// initialize any service, as usual.
const s3 = new aws.S3({
  credentials: { accessKeyId: 'xxx', secretAccessKey: 'xxx' },
  region: 'us-east-1'
})

// initialize the Extra service for extra methods
const extra = new aws.Extra({
  credentials: { accessKeyId: 'xxx', secretAccessKey: 'xxx' },
  region: 'us-east-1'
})

// call some powerful extra methods. More info below.
const certificate = await extra.deployCertificate(params)
```

# Reference

- [deployDistributionDomain](#deployDistributionDomain)
- [deployCertificate](#deployCertificate)
- [deployDistributionDns](#deployDistributionDns)
- [addDomainToDistribution](#addDomainToDistribution)
- [getDomainHostedZoneId](#getDomainHostedZoneId)
- [deployRole](#deployRole)
- [removeRole](#removeRole)
- [removeRolePolicies](#removeRolePolicies)
- [deployLambda](#deployLambda)
- [deployApigDomainDns](#deployApigDomainDns)
- [deployAppSyncApi](#deployAppSyncApi)
- [deployAppSyncSchema](#deployAppSyncSchema)
- [deployAppSyncResolvers](#deployAppSyncResolvers)
- [deployStack](#deployStack)
- [removeStack](#removeStack)

# deployDistributionDomain

Deploys a CloudFront distribution domain by adding the domain to the distribution and deploying the certificate and DNS records.

```js
const params = {
  domain: 'serverless.com',
  distributionId: 'xxx'
}

const {
  certificateArn,
  certificateStatus,
  domainHostedZoneId
} = await extra.deployDistributionDomain(params)
```

# deployCertificate

Deploys a free ACM certificate for the given domain.

```js
const params = {
  domain: 'serverless.com'
}

const { certificateArn, certificateStatus, domainHostedZoneId } = await extra.deployCertificate(
  params
)
```

# deployDistributionDns

Deploys a DNS records for a distribution domain.

```js
const params = {
  domain: 'serverless.com',
  distributionUrl: 'xxx.cloudfront.net'
}

const { domainHostedZoneId } = await extra.deployDistributionDns(params)
```

# addDomainToDistribution

Adds a domain or subdomain to a CloudFront Distribution.

```js
const params = {
  domain: 'serverless.com',
  certificateArn: 'xxx:xxx',
  certificateStatus: 'ISSUED'
}

const { domainHostedZoneId } = await extra.addDomainToDistribution(params)
```

# getDomainHostedZoneId

Fetches the hosted zone id for the given domain.

```js
const params = {
  domain: 'serverless.com'
}

const { domainHostedZoneId } = await extra.getDomainHostedZoneId(params)
```

# deployRole

Updates or creates the given role name with the given service & policy. You can specify an inline policy:

```js
const params = {
  name: 'my-role',
  service: 'lambda.amazonaws.com',
  policy: [
    {
      Effect: 'Allow',
      Action: ['sts:AssumeRole'],
      Resource: '*'
    },
    {
      Effect: 'Allow',
      Action: ['logs:CreateLogGroup', 'logs:CreateLogStream'],
      Resource: '*'
    }
  ]
}
const { roleArn } = await extra.deployRole(params)
```

Or you can specify the policy as a maanged policy arn string:

```js
const params = {
  name: 'my-role',
  service: 'lambda.amazonaws.com',
  policy: 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'
}
const { roleArn } = await extra.deployRole(params)
```

If you don't specify a policy property, an admin policy will be created by default.

# removeRole

Removes the given role and all its attached managed and inline policies.

```js
const params = {
  name: 'my-role'
}

await extra.removeRole(params)
```

# removeRolePolicies

Removes all attached managed and inline policies for the given role.

```js
const params = {
  name: 'my-role'
}

await extra.removeRolePolicies(params)
```

# deployLambda

Updates a lambda if it exists, otherwise creates a new one.

```js
const params = {
  lambdaName: 'my-lambda', // required
  roleArn: 'aws:iam:role:arn:xxx', // required
  lambdaSrc: 'path/to/lambda/directory' // required. could also be a buffer of a zip file
  memory: 512 // optional, along with the other lambda config
  vpcConfig: // optional, specify a VPC
    securityGroupIds:
      - sg-xxx
    subnetIds:
      - subnet-xxx
      - subnet-xxx
}

const { lambdaArn, lambdaSize, lambdaSha } = await extra.deployLambda(params)
```

# deployApigDomainDns

Deploys the DNS records for an Api Gateway V2 HTTP custom domain

```js
const params = {
  domain: 'serverless.com', // required. The custom domain you'd like to configure.
  apigatewayHostedZoneId: 'qwertyuiop', // required. The regional hosted zone id of the APIG custom domain
  apigatewayDomainName: 'd-qwertyuiop.xxx.com' // required. The regional endpoint of the APIG custom domain
}

const { domainHostedZoneId } = await extra.deployApigDomainDns(params)
```

# deployAppSyncApi

Updates or creates an AppSync API

```js
const params = {
  apiName: 'my-api',
  apiId: 'xxx' // if provided, updates the API. If not provided, creates a new API
}

const { apiId, apiUrls } = await extra.deployAppSyncApi(params)
```

# deployAppSyncSchema

Updates or creates an AppSync Schema

```js
const params = {
  apiId: 'xxx', // the targeted api id
  schema: '...' // valid graphql schema
}

await extra.deployAppSyncApi(params)
```

# deployAppSyncResolvers

Updates or creates AppSync Resolvers

```js
const params = {
  apiId,
  roleName: 'my-role', // name of the role that provides access for these resources to the required resources
  resolvers: {
    Query: {
      getPost: {
        lambda: 'getPost' // name of the lambda function to use as a resolver for the getPost field
      }
    },
    Mutation: {
      putPost: {
        lambda: 'putPost'
      }
    }
  }
}

await extra.deployAppSyncResolvers(params)
```

# deployStack

Updates or creates a CloudFormation stack.

```js
const inputs = {
  stackName: 'my-stack', // required
  template: {
    // required
    AWSTemplateFormatVersion: '2010-09-09',
    Description: 'Example Stack',
    Resources: {
      LogGroup: {
        Type: 'AWS::Logs::LogGroup',
        Properties: {
          LogGroupName: '/log/group/one',
          RetentionInDays: 14
        }
      }
    },
    Outputs: {
      firstStackOutput: {
        Value: {
          'Fn::GetAtt': ['LogGroup', 'Arn']
        }
      },
      secondStackOutput: {
        Value: {
          'Fn::GetAtt': ['LogGroup', 'Arn']
        }
      }
    }
  },
  capabilities: ['CAPABILITY_IAM'],
  parameters: {
    firstParameter: 'value'
  },
  role: 'arn:iam:xxx'
}

const outputs = await extra.deployStack(params)
```

# removeStack

Removes a CloudFormation stack if it exists.

```js
const prams = {
  stackName: 'my-stack' // name of the stack you want to remove
}

await extra.removeStack(params)
```
