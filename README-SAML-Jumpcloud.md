HOWTO: SAML and Jumpcloud with Guacamole
=========

This Ansible role allows you to configure SAML authentication with your IDP. We have tested with Jumpcloud and the steps here could be adapted for another IDP. Often with SAML the names of options are slightly different, which can lead to some confusion, so YMMV.

First you need a hostname and matching certificate accepted by your browser. Testing was only done with a public DNS hostname and a matching publicly trusted SSL certificate. 
Its probably best you get Gaucamole working first with whatever certificates and reverse proxies you need, before you attempt to use SAML. In particular you should be able to login with local (eg MySQL) authentication for testing and group setup. This example assumes your Guacamole is on https://guacamole.example.com/ (and not /gaucamole!)
NOTE - this role moved the guacamole.war file to ROOT.war - forcing guacamole to be at the / location in Tomcat, not /guacmole. This is in order to make saml_strict work correctly.


## JumpCloud Configuration

1. Login to your JumpCloud tenant as an administrator.

1. From the left-hand menu, select the "SSO" option (under "User Authentication").

1. Click the big green "plus" button at the top left of the main pane.

1. Ignore all the listed "SSO Applications", and instead click on the small blue
   "Custom SAML App" button towards the bottom of the screen.

1. Provide a suitable display label, description, and logo / colour as you see
   fit.

1. Click the SSO Tab, and start filling out
   form items.  Remember to replace `https://guacamole.example.com` with the public URL
   of your Guacamole service.

   * IdP Entity ID: `https://gaucamole.example.com` is a good default, to make it unique for that Jumpcloud tenant.
   * SP Entity ID: `https://gaucamole.example.com`
   * ACS URL: `https://gaucamole.example.com/api/ext/saml/callback`
   * SAMLSubject NameID: `username`
   * SAMLSubject NameID Format: `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`
   * Signature Algorithm: `RSA-SHA256` (should already be set to that)
   * Sign Assertion: *tick this box*
   * Default RelayState: *leave blank*
   * Login URL: `https://gaucamole.example.com/api/ext/saml/login`
   * Declare Redirect Endpoint: *leave blank*
   * IDP URL: `https://sso.jumpcloud.com/saml2/tennant-guacamole-example` should be unique enough

   * Group Attributes:
     1. Tell JumpCloud to pass the list of groups to Guacamole:
       * Include group attribute: *tick this box*
       * Unlabelled text box: `group` (the value of {{ saml_group_attribute }} in this role's vars)

1. After that fairly massive effort, you can now click "activate".  *Phew!*

1. Click "continue" on the "Please confirm your new SSO connector instance" dialog.

1. If all goes well, you'll get a pleasant "SSO application created" pop-up.

1. Finally, go back into the newly-created SSO application, scroll down, expand the
   "Single Sign-On Configuration" heading, and click "Export Metadata".  Save this file
    to the guacamole server in /etc/guacamole/idp.xml or whatever the value of {{ saml_idp_metadata_url }} is set to. 
    For files you and set it to `saml_idp_metadata_url: file:///etc/guacamole/idp.xml`

 ## Ansible Variables

`saml_idp_metadata_url` as above the location of the xml file downloaded from Jumpcloud

`saml_idp_url` the value of IDP URL set above (you can't change this in jumpcloud once set)

`saml_entity_id` the IDP entity ID above eg `https://gaucamole.example.com`

`saml_callback_url` the ACS URL above eg `https://gaucamole.example.com/` without the "api/ext/saml/callback" which the gaucamole SAML addon adds to this URL

`saml_strict` to to true to enforce proper SAML security - you really should! We also moved the WAR file to the root of the tomcat directory as per the main README note, to make this option work.

`saml_debug` true of false didn't seem to actually do much :sadface:

`saml_compress_request` true for jumpcloud

`saml_compress_response` true for jumpcloud

`saml_group_attribute` group as per the Group Attribute above. 'groups' would probably be more logical

