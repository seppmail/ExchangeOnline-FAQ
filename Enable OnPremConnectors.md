# Enabling the creation of OnPremise Inbound Connectors on new M365 tenants

## Introduction

This document describes how to solve the issue with disabled OnPrem inbound connectors in Exchange Online.

Abbreviations used:

EAC = Exchange Administration Center
TLD = Top Level Domain

## What is the problem?

Beginning with February 2023, Microsoft blocked the possibility to create new OnPrem inbound connectors.
This is also known as an advisory posted to the M365 Service Health Channel with ID EX505293.
OnPremise Connectors are a requirement for the integration with SEPPmail.cloud in parallel mode or a SEPPmail Appliance in general.

## How do I know I am affected?

Customers and partners, configuring the integration with SEPPmail will use a PowerShell Module (SEPPmail365 or SEPPmail365cloud) to setup rules and connectors. After creating you will get an error message similar to this one:

> Microsoft.Exchange.Management.Tasks.OrganizationInboundConnectorProhibitedException
> Für dieses Dienstangebot können Sie keine eingehenden Connectors in Ihrer Organisation "your-tenant-id" erstellen oder aktualisieren.

The setup of the __inbound connector__ will stay *disabled* and you cannot enable it via the GUI or Powershell.

## What do I need to do to solve the issue?

Microsoft Support enables inbound connectors in specific cases and the cases are documented in this [Supported Configurations to enable OnPrem Inbound Connectors](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/use-connectors-to-configure-mail-flow/inbound-connector-faq).

To speed up the process, we figured out 4 easy steps to speed up this process and avoid to run into a longer discussion with MS support.

## 4 Steps to enable your inbound connectors

### 1. Rename the connectors "sent email identity"

To give you a real world example, for a customer contoso.eu which wants to integrate SEPPmail.cloud region germany the sent email identity would be something like:

>*d563a719-f13b-4427-b196-eb139b7a56a8.de.seppmail.cloud*

Use Exchange Admin Center (EAC) to look into the connector configuration and save this record somewhere - you will need it later.

Login to your EAC, go to Mailflow ==> Connectors and edit the "[SEPPmail]" inbound connector. Go to "Edit sent email identity". Change the name from the automatically set value to: 

>__seppmail.customerdomain.tld__

so for our example *seppmail.contoso.eu*.

### 2. Create a new "Accepted Domain"

Open M365 Admin Center, go to settings == domains and add a new domain. As domain name use __the same name as used in the inbound connector__, i.e. *seppmail.contoso.eu*. Finish the Add-Domain wizard until the domain is visible in your domain list.

This domain does not have to exist in any DNS-Server, there will never be any mailflow with this domain, and we will delete it later on.

### 3. Open a ticket with Microsoft support

Contact Microsoft Support and request them to enable the inbound connector. You can adapt and use the following text-template:

*We need the inbound connector for our trusted and valued E-Mail Security 3rd Party "SEPPmail.Cloud" from Swizerland. SEPPmail provides cryptographic E-Mail processing, which is a required standard of our company for business communication.
This issue is referenced to the Service Health advisory with the  ID EX505293, and we have fulfilled all requirements according to the [Microsoft documentation](https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/use-connectors-to-configure-mail-flow/inbound-connector-faq)*

Microsoft should then positively answer your request and enable the connector.

### 4. Rebuild the Configuration

After Microsoft enabled the inbound connector, open the Inbound Connector in EAC and set the "sent email identity" back to original value provided with the PowerShell Module (guid.regioncode.customerdomain.tld)  - the one you recorded in step 1.

If the SEPPmail Transportrules are enabled, E-Mails should now flow through SEPPmail(.cloud).

As a cleanup action you may delete the *seppmail.customerdomain.tld* from your tenant domain list of leave it as an emotional reference to this thrilling support experience.

If there are any further issues contact us at support@seppmail.de with detailed information about your tenant, the setup you did, an Exchange Online Report and an error description.
