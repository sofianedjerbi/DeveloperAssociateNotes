# Security

## KMS Precisions

To **encrypt over 4KB** you need **envelope encryption** with the `GenerateDataKey` API.  
It uses a temporary data key to encrypt data **client side**.  
The client send back the encrypted file with an encrypted key.  
Data key caching is available.  

To encrypt it later you can use `GenerateDataKeyWithoutPlaintext`

## KMS Limits

Each cryptographic operations **share a quota**.  
You can **raise your limits with an API call or AWS support**.

## KMS In Lambda

You can **encrypt env variables** and decrypt them with:

```py
ENCRYPTED = os.environ['DB_PASSWORD']
# Decrypt code should run once and variables stored outside of the function
# handler so that these are decrypted once per container
DECRYPTED = boto3.client('kms').decrypt(
    CiphertextBlob=b64decode(ENCRYPTED),
    EncryptionContext={'LambdaFunctionName': os.environ['AWS_LAMBDA_FUNCTION_NAME']}
)['Plaintext'].decode('utf-8')
```

## S3 Bucket Key for SSE-KMS

Reduce the cost by using a **S3 Bucket Key** used to encrypt the data in S3.

## Key Policies

Can define **who can access KMS keys**.  
Supports **roles, users, assumed roles and federated users**.

## CloudHSM Precisions

Support **symetric and asymmetric encryption**.  
They are spread cross AZs.  
Can be used as a **KMS custom keystore**.

## SSM Parameter Store Example

```py
def lambda_handler(event, context):
    db_url = ssm.get_parameters(Names=["/my-app/dev/db-url"])
    print(db_url)
    db_password = ssm.get_parameters(Names=["/my-app/dev/db-password"], WithDecryption=True)
    print(db_password)
    return "worked!"
```

## CloudFormation SSM and SecretsManager integration

We can retrieve values from **SSM Parameter Store** and **Secrets Manager**.  
`ssm` for plaintext, `ssm-secure` and `secretsmanager` for secret values in Secrets Manager.  

Syntax: 
- `{{resolve:ssm(-secure):key:version}}` for SSM
- `{{resolve:secretsmanager:secret-id:secret-string:json-key:version-stage:version-id}}` for SSM  

## CloudFormation RDS

### Solution 1: MasterUserPassword

Create **admin secret implicitly**.  
Rotation is handles by RDS, Aurora.

```yaml
Resources:
  MyCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      Engine: aurora-mysql
      MasterUsername: masteruser
      ManageMasterUserPassword: true

Outputs:
  Secret:
    Value: !GetAtt MyCluster.MasterUserSecret.SecretArn
```

### Solution 2: Dynamic Reference

**Generate** secret **and reference** it.

```yaml
Resources:
  MyDatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Name: MyDatabaseSecret
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: "password"
        PasswordLength: 16
        ExcludeCharacters: "\"@/\""
 
  MyDBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBName: mydatabase
      AllocatedStorage: 20
      DBInstanceClass: db.t2.micro
      Engine: mysql
      MasterUsername: '{{resolve:secretsmanager:MyDatabaseSecret:SecretString:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:MyDatabaseSecret:SecretString:password}}'

  # For rotation:
  SecretRDSAttachment:
  Type: AWS::SecretsManager::SecretTargetAttachment
  Properties:
    SecretId: !Ref MyDatabaseSecret
    TargetId: !Ref MyDBInstance
    TargetType: AWS::RDS::DBInstance
```

## CloudWatch Logs Encryption

You can enable encryption with **KMS** at **log group level**.  
CloudWatch API got `associate-kms-key` and `create-log-group`.  

*Not available in the console.*

## CodeBuild Security

Use **env variables to reference store parameters / secrets**.

## AWS Nitro Enclaves

Process **sensitive data** in an **isolated compute environment**.  
Not a container, not persistent storage, no interactive acess, no external networking.

*Use case: Processing credit cards, secure multi-party computation*



