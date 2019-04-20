## Introduction

Welcome to Global Azure Bootcamp 2019! All around the world user groups and communities want to learn about Azure! On April 27, 2019, all communities will come together once again in the sixth great Global Azure Bootcamp event! Each user group will organize their own one day deep dive class on Azure the way they see fit and how it works for their members. The result is that thousands of people get to learn about Azure and join together online under the social hashtag #GlobalAzure!

### About the 2019 Louisville Global Azure Bootcamp

The 2019 Louisville Global Azure Bootcamp is a free one-day global training event on Azure, from the community to the community. See our event [home page](https://global.azurebootcamp.net/locations/united-states-louisville/) for more details. 

This years format will be a blend of brief presentations, followed by hands-on and guided labs. 

Our speakers include:

* [Chad Green](https://twitter.com/chadgreen)
    * [ScholarRx](http://http://www.scholarrx.com/)
    * [CodePaLOUsa](http://www.codepalousa.com/)
    * [ChadGreen.com](http://www.chadgreen.com/)
* [Mike Branstein](https://twitter.com/mikebranstein)
    * [KiZAN Technologies](http://kizan.com)
    * [Brosteins](https://brosteins.com)

### Getting Started

To get started you'll need the following pre-requisites. Please take a few moments to ensure everything is installed and configured.

* Microsoft Windows PC or Mac or Linux. Just have a laptop.
* [Azure Subscription](https://azure.microsoft.com) (Trial is ok, or an Azure account linked to a Visual Studio subscription. See later sections of this chapter to create a free trial account or activate your Visual Studio subscription)

### What You're Building

Azure is big. Really big. Too big to talk about all things Azure in a single day. 

We've assembled an exciting workshop to introduce you to several Azure services that cloud developers should know about:
* [Web app](https://azure.microsoft.com/en-us/services/app-service/web/)

TODO: Add more.

#### Key concepts and takeaways

* Navigating the Azure portal
* Using Azure Resource Groups to manage multiple Azure services
* Deploying a web app to Azure web app service

TODO: Add more.

### Agenda

* Chapter 0: Introduction
* Chapter 1: Getting Started in Azure

TODO: Update

### Materials

You can find additional lab materials and presentation content at the locations below:

* Presentation: [https://github.com/mikebranstein/global-azure-bootcamp-2019](https://github.com/mikebranstein/global-azure-bootcamp-2019)
* Source code for the code used in this guide: [https://github.com/mikebranstein/global-azure-bootcamp-2019](https://github.com/mikebranstein/global-azure-bootcamp-2019)
* This guide: [https://github.com/mikebranstein/global-azure-bootcamp-2019-instructions](https://github.com/mikebranstein/global-azure-bootcamp-2019-instructions/)

### Creating a Bootcamp Trial Subscription

There are several ways to get an Azure subscription, such as the free trial subscription, the pay-as-you-go subscription, which has no minimums or commitments and you can cancel any time; Enterprise agreement subscriptions, or you can buy one from a Microsoft retailer. In this exercise, you'll create a trial subscription using the code you were given at the bootcamp.

<h4 class="exercise-start">
    <b>Exercise</b>: Create a Subscription with the Bootcamp Trial
</h4>

Browse to [https://www.microsoftazurepass.com/](https://www.microsoftazurepass.com/).

Use the Azure Code on the handout you were given to get started.

This concludes the exercise.

<div class="exercise-end"></div>


### Creating a Trial Azure Subscription

> **If you already have an Azure account** 
>
> If you have an Azure account already, you can skip this section. If you have a Visual Studio subscription (formerly known as an MSDN account), you get free Azure dollars every month. Check out the next section for activating these benefits.

There are several ways to get an Azure subscription, such as the free trial subscription, the pay-as-you-go subscription, which has no minimums or commitments and you can cancel any time; Enterprise agreement subscriptions, or you can buy one from a Microsoft retailer. In exercise, you'll create a free trial subscription.

<h4 class="exercise-start">
    <b>Exercise</b>: Create a Free Trial Subscription
</h4>

Browse to the following page http://azure.microsoft.com/en-us/pricing/free-trial/ to obtain a free trial account.

Click *Start free*.

Enter the credentials for the Microsoft account that you want to use. You will be redirected to the Sign up page.

> **Note** 
>
> Some of the following sections could be omitted in the Sign up process, if you recently verified your Microsoft account.

If you already ahve an Azure subscription tied to your Microsoft account, you may see a screen like this:

<img src="images/chapter0/existing-subscription.png" class="img-medium" />

You're wekcome you use your existing subscription for the bootcamp. If you're planningto use your existing subscription, you can skip this exercise. Click *Sign Up* to create a new subscription.

Complete step 1 by entering your mobile phone number.

<img src="images/chapter0/verify.png" class="img-medium" />

Select *Text me* or *Call me* to verify that you are a real person. Typein the verification code you receive on the phone or via text.

Next, complete the Payment information section.

> **A Note about your Credit Card** 
>
> Your credit card will not be billed, unless you remove the spending limits. If you run out of credit, your services will be shut down unless you choose to be billed.

<img src="images/chapter0/payment.png" class="img-medium" />

Press *Next* after completing the credit card section.

In the *Technical Support* section, select the last option (No technical support) and click *Next*.

<img src="images/chapter0/tech-support.png" class="img-medium" />

In the *Agreement* section, check the *I agree to the subscription Agreement*, *offer details*, and *privacy statement* option, and click *Sign up*.

Your free subscription will be set up, and after a while, you can start using it. Notice that you will be informed when the subscription expires.

<img src="images/chapter0/agreement.png" class="img-medium" />

Your free trial will expire in 29 days from it's creation.

<div class="exercise-end"></div>

### Activating Visual Studio Subscription Benefits

If you happen to be a Visual Studio subscriber (formerly known as MSDN) you can activate your Azure Visual Studio subscription benefits. It is no charge, you can use your MSDN software in the cloud, and most importantly you get up to $150 in Azure credits every month. You can also get 33% discount in Virtual Machines and much more.

<h4 class="exercise-start">
    <b>Exercise</b>: Activate Visual Studio Subscription Benefits
</h4>

To active the Visual Studio subscription benefits, browse to the following URL: [http://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/](http://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/)

Scroll down to see the full list of benefits you will get for being a MSDN member. There is even a FAQ section you can read.

Click *Activate your monthly Azure credit* to activate the benefits.

<img src="images/chapter0/activate.png" class="img-medium" />

You will need to enter your Microsoft account credentials to verify the subscription and complete the activation steps.

<div class="exercise-end"></div>

### Preparing your Azure environment

You might be wondering how you can participate in a cloud development workshop and not need any software installed.

Thanks to the Azure Resource Manager and some nifty templates I put together, we're going to provision a virtual machine (VM) with Visual Studio (and all the tools you'll need) installed in your Azure subscription. From that point forward, you can work from the VM. 

It takes about 10 minutes to get the VM deployed to your subscription, so let's get started!

<h4 class="exercise-start">
    <b>Exercise</b>: Provisioning a Visual Studio Community VM in your Azure Subscription
</h4>

Start by clicking the *Deploy to Azure* button below.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmikebranstein%2Fvscommunity-workshop-vm%2Fmaster%2Ftemplate.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png" class="img-override" /></a>

This opens the Azure portal in a new tab of your browser. If you're prompted to sign in, do so. 

When the page loads, you'll see this custom deployment page:

<img src="images/chapter0/custom-deployment.png" class="img-override" />

#### Under *Basics*, select/enter the following
- Subscription: *your Azure subscription*
- Resource group: *Create new*
- Resource group name: *workshop-vm*, or some other name that's easy to remember
- Location: *East US*

> **Resource Groups** 
>
> Formally, resource groups provide a way to monitor, control access, provision and manage billing for collections of assets that are required to run an application, or used by a client or company department. Informally, think of resource groups like a file system folder, but instead of holding files and other folders, resource groups hold azure objects like storage accounts, web apps, functions, etc.

#### Under *Settings*, enter
- Virtual Machine Name: *workshop-vm*, or some other name that is less than 15 characters long, and no special characters
- Admin Username: *your first name*, or some other username without spaces
- Admin Password: *P@ssW0rd1234*, or another 12-character password with upper, lower, numbers, and a special character 

> **WARNING** 
>
> Do not forget your username and password. Write it down for today. 

#### Approving the "Purchase"

Scroll down to the bottom of the page and click two boxes:
1. I agree to the terms and conditions stated above
2. Pin to dashboard

Press the *Purchase* button.

#### Deploying the VM

After a few moments, the deployment of your VM will begin, and you'll see a status notification in the upper right:

<img src="images/chapter0/deployment-start1.png" class="img-override" />

...and a deployment tile on your dashboard:

<img src="images/chapter0/deployment-start2.png" class="img-override" />

Now, wait for about 10 minutes and your virtual machine will be deployed and ready to use.

<div class="exercise-end"></div>

That's it for the pre-requisites for today's workshop. Wait until your VM is created, and we'll be getting started soon!

