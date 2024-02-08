# Test Exchange Online Transport Rules functionality

Exchange Online Transport Rules are an important part of Exchange Online mailflow, and also SEPPmail Appliance and SEPPmail.cloud mailflow connectivity.

Sometimes, especially after customizing them, it might be necessary to diagnose why of why not a specific transport rule triggers or not. For this reason the Exchange Online PowerShell module offers a simply way to send an e-mail through the transport (and DLP-rule) engine in its recent incarnation.

We summarized the blog-post by the Exchange Online engineering team to the most necessary pieces below.

```powershell
Import-Module ExchangeOnlineManagement
connect-exchangeOnline
```

Be sure you have at least ExchangeOnlineManagement Version 3.0.4 by running with Get-Module:
![Show Module Version](https://github.com/seppmail/ExchangeOnline-FAQ/assets/22378822/0e752cc8-fbc1-4e23-b3d3-4ab832a72eb1)

## Option 1: Testing with a default Testmail

Lets try a Test-message for contoso.eu, which is hosted in Exchange Online.

```powershell
Test-Message -Sender 'user.ofthetenant@contoso.eu' -Recipients 'anotheruser.ofthetenant@contoso.eu' -SendReportTo 'someadmin@contoso.eu' -TransportRules
```

This will generate a report, sent from the postmaster@contoso.eu with information on the transport rule behavior.

The example shows:
![TransPortRuledDiagnostic](https://github.com/seppmail/ExchangeOnline-FAQ/assets/22378822/5540270b-44a5-4353-8506-acc77c934750)

1. It is the Standard diagnostic message
2. The Transport Rule name which is diagnosed
3. Sample condition "inOrganization" and the evaluation result
4. Transport Rule evaluation summary result ==> will the rule trigger

## Option 2: With data E-Mail file (no sender needed)

If you have an existing e-mail which fails in your environment you can store the e-mail as .EML file and use it with Test-Message

```powershell
$data = [System.IO.File]::ReadAllBytes('/Testmail.eml')
Test-Message -MessageFileData $data -SendReportTo 'someadmin@contoso.eu' -TransportRules
```

If the message data file contains a recipient, it can be overwritten with the -recipient parameter

## Things to know

- Check the rule priority. Check whether there is a triggered rule with a higher priority than the rule you are troubleshooting, whose action(s) can cause a problematic rule to not work as expected.
- Ensure ~30 minutes have passed since the last rule change.  It can take up to ~30 minutes for rule changes to take effect.
- Confirm the rule state. Rules must be in an Enabled state to be evaluated.  When rules are created, they are by default in a Disabled state.

Source: [original blog entry](https://techcommunity.microsoft.com/t5/exchange-team-blog/how-to-troubleshoot-exchange-online-transport-rules-using-the/ba-p/4000219)
