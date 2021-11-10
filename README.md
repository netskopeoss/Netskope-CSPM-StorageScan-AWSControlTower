**Implementation Guide:**

**Netskope AWS Control Tower Integration**

***Cloud Security Posture Management and Storage Scan services***

#  {#section .TOC-Heading}

# Foreword

The Netskope CSPM and Storage Scan services are multi-account security solutions that provides visibility into resources, configurations, data protection, and malware on the AWS cloud. Implementing this solution, you can identify and remediate risky misconfigurations, identify sensitive data (DLP), and detect malware and ransomware.

The purpose of this AWS Implementation Guide is to enable every AWS Marketplace customer to seamlessly activate, deploy and configure the Netskope CSPM and Storage Scan in AWS Control Tower environment while taking full advantage of the resources pre-configured by AWS Control Tower as part of the initialization.

# Solution overview and features

[[Netskope Cloud Security Posture Management (CSPM)]{.underline}](https://www.netskope.com/products/public-cloud-security) and Storage Scan allows organizations to confidently adopt and secure multi-cloud services. The Netskope solution gives organizations the ability to continuously assess public cloud deployments to mitigate risk, detect threats, scan and protect sensitive data, and monitor for regulatory compliance. Netskope simplifies the discovery and remediation of misconfigurations across your clouds to help prevent data loss and inadvertent exposure.

Netskope helps organizations maintain compliance and best practices, provides the ability to audit security configurations and prevent data exposure, and detect "shadow IaaS" services with real-time controls. With these controls you can stop data exfiltration from external and internal threat actors and protect your sensitive data.

-   [**[Netskope CSPM]{.underline}**](https://www.netskope.com/products/public-cloud-security) enables you to gain the visibility and control across the AWS accounts in multi-account environments to secure your data, applications and services, maintain best practices and standards compliance, and to automate your incident response.

-   **Netskope Storage Scan** services allows AWS customers to easily identify and protect sensitive data on the organization's S3 buckets and to detect malware.

With the integration between AWS Control Tower and Netskope CSPM and Storage Scan services you can automatically enroll your existing AWS Control Tower accounts into Netskope CSPM and Storage Scan services, as well assure that new accounts provisioned by the AWS Control Tower account factory will be also immediately protected by the Netskope Security Cloud services.

# Architecture diagram

![](media/image2.png){width="6.767716535433071in" height="4.861111111111111in"}

*Figure 1. Netskope CSPM and Storage Scan service with AWS Control Tower Architecture Diagram*

This solution uses the [Customization for AWS Control Tower](https://aws.amazon.com/solutions/implementations/customizations-for-aws-control-tower/) (CfCT) solution to deploy the integration, the solution source code can be found on GitHub repository. This repository contains two [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates and the manifest file. The first template, Netskope-CSPM-StorageScan-Account-Enrolment-ControlTower.yaml shall be deployed in the AWS Control Tower management account home region. The CloudFormation stack deployed using this template creates the following resources on the AWS Control Tower management account:

-   [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) secret encrypted by AWS Key Management Service (KMS) key to store your Netskope API access token.

-   [Amazon CloudWatch event rule](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) triggered by the SUCCEEDED status event from the Customizations for AWS Control Tower [[AWS Step Functions]{.underline}](https://docs.aws.amazon.com/solutions/latest/customizations-for-aws-control-tower/architecture.html).

-   [AWS Lambda](https://aws.amazon.com/lambda/) function NetskopeAutoAddInstanceLambda invoked by the CloudWatch event rule above. Upon receiving a SUCCEEDED event from the CfCT, this Lambda function calls the Netskope [[Get Instance Info]{.underline}](https://docs.netskope.com/en/get-instance-info.html), [[Create an AWS Instance]{.underline}](https://docs.netskope.com/en/create-an-aws-instance.html) and [[Update an AWS Instance]{.underline}](https://docs.netskope.com/en/update-an-aws-instance.html) APIs to create or update the AWS account enrolment in your Netskope tenant.

The manifest.yaml file for CfCT describes the target AWS accounts and Organization Units (OUs) which you'd like to automatically enroll into Netskope Security Cloud. This manifest.yaml file is configured to launch [AWS CloudFormation Stackset](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) from the second template, Netskope-CSPM-StorageScan-RoleTemplate-ControlTower.yaml. This template file creates the cross-account IAM role that will be used by Netskope.

When you trigger the CfCT code pipeline, the CfCT solution deploys AWS CloudFormation StackSets with the resources defined in the NetskopeCSPM-StorageScan-RoleTemplate-ControlTower.yaml template across the AWS accounts and OUs defined in the manifest.yaml file.

Later, when you create a new managed account using [[AWS Control Tower Account Factory]{.underline}](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html), the CfCT solution uses the [[AWS Control Tower Lifecycle Event]{.underline}](https://docs.aws.amazon.com/controltower/latest/userguide/lifecycle-events.html) to invoke the same CodePipeline workflow and deploys the AWS IAM roles will be used by Netskope on the newly created account. When the CfCT solution completed the Netskope AWS IAM role deployment, the CfCT Step Functions send the SUCCEEDED event in the Amazon CloudWatch which triggers the NetskopeAutoAddInstanceLambda Lambda function to configure your AWS account in the Netskope tenant.

![](media/image2.png){width="6.767716535433071in" height="4.861111111111111in"}

*Figure 1. Netskope CSPM and Storage Scan service with AWS Control Tower Architecture Diagram*

This solution uses the [Customization for AWS Control Tower](https://aws.amazon.com/solutions/implementations/customizations-for-aws-control-tower/) (CfCT) solution to deploy the integration, the solution source code can be found on GitHub repository. This repository contains two [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates and the manifest file. The first template, Netskope-CSPM-StorageScan-Account-Enrolment-ControlTower.yaml shall be deployed in the AWS Control Tower management account home region. The CloudFormation stack deployed using this template creates the following resources on the AWS Control Tower management account:

-   [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) secret encrypted by AWS Key Management Service (KMS) key to store your Netskope API access token.

-   [Amazon CloudWatch event rule](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) triggered by the SUCCEEDED status event from the Customizations for AWS Control Tower [[AWS Step Functions]{.underline}](https://docs.aws.amazon.com/solutions/latest/customizations-for-aws-control-tower/architecture.html).

-   [AWS Lambda](https://aws.amazon.com/lambda/) function NetskopeAutoAddInstanceLambda invoked by the CloudWatch event rule above. Upon receiving a SUCCEEDED event from the CfCT, this Lambda function calls the Netskope [[Get Instance Info]{.underline}](https://docs.netskope.com/en/get-instance-info.html), [[Create an AWS Instance]{.underline}](https://docs.netskope.com/en/create-an-aws-instance.html) and [[Update an AWS Instance]{.underline}](https://docs.netskope.com/en/update-an-aws-instance.html) APIs to create or update the AWS account enrolment in your Netskope tenant.

The manifest.yaml file for CfCT describes the target AWS accounts and Organization Units (OUs) which you'd like to automatically enroll into Netskope Security Cloud. This manifest.yaml file is configured to launch [AWS CloudFormation Stackset](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) from the second template, Netskope-CSPM-StorageScan-RoleTemplate-ControlTower.yaml. This template file creates the cross-account IAM role that will be used by Netskope.

When you trigger the CfCT code pipeline, the CfCT solution deploys AWS CloudFormation StackSets with the resources defined in the NetskopeCSPM-StorageScan-RoleTemplate-ControlTower.yaml template across the AWS accounts and OUs defined in the manifest.yaml file.

Later, when you create a new managed account using [[AWS Control Tower Account Factory]{.underline}](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html), the CfCT solution uses the [[AWS Control Tower Lifecycle Event]{.underline}](https://docs.aws.amazon.com/controltower/latest/userguide/lifecycle-events.html) to invoke the same CodePipeline workflow and deploys the AWS IAM roles will be used by Netskope on the newly created account. When the CfCT solution completed the Netskope AWS IAM role deployment, the CfCT Step Functions send the SUCCEEDED event in the Amazon CloudWatch which triggers the NetskopeAutoAddInstanceLambda Lambda function to configure your AWS account in the Netskope tenant.

Pre-requisites

The following pre-requisites are required to implement the Netskope integration with AWS Control Tower:

-   Fully deployed AWS Control Tower. For information about setting up an AWS Control Tower landing zone, see [[Getting Started with AWS Control Tower]{.underline}](https://docs.aws.amazon.com/controltower/latest/userguide/getting-started-with-control-tower.html). You also need administrator privileges in the AWS Control Tower management account.

-   [[Customizations for AWS Control Tower]{.underline}](https://aws.amazon.com/solutions/implementations/customizations-for-aws-control-tower/) (CfCT) solution deployed in your AWS Control Tower home account. Follow the [implementation guide](https://docs.aws.amazon.com/solutions/latest/customizations-for-aws-control-tower/welcome.html) to deploy CfCT solution.

-   An active Netskope Security Cloud tenant. You can subscribe from AWS Marketplace by following the instruction in the next section.

This solution guide assumes working knowledge with AWS management console. We also recommend that you become familiar with the following AWS services:

-   [[AWS Lambda]{.underline}](https://aws.amazon.com/lambda/)

-   [[Amazon CloudWatch]{.underline}](https://aws.amazon.com/cloudwatch/)

-   [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.html)

-   [AWS Step Functions](https://aws.amazon.com/step-functions/getting-started/)

If you are new to AWS, see [[Getting Started with AWS]{.underline}](https://aws.amazon.com/getting-started/) . For additional information about AWS Marketplace, see [[AWS Marketplace Overview](https://aws.amazon.com/mp/marketplace-service/overview/).]{.underline}

# Deployment and Configuration Steps

Step 1.1: Subscribe to Netskope CSPM & Storage Scan on AWS Marketplace.

Locate the [**Netskope Public Cloud Security** in the AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-rhphwiywrinha?ref_=srh_res_product_title).

![](media/image3.png){width="6.768055555555556in" height="2.095138888888889in"}

Click on the **Continue to Subscribe** button.

**Step 1.2: Guidance on Contract Duration and Renewal**

In the new screen, you can configure your contract. You can select the **Contract Duration** and set the

![Graphical user interface, text, application, email Description automatically generated](media/image4.png){width="6.767716535433071in" height="7.416666666666667in"} **Renewal Settings**. 1 year option (to add when publish listing)

\<Contract duration page ScreenShot>

**Step 1.3: Select Contract Options**

Select the Contract Options to be activated with your contract. Please note that this solution is applicable for **CASB_API** and **IAAS_STORAGE**

![Graphical user interface, text, application, email Description automatically generated](media/image5.png){width="6.767716535433071in" height="7.416666666666667in"}

**Step 1.4: Create the Contract and Pay**

Once you have configured your contract, you can click on the Create contract button. You will be prompted to confirm the contract. If you agree to the pricing, select the **Pay Now** button.

\<\[Optional\] -- Marketplace Contract creation screenshot>

**Step 1.5: Set up Account**

To complete registration, choose Setup your account and follow the remaining instructions. If SaaS offering: Customer will be redirected to partner portal. Partner should include any additional steps needed here.

## Configuration: Set up Additional configuration \[SaaS vs AMI\]

## Netskope Configuration - Rest API Token and External ID

**Step 2.1: Log into the Netskope**

Using your Netskope tenant URL, login to your Netskope portal.

![](media/image6.png){width="2.20209864391951in" height="2.0092596237970253in"}

**Step 2.2: Create Rest API Token**

From Netskope console, select **Settings \> Tools \> REST API v1** from the side-bar navigation.

![](media/image7.png){width="5.601851487314086in" height="1.6019247594050743in"}

Choose **Generate New Token** and take a note of the token id and keep it secure.

**Step 2.3: Retrieve External ID**

From Netskope console, navigate to **Settings \> API-enabled Protection \> IaaS**, then select **AWS** and click **Setup**.

![](media/image8.png){width="5.7037040682414695in" height="1.732296587926509in"}

In the **New Setup** window, enter the following details:

-   12 digit AWS account ID of your AWS Control Tower management, followed by the account name. Follow the format as described in the text box.

-   Keep the default service checked and click **Next**.

![](media/image9.png){width="5.361111111111111in" height="3.8109765966754154in"}

On the next dialog window, download the CFT file and exit the **New Setup** window.

![](media/image10.png){width="5.361111111111111in" height="3.231738845144357in"}

Open the CFT file using any text editor and search for the *ExternalId*. Copy the value for both the ExternalId and the AWS account ID (see reference below).

![](media/image11.png){width="4.408733595800525in" height="2.5092596237970253in"}

## Configure Account Enrollment Template

**Step 3.1: Clone [[Netskope CSPM & Storage Scan services]{.underline}](https://github.com/netskopeoss/Netskope-CSPM-StorageScan-AWSControlTower.git) GitHub repository.**

From your local terminal, run:

git clone https://github.com/netskopeoss/Netskope-CSPM-StorageScan-AWSControlTower

**Step 3.2:** **Deploy CloudFormation Stack**

Sign into the AWS Control Tower Management account as administrator and deploy the Netskope automation solution for enrolling AWS accounts managed by the AWS Control Tower in the Netskope CSPM and Storage Scan services.

1.  Navigate to the AWS CloudFormation management console and change the region to the AWS Control Tower home region.

2.  Click **Create Stack** and choose **With new resources (standard)**.

> ![Graphical user interface, application, Teams Description automatically generated](media/image12.png){width="6.5in" height="1.2381944444444444in"}

3.  Choose **Upload a template file** then click on Choose file. Choose the Netskope-CSPM-StorageScan-Account-Enrolment-ControlTower.yaml from the directory on your disk where you cloned the GitHub repository to, click **Open** and then click **Next**.

![Graphical user interface, text, application, email Description automatically generated](media/image13.png){width="6.5in" height="2.660416666666667in"}

4.  Enter the stack name and the parameters for your deployment:

  -------------------------------------- ------------------------------------------------------------------------------------------------------------------------
  Netskope tenant FQDN                   Enter Netskope tenant FQDN (f.e. example.goskope.com)

  Netskope tenant REST API token         Enter Netskope tenant REST API token provided from Step 2

  Netskope AWS Account ID                Enter Netskope trusted AWS Account ID provided from Step 2

  Security Administrator email address   Enter your Security Administrator email address

  AWS KMS Key ID                         Optional KMS Key ID to encrypt Netskope API token in Secrets Manager. If not specified aws/secretsmanager will be used

  Security Scan enabled                  Enter ( \"true\", \"false\" ), whether Netskope CSPM Security violations Scan is enabled

  CSPM security scan interval            Choose the CSPM security and compliance violations scan interval in minutes

  DLP Scan enabled                       Enter ( \"true\", \"false\" ), whether Netskope storage DLP Scan is enabled

  MalwareScan Scan enabled               Enter ( \"true\", \"false\" ), whether Netskope storage Malware Scan is enabled
  -------------------------------------- ------------------------------------------------------------------------------------------------------------------------

![Graphical user interface, application Description automatically generated](media/image14.png){width="5.6484044181977255in" height="6.519804243219598in"}

5.  Click **Next**.

6.  Optionally, enter the Tags for your CloudFormation stack and / or click Next.

![Graphical user interface, application Description automatically generated](media/image15.png){width="5.49667104111986in" height="4.137183945756781in"}

7.  Acknowledge creating IAM resources and click **Create stack**.

![Graphical user interface, text, application Description automatically generated](media/image16.png){width="5.468659230096238in" height="3.147985564304462in"}

8.  When CloudFormation stack is in the CREATE_COMPLETE state, you can navigate to the Resources tab and see the resources created by the stack.

![Graphical user interface, text, application, email Description automatically generated](media/image17.png){width="6.722222222222222in" height="2.4907403762029747in"}

You deployed the Netskope enrollment automation solution for AWS accounts managed by AWS Control Tower. This template deploys the AWS Lambda function that will perform account registration in Netskope.

**­­­CfCT Manifest Modification**

Next, you need to deploy AWS IAM cross-account role that will be used by the Netskope CSPM and Storage Scan services across your AWS Organizations accounts. You will use the Customizations for AWS Control Tower (CfCT) solution for this deployment. The CfCT Customizations Pipeline workflow will deploy the required AWS IAM role on the AWS accounts specified in the manifest.yaml file. The AWS Step Functions SUCCEEDED event upon customizations workflow completion will trigger the enrollment automation you deployed on step 3.2 and will provision your AWS Accounts in you Netskope tenant.

**Step 3.3:** Deploy the AWS IAM cross-account roles for Netskope access using the Customizations for AWS Control Tower.

1.  Open the manifest.yaml file you cloned from the Netskope GitHub repository.

2.  Replace the AWS Region in the lines 4, 11 and 34 with the AWS Region where your Control Tower landing zone is deployed.

3.  Replace the **TrustedAccountID** and the **ExternalID** with the corresponding values for you Netskope tenant.

    1.  Sing into the Netskope tenant management console and navigate to Settings \> API-enabled Protection \> IaaS, then select AWS and click Setup.

    2.  In the New Setup window, enter the 12 digits of any of your AWS accounts ID, followed by the account name. Follow the format as described in the text box. Keep the default service checked and click Next.

> ![Graphical user interface, text, application, email Description automatically generated](media/image18.png){width="4.370543525809274in" height="3.0593799212598425in"}

3.  Download the CFT file and close the New Setup - Amazon Web Services.

> ![Graphical user interface, text, application, email Description automatically generated](media/image19.png){width="5.169653324584427in" height="3.185191382327209in"}

4.  Open the CoudFormation template file you just downloaded and look for the AWS IAM policy statement similar to this one:

# Statement:

#  - Action:

#  - sts:AssumeRole

#  Condition:

#  StringEquals:

#  sts:ExternalId:01234567890abcdef01234567890abcdef0123456

#  Effect: Allow

#  Principal:

#  AWS:

#  - arn:aws:iam::123456789012:root

> The value of sts:ExternalId is your ExternalID and the account ID in the AWS account ARN is your TrustedAccountID

4.  Set the values for the **SecurityScan**, **DLPScan** and **TrustedAccountID** to true or false to configure the Netskope Security Cloud functionality you'd like to use to protect your AWS accounts.

5.  Configure the AWS Organizations Units and accounts you'd like to enroll in Netskope CSPM and Storage Scan services in the deployment_targets section of the manifest file. Please refer to the [[Customizations for AWS Control Tower Develop Guide]{.underline}](https://s3.amazonaws.com/solutions-reference/customizations-for-aws-control-tower/latest/customizations-for-aws-control-tower-developer-guide.pdf) for more details about working with the manifest file.

6.  Save the manifest file.

Now you can trigger the Customizations for AWS Control Tower code pipeline using the newly edited manifest file.

This instruction assume that you are using AWS [[CodeCommit]{.underline}](https://aws.amazon.com/codecommit/) as the Customizations for AWS Control Tower CodePipeline source repository. You can also use Amazon S3 as your configuration source as described in [[this Documentation]{.underline}](https://docs.aws.amazon.com/solutions/latest/customizations-for-aws-control-tower/appendix-a.html). Steps for deploying the Control Tower customizations for Netskope account enrollment using Amazon S3 as a configurations source are similar to the steps using AWS CodeCommit below.

7.  Sign into the AWS CodeCommit management console, choose the **custom-control-tower-configuration** repository and copy its **HTTPS (GRC) URL**:

> ![A screenshot of a computer Description automatically generated](media/image20.png){width="5.899311023622047in" height="1.1842738407699038in"}

8.  If not yet installed, install the git-remote-codecommit package in your local machine:

pip install git-remote-codecommit

9.  Assuming you already have proper AWS CLI credentials for accessing the control-tower-configuration repository in your Control Tower landing zone management account, clone this repository to your local machine:

git clone (HTTPS (GRC) URL copied above

1.  Replace the manifest.yaml file in the cloned repository by the one you edited above.

> cd control-tower-configuration
>
> cp \<new manifest.yaml file>

2.  Check in the manifest file with the customizations for Netskope cross-accounts AWS IAM roles deployment into the CodeCommit repository:

> git status git add -A
>
> git commit -m 'Netskope Automatic Accounts enrollment'
>
> git push

**Step 3.4: Verification steps:**

After you have triggered the Customizations for AWS Control Tower workflow for deploying Netskope cross-account AWS IAM roles on your AWS accounts, you can check its progress on the CodePipeline.

1.  Navigate to [AWS CodePipeline Console](https://console.aws.amazon.com/codepipeline/) on your AWS Control Tower Management account.

2.  Choose **Custom-Control-Tower-CodePipeline** to track the status of the pipeline at various stages.

![Graphical user interface, application, Teams Description automatically generated](media/image21.png){width="6.768055555555556in" height="3.4770833333333333in"}

3.  When the last stage of the pipeline completed, open the Netskope management console, navigate to **Settings \> API-enabled Protection \> IaaS** and check that your existing AWS accounts configured in the manifest file successfully enrolled in the Netskope CSPM and Storage Scan services.

![Graphical user interface, application, Teams Description automatically generated](media/image22.png){width="6.804141513560805in" height="2.75in"}

4.  You also you can monitor the NetskopeAutoAddInstanceLambda automation Lambda function execution logs by opening AWS CloudWatch management console, navigating to **Logs -\> Log groups** menu and choosing the NetskopeAutoAddInstanceLambda log group for your Lambda function.

![Graphical user interface, text, application, email Description automatically generated](media/image23.png){width="6.5in" height="2.084722222222222in"}

What to expect

Now, at any time when you provision a new AWS account using AWS Control Tower account factory, the solution will automatically enroll that account in the Netskope CSPM and Storage Scan services if this account belongs to the one of the AWS Organization's Units previously provisioned in the manifest file.

## Using the integration 

4.1.1 Netskope CSPM

When your existing or new AWS accounts are enrolled into Netskope CSPM, Netskope performs the complete scan of AWS instances and services running on these account and makes the list of your AWS inventory available for you in one single place on the Netskope Management console and via the [[View Cloud Provider Inventory API]{.underline}](https://docs.netskope.com/en/view-cloud-provider-inventory.html).

To view your AWS instances and services, sign into the Netskope Management console, navigate to API-enabled Protection-\>Inventory and use the filters on top of the page to find the accounts, instances and more.

![Graphical user interface, application Description automatically generated](media/image24.png){width="6.5404188538932635in" height="3.4205325896762906in"}

Next, you can configure Netskope Security Posture policies that will provide you a clear Image of your cloud security posture and enable you to see how the environment is performing against standards and best practices like CIS (Center for Internet Security) benchmarks.

To configure Netskope Security Posture policies, on the Netskope Management console navigate to Policies-\>Security Posture and click New Policy.

Here you can choose the AWS Accounts and Security Posture standards and frameworks for which you'd like Netskope to monitor your environment:

![Graphical user interface, text, application, email Description automatically generated](media/image25.png){width="6.768055555555556in" height="3.1305555555555555in"}

To assure your AWS cloud security posture is aligned with your organization's specific security standards you also can write [[custom security assessment rules]{.underline}](about:blank) using Domain Specific Language (DSL).

The Netskope Security Posture Assessment results are available for you on the Netskope Management console and via the [[View Security Assessment Violations]{.underline}](https://docs.netskope.com/en/view-security-assessment-violations.html) API. To view Security Posture Assessment results on the Netskope Management console, navigate to API-enabled Protection-\>Security Posture and use the filters on top of the page to find the accounts, instances and more. From this page, you can create ad hoc reports by printing the pages or exporting the tables. Or you can go to reports to schedule reports to be sent out on a schedule.

![Graphical user interface, application, website Description automatically generated](media/image26.png){width="6.768055555555556in" height="3.5965277777777778in"}

You can also take a look at the [[CSPM security violation findings Auto-Remediation framework]{.underline}](https://github.com/netskopeoss/CSPM-AWS-AutoRemediation) if you would like to remediate some of the security violations findings automatically.

4.1.2 Netskope Storage Scan services

Netskope Storage Scan services enable you to identify sensitive data and detect malware on your S3 buckets.

# When your AWS account is enrolled into Netskope Storage Scan, you can configure retroactive and ongoing scans for your S3 buckets. To configure the Netskope Storage Scan policies from the Netskope Management console, navigate to Policies-\>API Data Protection and click on NEW POLICY. You can define the AWS account, AWS S3 buckets and DLP profiles when you are defining the Storage Scan policy from the Netskope Management Console. 

![Graphical user interface, text, application Description automatically generated](media/image27.png){width="6.557292213473316in" height="3.1777646544181977in"}

You can also use [[Netskope Storage Scan Policies]{.underline}](https://docs.netskope.com/en/manage-storage-scan-policies.html) API to define granular policies for scanning your organizations' S3 buckets.

The results of the DLP and Malware scans can be consumed by the Alerts REST API ([[Netskope Alerts REST API]{.underline}](https://docs.netskope.com/en/get-alerts-data.html)), or via the Netskope Console Alerts or Incident Management pages.

For more information about Netskope CSPM and Storage Scan services, please refer to the [[Netskope Knowledge Portal]{.underline}](https://docs.netskope.com/?lang=en) and the [[Netskope Community]{.underline}](https://community.netskope.com/).

Best Practices

-   Visit [Netskope Knowledge portal](https://docs.netskope.com/index.html?lang=en) to learn more about Netskope product setup, administration and features.

-   Find the latest resource such as blogs, case studies and references in [Netskope resource center.](https://resources.netskope.com/)

Partner contact information

<https://www.netskope.com/contact-us>
