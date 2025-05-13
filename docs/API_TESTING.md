# Testing Sample Rest API

## Getting Started

1. Open your API testing tool. (e.g. Postman, Insomnia, Bruno)
2. Go to [API Gateway](https://console.aws.amazon.com/apigateway).
3. You can get the API URL from API Gateway, use the PROD stage URL.

### Authentication

Before making API requests, you need to authenticate using Cognito:

**If you see an error like `Auth flow not enabled for this client`, you need to enable the ADMIN_USER_PASSWORD_AUTH flow for your Cognito app client:**

1. Go to the [Amazon Cognito Console](https://console.aws.amazon.com/cognito/).
2. Select your user pool.
3. Go to **App integration** > **App clients and analytics**.
4. Click on your app client.
5. Under **Authentication flows**, enable:
   - `ADMIN_USER_PASSWORD_AUTH` (for admin-initiated auth)
   - `USER_PASSWORD_AUTH` (for user-initiated auth, if needed)
6. Save changes.

Or, using the AWS CLI:
```sh
aws cognito-idp update-user-pool-client \
  --user-pool-id <your-user-pool-id> \
  --client-id <your-client-id> \
  --explicit-auth-flows ALLOW_ADMIN_USER_PASSWORD_AUTH ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH ALLOW_USER_SRP_AUTH ALLOW_CUSTOM_AUTH  
```

Then proceed with authentication:

1. Sign in to your Cognito User Pool:
   - Use the AWS Console or your application's login page
   - Or use the AWS CLI:
     ```bash
     aws cognito-idp admin-initiate-auth \
       --user-pool-id <your-user-pool-id> \
       --client-id <your-client-id> \
       --auth-flow ADMIN_USER_PASSWORD_AUTH \
       --auth-parameters USERNAME=<username>,PASSWORD=<password>
     ```

2. Get the ID token from the authentication response

3. Include the token in your API requests:
   - Add the Authorization header: `Authorization: Bearer <your-id-token>`

### Making API Requests

- **URL: {apiurl}/qna-agent**
- **Headers:**
  ```
  Content-Type: application/json
  Authorization: Bearer <your-id-token>
  ```
- **Body:**
  ```json
  {
      "sessionId": "",
      "message": "Hello",
      "metadata": {} 
  }
  ```

### Chat summary (Optional)

You can test chat summary construct using the same API Url.

- **URL: {apiurl}/chat-summary**
- **Headers:**
  ```
  Content-Type: application/json
  Authorization: Bearer <your-id-token>
  ```
- **Body:**
  ```json
  {
      "sessionId": "{sessionId}" 
  }
  ```

### Update Agent Alias

When testing a new version of your customized agent, you need to update the agent alias. This is because:

- Each agent version represents a specific configuration, including:
    - Customized agent instructions
    - Modified orchestration prompts
    - For more information on customizing your prompt, see the [Customize Prompt](CUSTOMIZATION.md#customize-prompts).
- The alias acts as a pointer to a specific agent version, allowing you to switch between different configurations easily.

Follow these steps to update the alias:

1. Open the [Amazon Bedrock Console](https://console.aws.amazon.com/bedrock)
2. Select your agent and create a new version if you've made changes
3. Copy the new alias ID of the new version

Then, update the Lambda function to use the new alias:

1. Go to the [AWS Lambda Console](https://console.aws.amazon.com/lambda)
2. Select the API Backend Lambda function (TODO: provide the exact name)
3. Navigate to the Configuration tab
4. Select "Environment variables"
5. Update the `AGENT_ALIAS_ID` variable with the new alias ID
6. Save the changes

By updating the alias, you can test your new agent configuration without changing your application code.
This allows for easy switching between different versions of your agent during development and testing.
For more information on customizing your agent, see the [Customization Guide](CUSTOMIZATION.md#customize-prompts).

### Metadata Filtering for Knowledge Base

> Note: This section is **only applicable** if you set `"deploy:knowledgebase": true` in your configuration.

Metadata filtering allows you to refine the knowledge base search results based on specific attributes. 
This feature is particularly useful when you have a large knowledge base and want to narrow down the information retrieved by the agent.

#### Setting Up Metadata

1. Prepare your metadata file:
    - Create a JSON file with the same name as your source file, adding the `.metadata.json` suffix.
    - Example: For `bedrock.pdf`, create `bedrock.pdf.metadata.json`.

2. Format your metadata JSON file:

    ```json
    {
        "metadataAttributes": {
            "service": "bedrock",
            "year": 2023
        }
    }
    ```

3. Upload both the source file and its metadata file to your S3 bucket.

4. Synchronize your knowledge base in the Amazon Bedrock console.

> Tip: You can find sample documents and metadata files in the [knowledgebase assets folder.](../packages/cdk_infra/src/assets/knowledgebase)

#### Using Metadata Filters

When querying your knowledge base, you can apply metadata filters in your API requests. Here are some examples:

Exact match filter:

```json
{
    "message": "Tell me about bedrock knowledge base quota",
    "metadata": {"equals": {"key": "service", "value": "bedrock"}}
}
```

Starts with filter:
```json
"metadata": {"startsWith": {"key": "service", "value": "bed"}}
```

Contains filter:
```json
"metadata": {"stringContains": {"key": "service", "value": "rock"}}
```

Not in filter: 
```json
"metadata": {"notIn": {"key": "service", "value": ["qbusiness", "lambda"]}}
```

Multiple conditions:
```json
"metadata": {
    "andAll":[
        {
            "greaterThan": {
                "key": "year", 
                "value": 2020
            }
        },
        {
            "lessThan":  {
            "key": "year", 
            "value": 2025
            }
        }
    ]
}
```

Access Control Example:
```json
{
    "message": "Show me the security guidelines",
    "metadata": {
        "in": {
            "key": "allowed_roles",
            "value": ["admin", "security_team"]
        }
    }
}
```

This access control example demonstrates how to:
1. Control access based on user roles
2. Use the `in` operator to check if the user's role is in the allowed roles list
3. Keep the access control simple and focused on role-based permissions

Mixed Metadata Structure Example:
```json
{
    "message": "Show me all security guidelines and Lambda documentation",
    "metadata": {
        "orAll": [
            {
                "andAll": [
                    {
                        "equals": {
                            "key": "department",
                            "value": "security"
                        }
                    },
                    {
                        "equals": {
                            "key": "access_level",
                            "value": "confidential"
                        }
                    }
                ]
            },
            {
                "equals": {
                    "key": "service",
                    "value": "lambda"
                }
            }
        ]
    }
}
```

This example shows how to:
1. Use `orAll` to combine different metadata structures
2. Keep the access control conditions for the security document
3. Include the simpler metadata filter for Lambda documents
4. Search across both types of documents in a single query

For example, if you have:
- A security document with metadata:
```json
{
    "metadataAttributes": {
        "department": "security",
        "access_level": "confidential",
        "allowed_roles": ["admin", "security_team"]
    }
}
```
- And Lambda documents with metadata:
```json
{
    "metadataAttributes": {
        "service": "lambda",
        "topic": "quota",
        "year": 2014
    }
}
```

The query will return both:
- The security document (if access control conditions are met)
- Any Lambda documents (matching the service filter)

#### Response with Metadata

When you use metadata filtering, the API response will include citation information with the relevant metadata:

```json
{
    "sessionId": "21eb9dc5-eb6e-44d9-a131-a539a2e7d382",
    "message": "The following quotas apply to Knowledge bases for Amazon Bedrock: ...",
    "citations": [
        {
            "content": "The maximum number of text units that can be processed ...",
            "metadata": {
                "x-amz-bedrock-kb-source-uri": "s3://..file_name.pdf",
                "service": "bedrock"
            }
        }
    ]
}
```

## Best Practices

- Use meaningful and consistent metadata attributes across your documents.
- Keep metadata simple and relevant to your use case.
- Test different metadata filters to ensure they retrieve the expected information.
- Always include the Cognito ID token in your API requests.
- Handle token expiration by refreshing the token when needed.
 
For more details on metadata filtering options, refer to [the Amazon Bedrock documentation on Metadata and filtering.](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-test-config.html) 