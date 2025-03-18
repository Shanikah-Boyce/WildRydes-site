# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

This project is a recreation of the Wild Rydes ride-sharing app from the original [Amazon serverless workshop](https://aws.amazon.com/serverless-workshops/). The app leverages modern tools and AWS services to create a serverless unicorn ride-sharing service where users can request rides through an interactive map powered by ArcGIS. The repository showcases the completed app, built following a tutorial from [TinyTechnicalTutorials](https://youtu.be/K6v6t5z6AsU?si=tDEysyOLwFF7SD6D).

![image](https://github.com/user-attachments/assets/efbfa338-8b3d-453d-b1a8-2093cacb472f)

## Project Overview
The Wild Rydes app provides a fun and scalable ride-sharing service where users can create accounts, log in securely, and request unicorn rides. By utilizing AWS services such as IAM, Cognito, Amplify, Lambda, API Gateway, and DynamoDB, the app delivers real-time processing of ride requests while maintaining a serverless architecture for scalability and cost-effectiveness. GitHub is used for version control, and AWS Amplify manages the CI/CD pipeline.

## Business Problem
The Wild Rydes app addresses the need for a fun, user-friendly, and scalable ride-sharing platform. It allows users to request rides in a whimsical, magical environment while utilizing serverless architecture to ensure smooth, efficient performance without the need for manual server management.

## User Experience (UX)
The app prioritizes simplicity and engagement:
- Account Creation & Authentication: Handled securely through Amazon Cognito.
- Ride Requesting: Users interact with an ArcGIS-powered map to select pickup locations and request rides.
- Real-Time Processing: Serverless functions ensure instant ride processing without delay.

![Rocinante in Toronto](https://github.com/user-attachments/assets/60ee783b-4dc6-4fd8-ae15-36a60839ebef)

*Rocinante, the majestic unicorn, was summoned to the bustling cityscape of Toronto, Canada.*








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

