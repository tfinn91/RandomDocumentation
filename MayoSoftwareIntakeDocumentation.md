Software Requesting using SharePoint, SharePoint Designer, and TFS
===================================================================


##PROJECT OVERVIEW

The current project on which I am working involves creating a system that can be used by end users to request software to be built by the IT team within the CFI. As it currently stands, the process for requesting software is confusing for both end users as well as the developers that receive the requests. The goal is to create a unified system that takes requests for software from end users and presents it in a comprehensive and easily manageable way for the developers. To do this, Microsoft SharePoint will be used heavily as it is a cornerstone tool at Mayo Clinic. 

A SharePoint subsite will host a form that will be presented to and populated by the end users. This form will ideally document, in depth, what exactly the user wants while also providing enough information to the development team. The form results will end up in a list on a SharePoint subsite that will be accessible to the relevant personnel within the CFI. Once the team leaders within the CFI are able to see the requests made by the end users on their personal SharePoint subsite, they will have the ability to approve or deny requests. Ideally, approved requests will be sent either straight to TFS in order to be created as a new project (NEED TO KNOW WORKFLOW IN TFS) or will continue on to another member within the CFI for further consideration. A denial should prompt the CFI member to respond to the request and notify the end user as to why their software request was denied (DO WE WANT A FORM FOR THIS TOO? SO MANY FORMS!! But probably not, email would work best in my opinion**). 


###Create SharePoint subsite

In order to get started with CFI's SharePoint suite:

```
open Internet Explorer and navigate to
https://collab.mayo.edu/team/CFI/SitePages/Home.aspx
```

Once you are at the main CFI SharePoint site, a list of subsites will be visible at the top of the screen just above the "Center for Innovation" header. We need to create a specific subsite to host our form page.
In order to create a subsite of the CFI's site, you will need to talk to the local SharePoint administrator and request a subsite. 

Once the subsite is in place, the SharePoint administrator will present you with a URL of the site. The subsites are extremely easiy to navigate between as the root URL remains the same. For example, my subsite in this instance is located at:

```
https://collab.mayo.edu/team/CFI/finntest
```

This link should take you to the home page of the specific subsite I am using under the CFI SharePoint site. Now that there is a blank site set up, it can be edited by:

```
On the SharePoint toolbar at the top of the page,
Click Page and then select Edit
```

This will refresh the page and bring editing tools onto the top of the page and you will have full control over editing the specific site. You may view and edit the source code  (HTML) by clicking the `Edit Source` button in the Markup section of the top ribbon. For more detailed information on editing the site, please refer to the [Microsoft Custom SharePoint Sites][1] page.

However, because what we are creating requires tools beyond the scope of the SharePoint web-application, we need to use the Microsoft SharePoint Designer application.

###Dowloading Microsoft Sharepoint Designer

Request Mayo access
download from microsoft and install
**CURRENT STUCK POINT**


In order to download and install Microsoft SharePoint Designer, you must first request access to uninstalling and installing applications on the computer. As you only have one hour with install/uninstall permissions, I reccommend downloading and saving the SharePoint Designer file before actually requesting access and installing. The download is around 280MB and, depending on the network, can take some time to download. So, to download:
```
Open Internet Explorer and navigate to
https://www.microsoft.com/en-us/download/details.aspx?id=35491
Click the orange download button in the middle of the screen
Check the box for "sharepointdesigner_32bit.exe"
Once selected, click Next at the bottom of the screen
The download will start
```

```
A prompt will be displayed at the bottom of the screen
Make sure that you **SAVE** the file instead of selecting Run
```

Again, when it prompts you, **make sure you save the file instead of running it**. If you choose to open the file you will get an error saying: "You must have administrative privileges to install or uninstall this product."

Once the file has downloaded entirely, it is time to request local administrator permissions. To do this:
```
Navigate to the top of the screen on your computer to bring up the Mayo Dock
Select Global Applications
Select Computer Settings from the left column
Select Mayo Computer & Printer Settings from the right column 
Along the top toolbar, select Computer Privileges and read the entire page!
Once sure, select Local Administrator and hit submit. 
```

 This will automatically grant local administrator privileges for an hour. The computer will prompt you to log off and then log in again in order for the changes to take effect. Once you log back in do the following:

```
Click Start
Click <USERNAME> at the top right of the Start Menu
Click on Downloads on the left-most column in the window that appears
Double click the Microsoft SharePoint Designer executable
```
 
This should start the installation process. Microsoft SharePoint Designer is fairly straightforward to install, so just follow the on screen directions **ANYTHING SPECIFIC HERE???**


###Using Microsoft SharePoint Designer

-Open site for editing in SharePoint Designer
-Create custom forms
-Custom editing and markup
-Link to Microsoft pages for this as well



###References
1. [Microsoft Custom SharePoint Sites][1]


[1]: https://msdn.microsoft.com/en-us/library/dd583126(v=office.11).aspx "Editing Microsoft SharePoint Sites"







