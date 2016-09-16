# ECS server

Use Case when a user first registers an account in the app:

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

###LDAP schema

####users
We have users in gluu sever and we can make use of it.

####organizations
organizations will be an organizational unit. Each of these will have an entry for a different organization.

(ou=organizations,o=(org-inum),o=gluu).

For example:
  ```
  o=(guid1),ou=organizations,o=(org-inum),o=gluu
  objectclass: top
  objectclass: organization
  displayName: Acme Forming, INc.
  photo: https://acme.com/img/acme-logo.jpg
  description:
  o: (guid1)
  ```

####issuers
Like organizations, issuers will be an organizational unit but it will be under organizations.

(ou=issuers,o=(guid1),ou=organizations,o=(org-inum),o=gluu)

For example:
  ```
  id=(guid2),ou=issuers,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  objectclass: top
  objectclass: gluuIssuer
  owner: inum=(person-inum),ou=people,o=(org-inum),o=gluu
  active: true
  displayName: Issuer
  description: blah blah blah
  photo: https://acme.com/img/issuer-image.jpg
  id: (guid2)
  ```

####badges
Like issuers, badges will be an organizational unit under organizations. 

(ou=badges,o=(guid1),ou=organizations,o=(org-inum),o=gluu).

For example:
  ```
  id=(guid3),ou=badges,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  objectclass: top
  objectclass: gluuBadgeClass
  displayName: Badge Class 1
  description: blah blah
  owner: id=(guid2),ou=issuers,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  active: true
  photo: https://acme.com/img/badge-image.jpg
  id: (guid3)
  ```

####badges_requests
Like badges, badges_requests will be organizational unit under organizations.

(ou=badge_requests,o=(guid1),ou=organizations,o=(org-inum),o=gluu).

For example:
  ```
  id=(guid3),ou=badge_requests,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  objectclass: top
  objectclass: gluuBadgeRequest
  seeAlso: id=(guid3),ou=badges,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  active: true
  gluuBadgeRequester: inum=(person-inum),ou=people,o=(org-inum),o=gluu
  ```

####badges_instances
badges_instances will be organizational unit but will be under user (gluu person) as user will hold the badge.

(ou=badges,inum=(person-inum),ou=people,o=org-inum,o=gluu)

For example:
  ``` 
  id=(guid4),ou=badges,inum=(person-inum),ou=people,o=org-inum,o=gluu
  objectclass: top
  objectclass: gluuBadge
  gluuBadgeClassDN: id=(guid3),ou=badges,o=(guid1),ou=organizations,o=(org-inum),o=gluu
  status: active
  gluuBadgeHash: (hash of guid identifier -- so if db is stolen, all badge id's are not leaked!)
  gluuBadgeIssueDate: 234092384932 (Seconds from 1970...) 
  ```````
#####Notes for creating LDAP schema
  - When badge is isssued, request should be deleted from ldap, and it should be logged.
  - prefix custom attributes and objectclasses "gluu"

###APIs

Users:
  - POST : users => #Creates a new user.
  - GET : users => #Retrieves users list for admin.
  - GET : user => #Retrieves currently logged in user.
  - PUT : user => #Updates currently logged in user.
  - POST : user/forget-password => #Request an account recovery method (email or sms).
  - PUT : user/forget-password => #Updates a user password after verification.
  - GET : user/badges => #Retrieves list of valid badges assigned to currently logged in user.
  - GET : user/badges/id => #Retrieves a badge information assigned to currently logged in user.

Issuers:
  - POST : issuers => #Creates a new issuer.
  - GET : issuers => #Retrieves issuers list for admin.
  - GET : issuer => #Retrieves currently logged in issuer.
  - PUT : issuer => #Updates currently logged in issuer.
  - GET : issuer/badge-requests => #Retrieves list of badge request for an issuer.
  - GET : issuer/badge-issued => #Retrieves list of badge issued by an issuer.

Badges:
  - POST : badges => #Creates a new badge.
  - GET : badges => #Retrieves badges list for admin and users.
  - PUT : badges => #Updates a badge according to id.
