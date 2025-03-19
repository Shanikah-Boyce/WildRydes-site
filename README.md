# Building Wild Rydes: A Serverless Unicorn Ride-Sharing Web Application

### Project Overview
The *Wild Rydes* project reimagines the [AWS Serverless Workshop application](https://aws.amazon.com/serverless-workshops/), using AWS’s serverless ecosystem to create a scalable, cost-effective ride-sharing platform. The website and dependencies were sourced from the [Tiny Technical Tutorials guide](https://github.com/tinytechnicaltutorials/wildrydes-site), allowing focus on designing the serverless backend. The app enables users to register, log in and request rides via an ArcGIS-powered map.
![Front-end UI](https://github.com/user-attachments/assets/9e5782d9-b921-471d-ae7e-b61635932cf8)
*This image demonstrates the front-end functionality provided by AWS tools and ArcGIS.*

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
![image](https://github.com/user-attachments/assets/91cad274-ce7c-4733-88d0-7d2f396785aa)


### **Key Takeaways**
- Attention to Sign-In Attributes: It’s crucial to properly configure the sign-in attributes in AWS Cognito. Selecting username as the sign-in attribute ensures flexibility for users, allowing them to use their email, phone number or a custom username to log in.
- Serverless Scalability: AWS Lambda and DynamoDB played an essential role in building a scalable application, eliminating the need for manual infrastructure management.
- Simplified Authentication: AWS Cognito made user authentication and access management straightforward, enabling secure logins with minimal effort.
- Real-Time Performance: Optimizing Lambda functions for real-time processing was crucial for ensuring responsiveness and performance.
- CI/CD with Amplify: Setting up CI/CD pipelines with AWS Amplify significantly improved the development workflow, enabling faster iterations and seamless deployment.
- Effective IAM Role Management: Properly configuring IAM roles and permissions was critical to ensuring that only authorized services had access to necessary resources, thus ensuring the security of the entire application.
![lambda_execution_flow](https://github.com/user-attachments/assets/286321c5-c2f3-40f4-ae61-e5abab7165a0)

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




