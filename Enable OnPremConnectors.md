# Enabling the creation of OnPremise Inbound Connectors on new M365 tenants

## Whats the problem ?

Beginning with February 2023, Microsoft blocked the possibility to create new OnPrem inbound connectors.

This is also known as the Advisory posted to the Service Health with the ID EX505293.

OnPremise Connectors are a requirement for the integraton with SEPPmail.cloud in parallel mode or a SEPPmail Appliance.

## How do i know i am affected ?

Customers and partners, configuring the integration with SEPPmail will use a PowerShell Module (SEPPmail365 or SEPPmail365cloud) to setup rules and connectors.

After creating the setup, the __inbound connector__ will stay *disabled* and you cannot enable it via the GUI or Powershell

## What do i need to do to solve the issue ?

Microsoft Support enabled inbound connectors in specific cases and they are documented in this [link](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/use-connectors-to-configure-mail-flow/inbound-connector-faq)

Nevertheless there are 4 easy steps to speed up this process and avoid to run into a longer discussion with MS support.

### 1. Rename the connectors "sent email identity" from the preconfigured *guid.customerdomain.tld* to __seppmail.customerdomain.tld__

To give you a real world example, for a customer contoso.eu which wants to integrate SEPPmail.cloud Germany the sent email identity would be something like

*d563a719-f13b-4427-b196-eb139b7a56a8.de.seppmail.cloud*

Save this record somewhere - you will need it later

Login to your Exchange Admin Center (EAC), go to Mailflow == Connectors and edit the [SEPPmail] Inbound connector. Go to "Edit sent email identity". Change the name from the automatically set value to: __seppmail.customerdomain.tld__, so for our example *seppmail.contoso.eu*.

### 2. Create a new "Accepted Domain" in your Exchange Online Tenant

Open M365 Admin Center, go to settings == domains and add a new domain. As domain name use __the same name as in the inbound connector__, i.e. *seppmail.contoso.eu*. Finish the Add-Domain wizard until the domain is visible in your domain list.

This domain does not have to exist, there will never be any mailflow with this domain, and we will delete it later on.

### 3. Open a ticket with microsoft support

Contact Microsoft Support and request them to enable the inbound connector. You can adapt and use the following text-template:

We need the inbound connector for our trusted and valued E-Mail Security 3rd Party "SEPPmail.Cloud" from Swizerland. SEPPmail provides cryptographic E-Mail processing, which is a required standard of our company for business communication.

This issue is referenced to the Service Health advisory with the  ID EX505293, and we have fulfilled all requirements according to the [Microsoft documentation](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/use-connectors-to-configure-mail-flow/inbound-connector-faq)

Microsoft should then positively anser your request and enable the connector.

### 4. Rebuild the Configuration

Open the Inbound Connector in EAC and set the "sent email identity" back to original value provided with the PowerShell Module guid.customerdomain.tld ( the one you recorded in step 1).


Mail should now flow through SEPPmail(.cloud). As a cleanup action you may delete the *seppmail.customerdomain.tld* from your tenant domain list of leave it as an emotional reference to this thrilling support experience.

If there are any further issues contact us at support@seppmail.de