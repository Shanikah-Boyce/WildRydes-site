# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

### Project Overview
The *Wild Rydes* project reimagines the [AWS Serverless Workshop application](https://aws.amazon.com/serverless-workshops/), using AWS’s serverless ecosystem to create a scalable, cost-effective ride-sharing platform. The website and dependencies were sourced from the [Tiny Technical Tutorials guide](https://github.com/tinytechnicaltutorials/wildrydes-site), allowing focus on designing the serverless backend. The app enables users to register, log in and request rides via an ArcGIS-powered map.

#### Project Goals
The goal was to build a secure, scalable, and cost-effective serverless application, minimizing operational costs and enabling efficient scaling without traditional infrastructure.
This project provided valuable experience with AWS services like Cognito, Lambda and Amplify.

### **System Architecture**
To meet the objectives outlined above, I designed a robust serverless architecture using the following AWS services:
- **AWS Cognito**: Managed user authentication and secure access control.
- **AWS Lambda**: The provided Lambda code handled backend logic and ride request processing, enabling me to focus on configuring the serverless infrastructure rather than writing the code from scratch.
- **API Gateway**: Facilitated communication between the frontend and backend.
- **DynamoDB**: Stored user and ride data with low-latency access.
- **AWS Amplify**: Enabled continuous deployment and integration, linked with GitHub for seamless updates.
- **AWS IAM (Identity and Access Management)**: IAM roles and policies were used to securely manage permissions and access to the various AWS services in the project. I created IAM roles to assign appropriate permissions for the Lambda functions, API Gateway, and other AWS resources. These roles ensured that only authorized services could access sensitive data, thereby maintaining secure communication and operations between the components.
  ![image](https://github.com/user-attachments/assets/ecc2bc44-7ad6-481e-922a-ee29d777de5c)

These services worked together to create a secure, cost-efficient, and scalable architecture that met the app's needs.

### **Challenges Faced**
I encountered an issue with AWS Cognito while setting up user authentication. I selected email as the sign-in identifier but had trouble logging in using an email address on the website. After some investigation, I realized that I needed to choose username as the primary sign-in attribute instead.

In Cognito, when username is selected, users can sign in with their email, phone, or a custom username, depending on what they provided. By changing the sign-in attribute to username, I was able to resolve the issue, allowing users to log in using their email address, phone number, or a custom username.

This experience taught me to pay close attention to the sign-in configuration settings in Cognito, to avoid misconfigurations and ensure a smooth user experience.
![image](https://github.com/user-attachments/assets/a3d4ccab-7413-43e6-8ccf-20637d2e480c)


### **Key Takeaways**
- Attention to Sign-In Attributes: It’s crucial to properly configure the sign-in attributes in AWS Cognito. Selecting username as the sign-in attribute ensures flexibility for users, allowing them to use their email, phone number or a custom username to log in.
- Serverless Scalability: AWS Lambda and DynamoDB played an essential role in building a scalable application, eliminating the need for manual infrastructure management.
- Simplified Authentication: AWS Cognito made user authentication and access management straightforward, enabling secure logins with minimal effort.
- Real-Time Performance: Optimizing Lambda functions for real-time processing was crucial for ensuring responsiveness and performance.
- CI/CD with Amplify: Setting up CI/CD pipelines with AWS Amplify significantly improved the development workflow, enabling faster iterations and seamless deployment.
- Effective IAM Role Management: Properly configuring IAM roles and permissions was critical to ensuring that only authorized services had access to necessary resources, thus ensuring the security of the entire application.

### **Deliverables**
The final deliverables for this project include:
- **A Fully Functional Ride-Sharing Web Application**: The app allows users to securely log in, interact with the ArcGIS map, and request rides—all powered by serverless technologies.
- **GitHub Repository**: The project is open-sourced, demonstrating the integration of AWS services and serverless architecture.
- **System Architecture Documentation**: Detailed documentation outlining how the AWS services are integrated.
- **CI/CD Pipeline**: An automated CI/CD pipeline using AWS Amplify, integrated with GitHub for continuous updates.

### **Personal Reflection** 
This project enhanced my understanding of serverless architectures and AWS services, offering hands-on experience with tools like Lambda, Cognito, and DynamoDB to build scalable, real-time applications with minimal infrastructure.  

### **Key Challenges and Insights**  
Overcoming challenges, such as optimizing Lambda functions and managing IAM roles, provided valuable insights into cloud-native development and strengthened my confidence in using serverless technologies to create secure, cost-efficient solutions. These experiences have inspired me to refine my skills and apply them to more complex projects in the future.



---

///

## The Application Code
The application code is here in this repository.

## The Lambda Function Code
Here is the code for the Lambda function, originally taken from the [AWS workshop](https://aws.amazon.com/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/module-3/ ), and updated for Node 20.x:

```node
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
```

## The Lambda Function Test Function
Here is the code used to test the Lambda function:

```json
{
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

