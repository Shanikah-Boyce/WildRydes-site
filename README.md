# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

This project is a recreation of the Wild Rydes ride-sharing app from the original [Amazon serverless workshop](https://aws.amazon.com/serverless-workshops/). The app leverages modern tools and AWS services to create a serverless unicorn ride-sharing service where users can request rides through an interactive map powered by ArcGIS. The repository showcases the completed app, built following a tutorial from [TinyTechnicalTutorials](https://youtu.be/K6v6t5z6AsU?si=tDEysyOLwFF7SD6D).

![image](https://github.com/user-attachments/assets/efbfa338-8b3d-453d-b1a8-2093cacb472f)

## Project Goals
The primary objectives of this project were to deepen my understanding of serverless architecture and to create a unique, functional web application. Through this project, I gained hands-on experience with a range of AWS services and explored real-time processing, user authentication, and scalable design principles. Wild Rydes serves as a demonstration of how serverless solutions can be leveraged to build robust applications that are both cost-effective and efficient.

## User Experience and Functionality
The application provides a seamless and intuitive user experience. Users can securely create accounts and log in via AWS Cognito, allowing them to interact with a dynamic ArcGIS-powered map to request rides. AWS Lambda is used for real-time ride processing, ensuring quick response times, while the overall application is built on a serverless architecture to optimize efficiency and scalability. Additionally, AWS Amplify integrates with GitHub for version control and continuous integration/continuous delivery (CI/CD), streamlining the development and deployment process.

![Rocinante in Toronto](https://github.com/user-attachments/assets/60ee783b-4dc6-4fd8-ae15-36a60839ebef)

*Rocinante, the majestic unicorn, was summoned to the bustling cityscape of Toronto, Canada.*


## Technical Architecture
The application's design relies on multiple AWS services, which work together seamlessly:
- AWS IAM manages secure access across resources.
- AWS Cognito enables secure user authentication.
- AWS Lambda handles backend logic and real-time processing.
- API Gateway facilitates communication between the front-end and back-end.
- DynamoDB stores user and ride data with low latency.
This robust architecture ensures reliability, scalability, and cost-effectiveness.

## Challenges and Solutions
One of the key challenges I faced was adjusting to the redesigned AWS Amplify user interface (UI). However, with persistence and a willingness to explore its features, I gradually developed a deeper understanding and became more proficient.

## Performance and Scalability
The serverless approach ensures efficient performance and seamless scalability. AWS Lambda processes requests in real time, while the pay-as-you-go model minimizes operational costs. The architecture effectively handles demand spikes, showcasing the practicality of serverless solutions in real-world applications.

## Conclusion
Wild Rydes exemplifies how serverless architecture enables the development of scalable and efficient applications. Inspired by the AWS Serverless Workshop, as adapted by TinyTechnicalTutorials, this project provided valuable experience in AWS services while overcoming technical challenges. Its innovative design highlights the possibilities of modern development practices, even when based on a now-unavailable resource.



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

