# Create a Service User for UXRisk integration <!-- omit in toc -->

- [App Registration](#app-registration)
- [API permissions](#api-permissions)
- [UXRisk Admin App](#uxrisk-admin-app)

## App Registration

Log into portal.azure.com as a Global administrator.

Select 1. Azure Active Directory in the left menu – 2. App registrations in the next menu and 3. New registration to the right.
![alt text](images/create_app_1.png)

On the next screen
Give the application a name representing the service user and add https://authy as Redirect URI.
Click Register
![alt text](images/create_app_2.png)

On the next screen
![alt text](images/create_app_3.png)
The highlighted ids will be used for your integration to UXRisk

## API permissions
![alt text](images/create_app_4.png)

Click Add a permission
![alt text](images/create_app_5.png)

Select APIs my organization uses, search for uxrisk and select it

![alt text](images/create_app_6.png)

Make sure Delegated permissions is selected and tick user_impersonation.

Click Add permissions

![alt text](images/create_app_7.png)

Add a new secret for this user – click New client secret

![alt text](images/create_app_8.png)

Give it a description and choose when it should expire

![alt text](images/create_app_12.png)

Make sure that you copy and save the secret value.
This completes the setup of the service user in Azure AD.

## UXRisk Admin App

Make sure to select the correct organisation – main or sandbox – by selecting from the dropdown
![alt text](images/create_app_10.png)

Click Access Control

![alt text](images/create_app_11.png)

Give the service user a name and enter (paste) the Object ID from a previous step.

Click create. Now you have a service user in the correct organisation to run integrations.
