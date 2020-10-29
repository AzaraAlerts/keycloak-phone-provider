# Keycloak Phone Provider

With this provider you can **enforce authentication policies based on a verification token sent to users' mobile phones**.
Currently, there are implementations of Twilio and TotalVoice and YunTongXun SMS sender services. That said, is nice to note that more
services can be used with ease thankfully for the adopted modularity and in fact, nothing stop you from implementing a 
sender of TTS calls or WhatsApp messages. 

This is what you can do for now:
  + Check ownership of a phone number (Forms and HTTP API)
  + Use SMS as second factor in 2FA method (Browser flow)
  + Reset Password by phone
  + Authentication by phone
  + only use phone Register
  
Two user attributes are going to be used by this provider: _phoneNumberVerified_ (bool) and _phoneNumber_ (str). Many
users can have the same _phoneNumber_, but only one of them is getting _phoneNumberVerified_ = true at the end of a 
verification process. This accommodates the use case of pre-paid numbers that get recycled if inactive for too much time.

## Compatibility

This was initially developed using 10.0.2 version of Keycloak as baseline, and I did not test another user storage beyond
the default like Kerberos or LDAP. I may try to help you but I cannot guarantee.

## Usage

**Build:** To build the project simply run `mvn package` after cloning the repository. At the end of the goal, the `build`
directory will contain all jars correctly placed in a WildFly-like folder structure. 

**Installing:**
 
  1. Merge that content with the root folder of Keycloak. You can of course delete the modules of services you won't use,
  like TotalVoice if you're going to use Twilio.
  2. Open your `standalone.xml` (or equivalent) and (i) include the base module and at least one SMS service provider in
  the declaration of modules for keycloak-server subsystem. (ii) Add properties for overriding the defaults of selected
  service provider and expiration time of tokens. (iii) Execute the additional step specified on selected service provider
  module README.md.
  3. Start Keycloak.

i. add modules defs
```xml
<subsystem xmlns="urn:jboss:domain:keycloak-server:1.1">
    <web-context>auth</web-context>
    <providers>
        <provider>classpath:${jboss.home.dir}/providers/*</provider>
        <provider>module:keycloak-sms-provider</provider>
        <provider>module:keycloak-sms-provider-dummy</provider>
    </providers>
...
```
ii. set provider and token expiration time
```xml
<spi name="phoneMessageService">
    <provider name="default" enabled="true">
        <properties>
            <property name="service" value="TotalVoice"/>
            <property name="tokenExpiresIn" value="60"/>
        </properties>
    </provider>
</spi>
```

**User and password login after verification phone by sms**

  in Authentication page, copy the browser flow and add a subflow to the forms, then adding `OTP Over SMS` as a
  new execution. Don't forget to bind this flow copy as the de facto browser flow.
  Finally, register the required actions `Update Phone Number` and `Configure OTP over SMS` in the Required Actions tab.


**Only use phone login or get Access token use endpoints:**

Under Authentication > Flows:
Copy the 'Direct Grant' flow to 'Direct grant with phone' flow
Click on 'Actions > Add execution' on the 'Provide Phone Number' line
Click on 'Actions > Add execution' on the 'Provide Verification Code' line
Delete or disable other
Set both of 'Provide Phone Number' and 'Provide Verification Code' to 'REQUIRED'

Under 'Clients > $YOUR_CLIENT > Authentication Flow Overrides' or 'Authentication > Bindings' 
Set Direct Grant Flow to 'Direct grant with phone' 


**Reset credential**
 Testing , coming soon!

**Fast registration by phone**
Under Authentication > Flows:
Copy the 'Registration' flow to 'Registration fast by phone' flow
Click on 'Registration Fast By Phone Registration Form > Actions > Add execution' on the 'Fast Registration By Phone' line
Click on 'Registration Fast By Phone Registration Form > Actions > Add execution' on the 'Provide Phone Validation' line
Delete or disable 'Password Validation'

**registration add http request param to user attribute**
Click on 'Registration Fast By Phone Registration Form > Actions > Add execution' on the 'Request Param Reader' line

Set both of 'Provide Phone Number' and 'Provide Verification Code' and 'Request Param Reader' to 'REQUIRED'

Under Authentication > Bindings
Set Registration Flow to 'Registration fast by phone' 

**About the API endpoints:** 

You'll get 2 extra endpoints that are useful to do the verification from a custom application.

  + GET /auth/realms/{realmName}/sms/verification-code?phoneNumber=+5534990001234 (To request a number verification. No auth required.)
  + POST /auth/realms/{realmName}/sms/verification-code?phoneNumber=+5534990001234&code=123456 (To verify the process. User must be authenticated.)

You'll get 2 extra endpoints that are useful to do the OTP from a custom application.
  + GET /auth/realms/{realmName}/sms/authentication-code?phoneNumber=+5534990001234 (To request a number verification. No auth required.)
  + POST /auth/realms/shuashua/protocol/openid-connect/token
    Content-Type: application/x-www-form-urlencoded
    grant_type=password&phone_number=$PHONE_NUMBER&code=$VERIFICATION_CODE&client_id=$CLIENT_ID&client_secret=CLIENT_SECRECT


And then use Verification Code authentication flow with the code to obtain an access code.


## Thanks
Some code written is based on existing ones in these two projects: [keycloak-sms-provider](https://github.com/mths0x5f/keycloak-sms-provider)
and [keycloak-phone-authenticator](https://github.com/FX-HAO/keycloak-phone-authenticator). Certainly I would have many problems
coding all those providers blindly. Thank you!
