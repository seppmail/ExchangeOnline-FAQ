# Why do i see mails with <somename>.onmicrosoft.com in the maillogs ?

Every M365 Exchange Online tenant has a *.onmicrosoft.com domain. This domain is automatically created during tenant initialization.
  
Therefor, it is the first domain in a new tenant and subsequently the intial "Tenant Default Accepted Domain (TDAD)".

After tenant initialization, customers add their custom domains to the tenant and finally a domain-list in a tenant may look similar to the one below:

- contosoeu.onmicrosoft.com
- contoso.eu
- contoso.de

Customers S-H-O-U-L-D set the TDAD away from the onmicrosoft.com domain to a custom domain, but in some cases this does not happen.

Unfortunately, under rare circumstances, exchange tenants then send out E-mails with the *.onmicrosoft.com domain as maildomain. Typical examples are:

- monitoring@contosoeu.onmicrosoft.com
- printer4711@contosoeu.onmicrosoft.com
- ... and so on.

Routing those E-Mails is a problem, because onmicrosoft.com is owned by Microsoft and using this mail-domain in a SEPPmail-enabled environment can lead to issues which are described below. Please read carefully, if you see E-Mails with *.onmicrosoft.com domains in your maillogs on the appliance of in the Log-Viewer in SEPPmail.cloud.

## Some notes about the MX Record

Every onmicrosoft.com domain gets an MX record automatically by Microsoft.
It´s not possible to view or modify this MX record in the Microsoft Admin Portal or via PowerShell.

## E-Mail Signatures

As .onmicrosoft.com is not registered in the MPKI, SEPPmail(.cloud) is not able to issue certificates for E-Mail addresses and sign E-Mails, si signing of E-Mails will fail. Depending on your SEPPmail ruleset, E-Mails with are marked to be signed, may be unsigned or rejected.

## Sending E-Mails to external recipients

Users are able to SEND E-Mails from an *.onmicrosoft.com domain to EXTERNAL recipients. As described above cryptographic actions may fail.

<bob@contosoeu.onmicrosoft.com> ==> <alice@fabrikam.com> unfortunately works.

A reply to this E-Mail will fail.

## Sending internal E-Mails

Users are able to SEND and RECEIVE e-mails from an *.onmicrosoft.com domain to INTERNAL.

bob@contosoeu.onmicrosoft.com ==> alice@contosoeu.onmicrosoft.com,alice@contoso.de

This does not affect SEPPmail(.cloud).

## Receiving E-Mails to *.onmicrosoft.com

Users cannot RECEIVE E-Mail to an *.onmicrosoft.com domain from outside. This is a Microsoft standard and cannot be overridden.

alice@fabrikam.com ==> bob@contosoeu.onmicrosoft.com ==> FAILS.

## Sending *.onmicrosoft.com E-Mails outbound through SEPPmail(.cloud)

E-Mails which are routed over SEPPmail.cloud are rejected bcs. *.onmicrosoft.com is not listed as managed domain in the SEPPmail.cloud.

bob@contosoeu.onmicrosoft.com ==> SEPPmail.cloud ==> alice@fabrikam.com FAILS.

## Receiving *.onmicrosoft.com E-Mails inbound through SEPPmail(.cloud)

Incoming *.onmicrosoft.com e-mails (from an SEPPmail-unknown external sender) need to be handled by SEPPmail.cloud as any other mail.

bob@contosoeu.onmicrosoft.com ==> SEPPmail.cloud ==> alice@fabrikam.com must work.

We hope that this explanaition helps to understand the wide range of issues and forces you to configure your accepted domains correctly.
