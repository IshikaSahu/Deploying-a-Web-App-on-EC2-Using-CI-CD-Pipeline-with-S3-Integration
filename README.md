# Deploying a Web App on EC2 Using CI/CD Pipeline with S3 Integration.

**Introduction:**

In this project, we will deploy a web application on an Amazon EC2 instance using a Continuous Integration and Continuous Deployment (CI/CD) pipeline. We will utilize AWS CodeCommit, AWS CodeBuild, AWS CodeDeploy, and S3 bucket to achieve this. The web application, named "User Details Form," collects user information such as full name, email ID, mobile number, and address. When a user submits the form, the data is stored in an S3 bucket, triggering a Lambda function called "sending_email" that sends a notification email to the designated email address.

**Components:**

1. **CodeCommit Repository:**
   - Repository Name: `User_Details_Form`
   After creating repository create a IAM User go to User > Security Credentials and generate HTTPS Git credentials for AWS CodeCommit and download it.
   Now come inside of your repositiory and click on clone URL > clone HTTPS. You will get one link. Copy that link.
   Go to VS Code and open your terminal and write git clone <paste link> and enter. Your repository will open inside your vs code.
   Now paste all your files which need to commit and write these commands to commit them
   - cd User_Details_Form
   - git add .
   - git commit -m "This is my code"
   - git push origin master
   You can see your files are in your CodeCommit repository

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/a7d5c5de-a1a5-4598-827f-0f51db5263b2)

2. **CodeBuild Project:**
   - Project Name: `User_Detail_Project`
   Add source AWS CodeCommit, choose repository that we created above and choose branch master.
   Choose a operating system as ubuntu and runtime standard
   Create a new service role and give it a name.
   Add one more service in that role after project get created in IAM > Roles > Your Role i.e. AmazonS3FullAccess
   - Build Specification: `buildspec.yml`
   No need to give buildspec file name it will pick it itself.
   The buildspec.yml file is uploaded when we commited code.
   - Artifacts: `User_Detail_Project.zip`
   - Artifacts Stored in S3 Bucket: `ish-user-details-app`

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/377f301e-e5f6-409f-897d-d064f73f6232)

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/38409d70-829f-40d1-bfbb-cff9ff798b19)

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/3a7bbecb-ab8e-49f7-9c82-2897220e52ba)

3. **EC2 Instance:**
   - Instance Name: `ish-user-details-app`

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/f81b4ef6-b1c5-45ee-b61a-eda67b29d5d5)

4. **CodeDeploy Application and Deployment Group:**
   - Application Name: `User_Details_App`
   Create application and choose compute platform as EC2 instance

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/a3976ee9-ae44-4d5e-b291-5ad83b05b13f)

   - Deployment Group Name: `User_Details_App_Grp`
   Create a service role which have these all permissions and choose it 
   AmazonEC2FullAccess, AmazonS3FullAccess, AWSCodeDeployRole, AmazonEC2RoleforAWSCodeDeploy, AWSCodeDeployFullAccess, AmazonEC2RoleforAWSCodeDeployLimited
   Attach this IAM role in Ec2 instance also.
   Select Instance Action > Security > Modify IAM role.
   Deployment Type - in-place
   Environment - EC2 Instance
   Key - Name, Value - ish-user-details-app
   Agent - Never. We are going to build our agent in EC2 instance only.

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/73e2f88a-0a7a-4bfb-b133-52ccb425a3f3)  

   - Create Delopyment - `User_Details_App_Deploy`
   - Revision Location for Deployment: `s3://ish-user-details-app/User_Detail_Project`
It will start deploying.

5. **Agent Setup:**
   - An `install.sh` file will be created to the EC2 instance to set up the CodeDeploy agent.
   - vim install.sh
   #!/bin/bash 
   This installs the CodeDeploy agent and its prerequisites on Ubuntu 22.04.  
   sudo apt-get update 
   sudo apt-get install ruby-full ruby-webrick wget -y
   cd /tmp 
   wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb     
   mkdir codedeploy-agent_1.3.2-1902_ubuntu22
   dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22
   sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control
   dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/
   sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb
   systemctl list-units --type=service | grep codedeploy
   sudo service codedeploy-agent start
   sudo service codedeploy-agent status
   = esc + :wq + enter
   - bash install.sh

6. **AppSpec File:**
   - The `appspec.yml` file, fetched from the S3 bucket in the zip file, will be used for installing the server and running the web application.
   The appspec.yml file is uploaded when we commited code.

7. **CodePipeline:**
   - Pipeline Name: `User_Details_Form_Pipeline`
   - Give source as CodeCommit and details
   - Give Choose AWS CloudPipeline
   - Choose AWS CodeBuild and give project name
   - Choose AWS CodeDeploy

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/5c29db6e-bdf2-479b-94ea-7f4dcdadce0f)

After deploy process done. Copy public Ip of EC2 instance and paste it in new tab and hit enter a form will get open.

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/bba1d78c-62f6-4adc-ab48-f3a1b1c8d225)

8. **Form Data Storage:**
   - Fill the details of user
   - When the user clicks on the submit button, the user details will be stored in an S3 bucket named `ish-form-data`.

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/53e9b7f8-273b-400a-af9e-56fb058b4a2a)  

9. **Event Trigger and Lambda Function:**
   - An event is triggered when the form data is stored in `ish-form-data`.
   - A Lambda function named `sending_email` is executed.
   - The Lambda function fetches the email ID of the user from the form data.
   - Using AWS SES service, an email is sent to `sahuishika2000@gmail.com` with the subject "Got user detail" and the body "Hello,
   We got User details of Email Id  [email_address]"

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/7d3971f6-4350-4d80-8572-b0dd4f03ab36)

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/dc260c6e-c974-40fb-832a-2d1cfedf6cc3)

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/8103e27a-38ae-4050-b379-7cf18de0a61c)

**Lambda Function Code**

import boto3
import re

def get_most_recent_object(bucket_name):
    s3_client = boto3.client('s3')
    response = s3_client.list_objects_v2(Bucket=bucket_name)
    
    if 'Contents' in response:
        # Sort the objects based on the last modified timestamp in descending order
        objects = sorted(response['Contents'], key=lambda x: x['LastModified'], reverse=True)
        return objects[0]['Key']  # Get the key of the most recent object
    else:
        return None

def lambda_handler(event, context):
    # Get the S3 bucket and object information from the event
    s3_bucket = 'ish-form-data'

    # Get the most recent object in the bucket
    most_recent_object_key = get_most_recent_object(s3_bucket)
    
    if most_recent_object_key:
        # Read the content of the most recent object
        s3_client = boto3.client('s3')
        response = s3_client.get_object(Bucket=s3_bucket, Key=most_recent_object_key)
        text_data = response['Body'].read().decode('utf-8')
  

    # Use regular expression to find the email address
    email_pattern = r'Email Id\s*:\s*([\w\.-]+@[\w\.-]+)'
    match = re.search(email_pattern, text_data)
    if match:
        email_address = match.group(1) //xyz@gmail.com
    else:
        email_address = None

    if email_address:
        # Send email using Amazon SES
        ses = boto3.client('ses', region_name='ap-south-1')  # Change the region if necessary
        sender_email = 'sahuishika2000@gmail.com'  # Replace with your verified sender email address
        subject = "Got User Details"
        message = "Hello,\n\nWe got User details of Email Id " + " " + email_address

        # Send the email
        response = ses.send_email(
            Source=sender_email,
            Destination={
                'ToAddresses': [sender_email]
            },
            Message={
                'Subject': {
                    'Data': subject
                },
                'Body': {
                    'Text': {
                        'Data': message
                    }
                }
            }
        )

        print("Email sent to:", sender_email)
        print("SES Response:", response)
    else:
        print("Email address not found in the text file.")

10. **Recieved Mail:**

![image](https://github.com/IshikaSahu/Deploying-a-Web-App-on-EC2-using-CI-CD-Pipeline/assets/71627396/0d5e2402-60a5-4e2f-af1f-6bfcd25c08ed)

**Conclusion:**

Through the integrated CI/CD pipeline, the "User Details Form" web application is deployed on the EC2 instance `ish-user-details-app`. Users can submit their details, which are stored in the `ish-form-data` S3 bucket and trigger an email notification to the designated email address. This seamless process ensures efficient deployment and communication with the end-users.
