# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

### **Project Overview**
The *Wild Rydes* project reimagines the AWS Serverless Workshop app, utilizing AWS’s serverless ecosystem to create a scalable, cost-efficient, and feature-rich ride-sharing application. Serverless architectures are becoming increasingly relevant for their scalability, cost-effectiveness, and minimal management. The app allows users to sign up, log in, and interact with an ArcGIS-powered map to request rides, while providing hands-on experience with AWS services like Cognito, Lambda, and Amplify.

---

### **Project Objectives**
The primary goal was to build a fully serverless ride-sharing app that:
- **Is Secure and Scalable**: Implementing AWS Cognito for user authentication.
- **Handles Real-Time Requests**: Utilizing AWS Lambda for dynamic ride request processing.
- **Minimizes Costs**: Leveraging serverless technologies to reduce operational overhead.

Additionally, I integrated a CI/CD pipeline with AWS Amplify to ensure seamless updates, tied to GitHub for version control.

---

### **System Architecture**
To achieve the goals outlined above, I designed a comprehensive serverless architecture using the following AWS services:

- **AWS Cognito**: Managed user authentication and access control securely.
- **AWS Lambda**: Handled backend logic and real-time ride processing.
- **API Gateway**: Facilitated communication between the frontend and backend.
- **DynamoDB**: Stored user and ride data with low-latency access.
- **ArcGIS**: Provided an interactive map to help users request rides in real-time.
- **AWS Amplify**: Enabled continuous deployment, integrated with GitHub for seamless updates.

These services work together to form a scalable, cost-efficient, and secure architecture that meets the needs of the app.

---

### **Challenges Faced**
As with any project, there were challenges:

- **Navigating AWS Amplify’s Interface**: Initially, the new interface felt overwhelming, but with the help of documentation and hands-on exploration, I was able to configure the CI/CD pipeline.

- **Optimizing Lambda Performance**: Lambda functions needed to execute within time limits, and real-time performance was crucial. I optimized execution time by adjusting timeout settings and removing unnecessary processing steps.

- **Integrating ArcGIS with Backend**: Ensuring the interactive map received real-time data was tricky. I worked on synchronizing the front-end and backend to ensure accurate location data for the ride requests.

Each challenge taught me valuable lessons about serverless architectures, cloud services, and the need for careful optimization.

---

### **Key Takeaways**
Key lessons learned include:
- **Serverless Scalability**: AWS Lambda and DynamoDB proved to be invaluable in building scalable applications without the need for manual infrastructure management.
- **Streamlined Authentication**: Cognito simplified user authentication and access management, enabling secure logins with minimal effort.
- **Optimizing for Real-Time Performance**: The importance of optimizing Lambda functions to handle real-time requests quickly became clear. Fine-tuning the execution speed was crucial for performance.
- **CI/CD with Amplify**: Setting up continuous integration and deployment through Amplify significantly improved the development process, allowing for faster iteration and seamless deployment.

I would focus on optimizing DynamoDB queries and enhancing the user interface in future updates to improve the overall experience.

---

### **Deliverables**
The outcome of this project includes:
- **A Fully Functional Ride-Sharing Web Application**: The app allows users to securely log in, interact with the ArcGIS map, and request rides—all powered by serverless technologies.
- **GitHub Repository**: The project is open-sourced, showcasing the integration of AWS services and serverless architecture.
- **System Architecture Documentation**: Detailed descriptions of how the various AWS services are integrated.
- **CI/CD Pipeline**: The project includes an automated pipeline using AWS Amplify, integrated with GitHub for continuous updates.

---

### **Personal Reflection**
This project significantly enhanced my understanding of serverless architectures and AWS services. By using AWS Lambda, Cognito, DynamoDB, and other services, I learned how to build scalable, real-time applications with minimal management overhead. Overcoming challenges like optimizing Lambda functions and integrating ArcGIS provided valuable insights into cloud-native development. I look forward to applying these lessons in future projects and exploring more serverless solutions.

---

### **Conclusion**
This project has reinforced my confidence in serverless technologies and cloud-native development. The hands-on experience with AWS services like Lambda, Cognito, and DynamoDB has deepened my understanding of how to build scalable, cost-efficient applications. In the future, I plan to continue refining my skills in serverless computing and leverage these tools to tackle more complex projects.

---

This structure flows smoothly, progressing logically from a **Project Overview** to **System Architecture**, **Challenges**, **Key Takeaways**, and finally, the **Deliverables** and **Personal Reflection**. The more traditional subheadings make it easier to read and navigate the document. Let me know if you'd like any further adjustments!
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

