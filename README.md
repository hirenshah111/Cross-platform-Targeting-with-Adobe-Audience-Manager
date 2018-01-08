# Cross-Platform Targeting with Adobe Audience Manager
Integrate Adobe Audience Manager with Adobe I/O for Cross-Platform Targeting

These instructions describe how to to implement cross-platform targeting solutions with Adobe Experience Cloud and Adobe I/O products integration. The solution uses below Adobe products:


1. [Introduction](#Introduction)

1. [Set Up Products](#Set-Up-Products)

1. [Watch the Solution Work](#Watch-It-Work)

## <a name="Introduction">Introduction</a>


### Solution Use Case

An anonymous user visits a retail website. The user visits the product page, selects a color and size of the product and adds the product into the cart. The user proceeds to checkout but before completing the transaction, leaves the website, abandoning the cart. The cart abandonment activity is captured in Adobe Audience Manager segment and the user profile is served on multiple platforms to create personalized experiences.


### Solution Architecture

The following diagram shows the architecture for this solution:

![aam architecture](https://user-images.githubusercontent.com/29133525/34681317-c1c44bca-f458-11e7-9dcc-6b5a45e27a9f.png)

### What's Needed

To complete this solution, you will need authorization to use the following services:

* Adobe Launch
* Adobe Analytics/Triggers
* Adobe I/O Events
* Adobe I/O Runtime
* Adobe Audience Manager APIs
* Adobe audience Manager
* Adobe Target-Experience Targeting


## <a name="Set-Up-Products">Set Up Products</a>

To set up Adobe products for this solution:

1. [Set Up Adobe Launch](#Set-up-Launch)

1. [Set Up Analytics Triggers](#Set-up-Triggers)

1. [Set Up Adobe I/O Runtime](#Set-up-Runtime)

1. [Set Up Adobe I/O Events](#Set-up-Events)

1. [Set Up Adobe Audience Manager](#Set-up-AAM)

1. [Set Up Adobe Target](#Set-up-Target)


### <a name="Set-up-Launch">Set Up Adobe Launch</a>

To set up Launch:

1. On www.launch.adobe.com, click **New Property**.

   ![create new property](https://user-images.githubusercontent.com/29133525/34681760-13437952-f45a-11e7-898c-1561265c27b7.png)

1. On the **Create Property** box, provide the details for the new property and click **Save.**

   ![create property box details](https://user-images.githubusercontent.com/29133525/34681834-4af921e4-f45a-11e7-9f8f-3daca980310c.png)

1. Click the **Extensions** tab and install the following extensions

   * Target
   * Analytics
   * ContextHub
   * Core
   * Experience Cloud ID Service
   
   ![extensions](https://user-images.githubusercontent.com/29133525/34681903-83087260-f45a-11e7-9f10-a9b39123e28b.png)

1. Click the **Rules** tab and create a **Target** rule with the following specifications:

   ![edit rule](https://user-images.githubusercontent.com/29133525/34681953-a1b34b40-f45a-11e7-920a-352ac58f3696.png)

1. Similarly, create an **Analytics** rule with the following specifications:

   ![analytics rule](https://user-images.githubusercontent.com/29133525/34682376-038da364-f45c-11e7-959b-413760b2c451.png)

1. For the **Adobe Analytics - Set Variables** action, click the **</> Open Editor** button at the bottom of the page and add the following custom script:

```
var cart=ContextHub.getItem("cart");
var profile=ContextHub.getItem("profile");
 
var price=[];
var items=[];
var links=[];
var thumbs=[];
var origin=window.location.origin;
for(i=0;i<cart.entries.length;i++)
{
     price[i]=cart.entries[i].price;
     items[i]=cart.entries[i].title;
     links[i]=origin+cart.entries[i].page;
     thumbs[i]=origin+cart.entries[i].thumbnail;
}
 
s.eVar3=document.cookie.split("PC#")[1].split("#")[0];
s.eVar4=price.join("|");
s.eVar5=items.join("|");
s.eVar6=links.join("|");
s.eVar7=thumbs.join("|");
s.eVar8=_satellite.getVisitorId().getAudienceManagerLocationHint();
```


7. Click the **Environments** tab and create the following environments:

   * Dev
   * Stage
   * Production

      ![create environments](https://user-images.githubusercontent.com/29133525/34682566-7aee09b2-f45c-11e7-937a-0c5aadb0aef8.png)

8. Save the rule and then click the **Publishing** tab.

9. Click **Add New Library** button.

   ![add new library](https://git.corp.adobe.com/storage/user/17975/files/044c20f4-ca41-11e7-8717-c29372c9683e)
   
10. On the **Create New Library** form, specify a **Name** for the build and then select **Dev (development)** from the **Environment** drop down.

11. Click the **Add All Changed Resources** button. 
   
    ![specify build](https://git.corp.adobe.com/storage/user/17975/files/6399305a-ca42-11e7-8472-7f9dce021645)
   
12. Under **Development**, select **Build for Development** in the library drop down.
   
    ![build for dev](https://git.corp.adobe.com/storage/user/17975/files/3d7369b2-cb97-11e7-967c-6d53e37db24e)

13. Approve and publish the library by selecting the appropriate option under the drop down arrow for each phase of the build workflow (**Submitted** and **Approved**).

    ![full flow](https://git.corp.adobe.com/storage/user/17975/files/2770abb0-cba7-11e7-8c79-aaf0c67d7aeb)

14. Repeat this process for the **Stage** and **Production** environments as well.

15. The last step in the workflow is to select **Build and Publish to Production** on the drop down arrow under **Approved**.

    ![build and publish to production](https://git.corp.adobe.com/storage/user/17975/files/ab991a46-cba5-11e7-8ba8-8e0aeef7d267)

Step 2: Analytics Triggers
Triggers is a Marketing Cloud Activation core service that enable marketers to identify, define, and monitor key consumer behaviors, and then generate cross-solution communication to re-engage visitors. You can use triggers in real-time decisions and personalization.

New to Triggers? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step5:Analytics/TriggersSetup

Create a simple cart abandonment Trigger rule as shown below.



Add below dimensions with Trigger.
Dimensions
 
Custom eVar 3
Custom eVar 4
Custom eVar 5
Custom eVar 6
Custom eVar 7
Custom eVar 8
Page URL
Note: To enables these dimensions in your report suite go to Adobe Analytics->Admin→Report Suites and follow below steps.

i) Select a report suite then click on Edit Settings->Conversion→Conversion Variables



ii) Click on Add New→check the status check box and select "Enabled" from the drop down menu. Click Save.



iii) Add as many conversion variables you need, it may take some time to show up in the Trigger dimensions UI.

 

Step 3: Adobe I/O Runtime
The Adobe I/O Runtime is a serverless platform that allows you to quickly deploy custom code to respond to events, execute functions right in the cloud, all with no server set-up.

For our demo we will be deploying a script for handling Triggers I/O Events and updating the visitor profile in Customer Attributes via Target Profile APIs.



var request = require('request');
 
function main(args) {
 
    var method = args.__ow_method;
 
    if (method == "get") {
        var res = args.__ow_query.split("challenge=");
        var challenge = res[1];
        if (challenge)
            console.log("got challenge: " + challenge);
        else
            console.log("no challenge");
 
        return {body: challenge}
    }
 
    if (method == "post") {
        try {
            var body = new Buffer(args.__ow_body, 'base64');
            var jSon = JSON.parse(body);
            var mcId = jSon.trigger.mcId;
            var index = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar5.data.length - 1;
            var pcId = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar3.data[index];
            var trPrices = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar4.data[index];
            var trProducts = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar5.data[index];
            var trLink = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar6.data[index];
            var trThumb = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar7.data[index];
            var dcs_region = jSon.trigger.enrichments.analyticsHitSummary.dimensions.eVar8.data[index];
 
 
            var url = 'http://adobeiosolutionsdemo.demdex.net/event?trProducts=' + trProducts + '&trPrices=' + trPrices + '&d_mid=' + mcId + '&d_orgid=<your_org_id>@AdobeOrg&d_rtbd=json&d_jsonv=1&dcs_region=' + dcs_region;
            console.log("Calling AAM API with:" + url);
            return new Promise(function(resolve, reject) {
                request.get(url, function(error, response, body) {
                    if (error) {
                        reject(error);
                    } else {
                        resolve({body: response});
                    }
                });
            });
        } catch (e) {
            console.log("Error occured while calling the API", e);
        }
 
    }
 
}



Execute below commands to deploy the webhook.js and create a web action.


wsk action create aam webhook.js
 
wsk action update aam --web raw



After successful execution of the above commands, the web action is accessible in WWW at below location.


https://runtime-preview.adobe.io/api/v1/web/<your_openwhisk_namespace>/default/aam



Step 4: Adobe I/O Events
With Adobe I/O Events, you can code event-driven experiences, applications, and custom workflows that leverage and combine the Adobe Experience Cloud, Creative Cloud, and Document Cloud.

New to I/O Events? Before we proceed, We recommend you to read: Step by step guide for Analytics Triggers#Step6:AdobeI/OConsoleIntegration

Provide your web action URL for I/O Events integration webhook, this is where Trigger messages will be delivered by I/O Events.

 


Step 5: Adobe Audience Manager
It’s a data management platform (DMP) that helps you build unique audience profiles so you can identify your most valuable segments and use them across any digital channel.

Go to Adobe Audience Manager.

Click on "Manage Data" on the left panel.



Click on Traits.



Create a new "Trait" with below expression.




Click on "Segments" and create a new "Segment" as below. Make sure you choose the correct report suite. Select the trait created in the above step.



Click on "Destinations" and create a new "destination" as below. Choose "Browser" platform and add the segment created in the above step in segment mapping.



Save the changes.


Step 6: Adobe Target
Adobe Target is a personalization solution that makes it easy to identify your best content through tests that are easy to execute. So you can deliver the right experience to the right customer.

Go to "Target" and click on Launch.



Go to Activity and click on "Create Activity" and select "Experience Targeting".



Provide the target page URL.



Select an empty container and click on "Edit Text/HTML", we will be using this space to add custom UI elements for the user.



Click on HTML button and paste below HTML code. Save and click Next.




 

Click on "Change Audience".



Select the "Trigger-segment" audience by aam-integration-user, this is the same segment that we created in Adobe Audience Manager.
Note: If this segment doesn't appear in your target then please make sure that "Shared Audiences" is enabled for your organization. Please visit https://adobe.allegiancetech.com/cgi-bin/qwebcorporate.dll?idx=X8SVES



Update the activity settings as per your goals.



Click Save & Close.
 

Step 7: Let's Test 
Execute below steps to test and debug the solution.

Go to http://localhost:4502/content/we-retail/us/en/products/men.html
Click on any product you like.



Select a Color and Size and click on "ADD TO CART".



Click on Checkout.



Enter details and click continue.



Close the tab to initiate the cart abandonment scenario.

Go to Triggers UI page, wait for your trigger event to surface.



Go to Command Line Interface (CLI) and execute below commands.

wsk activation list runtime



Select the top most activation and execute below command.

wsk activation get f6f5ae1dcb3d4292991d63f22283fb94





Go to http://localhost:4502/content/we-retail/us/en/products/men.html again and you should see custom elements on the page.




