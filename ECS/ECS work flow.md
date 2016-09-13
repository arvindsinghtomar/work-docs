# ECS server

Use Case when a user first registers a account in the app:

  - User should have a unique ID
  - When user is registered with his/her information, Server creates a Unique string (Might be from user details | only user id | totally random string) which will be sent to mobile App
  - Using this unique string app generates a unique QR code for the user and displays it to user.

Use Case when a user tries to get authorization for any resource:
  - User should display his/her QR code to admin
  - Admin should scan the QR code and app sends ID of QR code which was used to generate it to server for verification and user details.
  - Server verifies the unique ID and send back the user information to the app.
  - Admin validates the information and the user for authorization of the resources.

> For securing the server API we will be using JWT token 
> as JWT tokens are light weight and also the token is minimal
> and contain data as well.
> We can figure out another security mechanism if we find any better way
> of securing the APIs.

###Data design
![Alt text](images/datadesign.png "Data Design")
