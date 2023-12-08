# Leveraging Managed Identities to Invoke API Management from Azure Logic App

Organizations are continuously looking for ways to streamline their processes, enhance security, and improve the overall efficiency of their operations. One common challenge faced by businesses is securely accessing and managing APIs  while ensuring the confidentiality and integrity of data. 

In this workshop, we'll explore how organizations can leverage Azure Logic App and Managed Identities to achieve seamless integration with Azure API Management, simplifying authentication and enhancing security.

Managed Identities provide a secure way for Azure services and resources to authenticate themselves to other Azure services, eliminating the need for explicit credentials or secrets. They allow services like Azure Functions and Azure Logic Apps to access other resources securely without the complexities of managing credentials.

## **High level steps:**

Step 1: Register an application in Azure AD to represent the logic app(client application)

Step 2: Create a managed identity for Logic App

Step 3: Associate the Managed Identity to the Application Role

Step 4: Configure Logic App to trigger HTTP Action to invoke the API

Step 5: Configure a JWT validation policy in the APIM to pre-authorize requests

Step 6: Testing - Trigger the logic app to run

**Step 1: Register an application in Azure AD to represent the logic app(client application)**

1. In your Azure Portal, go to Azure Active Directory, select App Registrations
2. Select New registration
3. When the Register an application page appears, enter your application's registration information:
    In the Name section, enter a meaningful application name that will be displayed to users of the app, such as client-app.
    In the Supported account types section, select an option that suits your scenario.
4. Leave the Redirect URI section empty
5. Select Register to create the application.
6. On the app Overview page, find the Application (client) ID value and record it for later.
7. Under the Manage section of the side menu, select Expose an API and set the Application ID URI with the default value. Record this value for later.
8.Under the Manage section of the side menu, select App roles then click Create app role:
    In the Display name, enter a meaningful role name for example: AddRole
    Allowed member types: select Applications
    Value: example: AddRole
    Description: <as necessary>
    Do you want to enable this app role? checked
    Click Apply
    Record the role ID for later
9. Repeat the step 8 to add additional App roles (if any) supported by your API.
 
**Step 2: Create a managed identity for Logic App**

You can either use system assigned managed identity or user assigned managed identity. 

System assigned managed identity is tied directly to the lifecycle of the Azure resource which its assigned. When you delete the resource, the managed identity is also removed. Each resource can have only one System Assigned Managed Identity, and it can't be shared with other resources. 

User Assigned Managed Identities, as the name suggests, are created explicitly by users within Azure AD. You can create them independently of Azure resources. Unlike System Assigned Managed Identities, User Assigned Managed Identities are not tied to a specific Azure resource's lifecycle. They are created and deleted independently of any resource. It can be associated with one or more Azure resources, allowing you to share the identity across different resources. 

To learn more about it refer this [link](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).

I have used system assigned managed identity for Logic App.

  Go to the Azure Portal (https://portal.azure.com/).
  Navigate to the Logic App you want to configure with Managed Identity.
  Under the Logic App's "Settings," click on "Identity."
  In the "Identity" blade, enable the System-assigned Managed Identity for your Logic App. Click "Save." 

![image](https://github.com/anu-01/ManagedIdentityforAPIM/assets/19187402/bc492c7a-0121-420c-bdea-acac73bd6c07)


**Step 3: Assign Managed Identity access to the Application Role using powershell**

Use the script from [file](https://github.com/anu-01/ManagedIdentityforAPIM/blob/main/AssignManagedID.ps)   
in azcli powershell to assign managed identity access to the application role


 # Assign the managed identity access to the app role.

    New-AzureADServiceAppRoleAssignment -ObjectId $managedIdentityObjectId  -Id $appRoleId -PrincipalId $managedIdentityObjectId -ResourceId $serverServicePrincipalObjectId


I have used the script from Microsoft docs [link](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-assign-app-role-managed-identity-powershell?WT.mc_id=AZ-MVP-5002438)

** Step 4: Configure Logic App to trigger HTTP Action to invoke the API**

In your Logic App workflow:
<ol>
  <li>Add an HTTP action to make requests to the APIM API endpoint</li>
  <li>Provide the API endpoint in the URI and select the method</li>
 <li> Add the Ocp-Apim-Subscription-Key HTTP header to the request, passing the value of a valid subscription key. (you can fetch it from APIM)</li>
  
  <li>Add authentication header and select the Authentication type as Managed Identity, select system-assigned managed identity and audience as the Application ID URI you recorded from the step 1.</li>
</ol>

![image](https://github.com/anu-01/ManagedIdentityforAPIM/assets/19187402/431dd124-7968-4a1b-a12b-ee605346fb9d)


**Step 5: Configure a JWT validation policy in the APIM to pre-authorize requests**

Add the following  [Validate JWT policy](https://github.com/anu-01/ManagedIdentityforAPIM/blob/main/jwtpolicy.xml) to <inbound> policy section of your API which checks the value of the audience claim in an access token obtained from Azure AD and checks the additional claims for the app role and returns an error message if the token is not valid. Refer this link for more information.
 

 

**Step 6: Testing - Trigger the logic app to run**

Trigger your Logic App to run, and it will use its Managed Identity to authenticate and make requests to the APIM resource.

 ![image](https://github.com/anu-01/ManagedIdentityforAPIM/assets/19187402/c3de1a93-0f7f-4132-8be6-6319c288499a)
	
	
That's it! Your Logic App is now configured to access the API Management resource using Managed Identity.

The Managed Identity seamlessly handles authentication to Azure API Management, eliminating the need for managing credentials or tokens manually. Although the setup process might initially appear complex, the long-term benefits are invaluable.

With this approach, you've effectively established a password less solution for your internal API interactions. This is especially valuable for projects involving extensive system integrations, where simplicity, security, and streamlined workflows are paramount. Embracing Managed Identities paves the way for a future where secure, hassle-free API communication becomes the norm.

 

 
