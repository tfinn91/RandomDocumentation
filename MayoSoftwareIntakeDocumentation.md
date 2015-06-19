Software Requesting using SharePoint, SharePoint Designer, and TFS
===================================================================


##PROJECT OVERVIEW

The current project on which I am working involves creating a central system that can be used by end users to request software to be built by the IT team within the CFI. As it currently stands, the process for requesting software is confusing for both end users as well as the developers that receive the requests. The goal is to create a unified system that takes requests for software from end users and presents it in a comprehensive and easily manageable way for the developers. To do this, Microsoft SharePoint will be used heavily as it is a cornerstone tool at Mayo Clinic. 

A SharePoint subsite will host a form that will be presented to and populated by the end users. This form will ideally document, in depth, what exactly the user wants while also providing enough information to the development team. The form results will end up in a list on a SharePoint subsite that will be accessible to the relevant personnel within the CFI. Once the team leaders within the CFI are able to see the requests made by the end users on their personal SharePoint subsite, they will have the ability to approve or deny requests. Ideally, approved requests will be sent either straight to TFS in order to be created as a new project (NEED TO KNOW WORKFLOW IN TFS) or will continue on to another member within the CFI for further consideration. A denial should prompt the CFI member to respond to the request and notify the end user as to why their software request was denied (DO WE WANT A FORM FOR THIS TOO? SO MANY FORMS!! But probably not, email would work best in my opinion**). 


###Create SharePoint subsite

In order to get started with CFI's SharePoint suite, navigate, in Internet Explorer, to:

```
https://collab.mayo.edu/team/CFI/SitePages/Home.aspx
```

