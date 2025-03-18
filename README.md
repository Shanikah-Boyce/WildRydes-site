# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

The Wild Rydes project is a recreation of the original Wild Rydes ride-sharing app from the [Amazon serverless workshop](https://aws.amazon.com/serverless-workshops/). The objective was to utilize AWS's serverless architecture to create a scalable, cost-efficient, and robust unicorn ride-sharing application. This project enabled me to deepen my understanding of serverless architecture and gain hands-on experience with various AWS services such as Cognito, Lambda, and Amplify. The repository showcases the completed app, built following a tutorial from [TinyTechnicalTutorials](https://youtu.be/K6v6t5z6AsU?si=tDEysyOLwFF7SD6D).

![image](https://github.com/user-attachments/assets/efbfa338-8b3d-453d-b1a8-2093cacb472f)

## Project Objectives
The main goal of this project was to build a serverless ride-sharing platform where users could securely create accounts, log in, and interact with an ArcGIS-powered map to request rides. The platform had to provide a seamless, responsive user experience while ensuring backend scalability and cost-efficiency.

To achieve this, I focused on:
- Leveraging AWS services to manage authentication, processing and data storage in a serverless environment.
- Implementing real-time ride requests with AWS Lambda.
- Integrating ArcGIS for interactive map features.
- Ensuring scalability and performance using serverless tools like AWS Amplify and DynamoDB.

## System Design & Implementation
The application was built using a combination of AWS services to support the serverless architecture:
- Authentication: AWS Cognito was used for user registration, login, and authentication to ensure secure access.
- Backend Processing: AWS Lambda was employed to handle ride requests and process user data without the need for dedicated servers.
- API Communication: API Gateway facilitated communication between the frontend and backend, connecting users to the Lambda functions.
- Data Storage: DynamoDB was chosen for its low-latency, scalable storage to hold ride data and user details.
- Map Integration: ArcGIS powered the dynamic, interactive map interface where users could request rides.
- CI/CD Pipeline: AWS Amplify was integrated with GitHub to manage the continuous integration and deployment of the application, ensuring efficient updates.
The front-end and back-end services were connected through APIs, enabling smooth communication for real-time ride requests. Amplify played a crucial role in managing the deployment process, allowing the application to be updated automatically as new changes were pushed to the repository.

![Rocinante in Toronto](https://github.com/user-attachments/assets/60ee783b-4dc6-4fd8-ae15-36a60839ebef)

*Rocinante, the majestic unicorn, was summoned to the bustling cityscape of Toronto, Canada.*

## Challenges & Solutions
While developing Wild Rydes, I encountered several challenges:
- Learning AWS Amplify’s New UI: The redesigned Amplify interface initially seemed overwhelming, but I overcame this by reading documentation and experimenting with the platform’s features.
- Lambda Timeouts: Ensuring that Lambda functions executed quickly enough to process ride requests in real time was a key hurdle. I optimized the Lambda functions to handle requests efficiently by reducing the processing time and adjusting timeout settings.
- Integration with ArcGIS: Ensuring smooth communication between the interactive map and the backend was a challenge. By fine-tuning the integration, I ensured accurate real-time updates on the user’s location and ride request status.
These challenges were valuable learning experiences, helping me refine my understanding of serverless technologies and their application in real-world scenarios.

## Key Takeaways
- Serverless Design Benefits: I gained a deeper appreciation for the advantages of serverless architecture, especially in terms of scalability and cost-effectiveness. With AWS Lambda, I was able to handle backend processing without worrying about server maintenance or provisioning.
- Authentication & Security: AWS Cognito simplified user authentication and management, providing robust security features with minimal configuration
- Handling Real-Time Data: AWS Lambda and DynamoDB allowed the application to process and store real-time data efficiently, even under variable load conditions.
Next time, I would focus on optimizing database queries in DynamoDB to further improve performance during demand spikes and refine the UI/UX to make it more intuitive for users.

## Project Deliverables
- Fully Functional Web Application: A ride-sharing app with user authentication, an interactive map, and real-time ride requests powered by AWS Lambda.
- Source Code: Available on GitHub, demonstrating the use of AWS services in a serverless application.
- Architecture Documentation: Detailed overview of how the AWS services were integrated and how they work together to provide the serverless solution.
- CI/CD Pipeline: A working deployment pipeline integrated with AWS Amplify and GitHub to automate updates and streamline development.

/////
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

