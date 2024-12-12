# Create Custom Windows AMI Image for AWS Marketplace

## EC2 Preparation:

1. Launch a base/original EC2 instance
2. Install or place the software developed by Cornerstone into the EC2
3. Install all additional dependencies (e.g. Python, AWS CLI, Java SDK etc.)
4. Upload CloudFormation templates and additional documentation (optional)
5. Test that your software works as expected
6. Empty the Recycle Bin

---

## Shutting Down EC2:

After all the installed software works as expected and all the files are in the structured directory paths ready for customer use:

1. From the Start menu, open the EC2Launch settings application
2. Without making any changes, click the **‘Shutdown with Sysprep’** button.  
   *(This will prepare your EC2 for appropriate creation of AMI for AWS Marketplace)*
3. After shutting down your EC2 with Sysprep, the instance will take a couple of minutes to be in a **‘Stopped’** state.  
   **Please do NOT do anything with the instance nor create any AMI images before the EC2 instance is in the ‘Stopped’ state.**

---

## EC2Launch Application Interface

The EC2Launch application interface should look something like this:

![ec2launchv2-settings](https://github.com/user-attachments/assets/7c5fd0a6-ec0e-4ce1-b3a4-7fa6fb9300a1)


[Reference: EC2Launch Settings Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2launch-v2-settings.html)

---

## Creating AMI from the Base/Original EC2:

1. After your EC2 instance is in a **‘Stopped’** state, select your instance.
2. Click on **‘Actions’**.
3. Select **‘Image and Templates’**.
4. Select **‘Create Image’**.
5. Provide a **Name**, **Description**, and **Tags** to your Custom AMI.
6. Wait a couple of minutes while the AMI state goes from **‘Pending’** to **‘Available’**.
7. Once the AMI is in the **‘Available’** state, it is ready to be used for launching an EC2 instance from it.

---

## Scanning the AMI for AWS Marketplace:

1. Open the Scanning tool in AWS Marketplace: [AWS Marketplace Management](https://aws.amazon.com/marketplace/management/manage-products/?#/).
2. Click on **‘Add AMI’**.
3. Select the AMI ID of the AMI you created in the steps above.
4. Enter the ARN of the AWS IAM Role for the marketplace allowing access to your AWS Account where AMI is.  
   The ARN construct is:  
   `arn:aws:iam::<Account ID>:role/<Role Name>`
   
   **Note**: In the Cornerstone Consulting account, this IAM role already exists, and the ARN is:  
   `arn:aws:iam::020877189064:role/AWSMarketplaceAMIRole`
   
   If you do not have an IAM role for AWS Marketplace in the account where the AMI is created, you can create the role. [Here is the AWS documentation on ‘Giving AWS Marketplace access to your AMI’](https://docs.aws.amazon.com/marketplace/latest/userguide/product-and-ami-policies.html).

---

## Create the IAM Role for AWS Marketplace:

1. Sign in to the AWS Management Console, open the IAM console, and go to the **Roles** page.
2. Select **Create role**.
3. On the **Create role** page, make the following selections:
   - **Select type of trusted entity** – Choose **AWS Service**.
   - **Choose a use case** – Choose **AWS Marketplace**.
   - **Select your use case** – Choose **Marketplace – AMI Assets Ingestion**.
4. To move to the next page, select **Next: Permissions**.
5. Select the **AWSMarketplaceAmiIngestion** policy. Add a permissions boundary if required, and then select **Next: Tags** to continue.
   
   **Note**: You can use permissions boundaries to limit the access given to AWS Marketplace with this role. For more information, see [Permissions boundaries for IAM entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) in the AWS Identity and Access Management User Guide.

6. To continue, select **Next: Review**.
7. Provide a name for the role, and select **Create role**.
8. You should see **"The role rolename has been created"** at the top of the page, and the role should appear in the list of roles.

---

## Finalizing AMI Scanning:

1. Click **‘Next’**.
2. If AWS Marketplace has not provisioned ProductID yet, you can still scan the AMI by leaving the **Product ID** field empty and clicking **‘Next’**.
3. The AMI image will be scanned within a couple of hours, and the report on the scanning of your AMI will be shown.
   - If everything is fine, the scan status will show as **‘Completed’**.
   - Otherwise, the scan status will show as **‘Issues Found’**.

---

## Additional Resources:

- [AMI-based product requirements for AWS Marketplace](https://docs.aws.amazon.com/marketplace/latest/userguide/product-and-ami-policies.html)
- [AWS Best Practices to Create AMI](https://docs.aws.amazon.com/marketplace/latest/userguide/best-practices-for-building-your-amis.html)
- [AWS Blog on Using Sysprep to Create a Custom Reusable Windows AMI](https://repost.aws/knowledge-center/sysprep-create-install-ec2-windows-amis)
