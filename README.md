# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application
---

### **Project Overview**
The *Wild Rydes* project reimagines the AWS Serverless Workshop app, leveraging AWS’s serverless ecosystem to create a scalable, cost-efficient and feature-rich ride-sharing application. The website, including HTML, CSS, javascript and ArcGIS integration, was provided as part of the tinytechnicaltutorials guidance. This allowed me to focus primarily on building and configuring the serverless backend. The app enables users to sign up, log in, and interact with the interactive map powered by ArcGIS to request rides, offering hands-on experience with AWS services like Cognito, Lambda, and Amplify.

---

### **Project Objectives**
The primary goal of this project was to build a fully serverless ride-sharing application that is secure, scalable, and cost-effective. To achieve this, I implemented AWS Cognito for user authentication, ensuring a smooth and secure login process. AWS Lambda was utilized for processing real-time ride requests, with the Lambda code provided as part of the tutorial. This allowed me to concentrate on integrating the serverless architecture, while ensuring that the app could process ride requests dynamically. By using a serverless approach, I minimized operational costs and enabled the app to scale efficiently without relying on traditional infrastructure. Additionally, I integrated AWS Amplify to establish a CI/CD pipeline, ensuring continuous deployment and smooth updates, all while maintaining version control through GitHub.

---

### **System Architecture**
To meet the objectives outlined above, I designed a robust serverless architecture using the following AWS services:

- **AWS Cognito**: Managed user authentication and secure access control.
- **AWS Lambda**: The provided Lambda code handled backend logic and ride request processing, enabling me to focus on configuring the serverless infrastructure rather than writing the code from scratch.
- **API Gateway**: Facilitated communication between the frontend and backend.
- **DynamoDB**: Stored user and ride data with low-latency access.
- **ArcGIS**: The interactive map, included in the AWS workshop, allowed users to view their location and request rides in real-time.
- **AWS Amplify**: Enabled continuous deployment and integration, linked with GitHub for seamless updates.
- **AWS IAM (Identity and Access Management)**: IAM roles and policies were used to securely manage permissions and access to the various AWS services in the project. I created IAM roles to assign appropriate permissions for the Lambda functions, API Gateway, and other AWS resources. These roles ensured that only authorized services could access sensitive data, thereby maintaining secure communication and operations between the components.

These services worked together to create a secure, cost-efficient, and scalable architecture that met the app's needs.

---

### **Challenges Faced**
As with any project, there were several challenges:

- **Navigating AWS Amplify’s Interface**: The new interface initially felt overwhelming, but with thorough documentation and hands-on exploration, I was able to configure the CI/CD pipeline effectively.
- **Setting Up IAM Roles and Permissions**: While configuring IAM roles, it was essential to ensure the right permissions were granted for each AWS service. I had to carefully configure IAM roles for Lambda functions, API Gateway, and other services, which initially posed a challenge, but with research and trial and error, I successfully ensured secure and efficient access control.

These challenges provided valuable learning experiences, teaching me more about serverless architectures, cloud services, and the importance of careful optimization.

---

### **Key Takeaways**
Key lessons learned from this project include:

- **Serverless Scalability**: AWS Lambda and DynamoDB played an essential role in building a scalable application, eliminating the need for manual infrastructure management.
- **Simplified Authentication**: AWS Cognito made user authentication and access management straightforward, enabling secure logins with minimal effort.
- **Real-Time Performance**: Optimizing Lambda functions for real-time processing was crucial for ensuring responsiveness and performance.
- **CI/CD with Amplify**: Setting up CI/CD pipelines with AWS Amplify significantly improved the development workflow, enabling faster iterations and seamless deployment.
- **Effective IAM Role Management**: Properly configuring IAM roles and permissions was critical to ensuring that only authorized services had access to necessary resources, thus ensuring the security of the entire application.

In future updates, I would focus on optimizing DynamoDB queries and enhancing the user interface to improve the overall user experience.

---

### **Deliverables**
The final deliverables for this project include:

- **A Fully Functional Ride-Sharing Web Application**: The app allows users to securely log in, interact with the ArcGIS map, and request rides—all powered by serverless technologies.
- **GitHub Repository**: The project is open-sourced, demonstrating the integration of AWS services and serverless architecture.
- **System Architecture Documentation**: Detailed documentation outlining how the AWS services are integrated.
- **CI/CD Pipeline**: An automated CI/CD pipeline using AWS Amplify, integrated with GitHub for continuous updates.

---

### **Personal Reflection**
This project significantly enhanced my understanding of serverless architectures and AWS services. By working with AWS Lambda, Cognito, DynamoDB, and other services, I gained hands-on experience building scalable, real-time applications with minimal infrastructure management. Overcoming challenges like optimizing Lambda functions, working with IAM roles, and integrating ArcGIS provided valuable insights into cloud-native development. I look forward to applying these lessons in future projects and further exploring serverless solutions.

---

### **Conclusion**
This project has reinforced my confidence in using serverless technologies for cloud-native development. The hands-on experience with AWS services like Lambda, Cognito, and DynamoDB has strengthened my understanding of how to create scalable, cost-efficient applications. Moving forward, I plan to continue refining my serverless computing skills and leverage these tools for even more complex projects.

---

### **Changes Made:**
1. **Added IAM Role Mention**: IAM roles and permissions have been included to explain how security and access control were handled for AWS resources.
2. **Clarified Role Configuration**: The challenges section now includes specific information on setting up IAM roles and policies for secure access to resources.

Let me know if you need any additional changes!

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

