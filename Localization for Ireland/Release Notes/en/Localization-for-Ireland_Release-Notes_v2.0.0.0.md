Localization for Ireland

<figure>
<img src="../../Feature%20List/en/images/Localization-for-Ireland_Feature-List/image1.png" style="width:2.25in;height:0.57686in" alt="A blue and black logo Description automatically generated" />
<figcaption><p>Microsoft Dynamics 365 Business Central</p></figcaption>
</figure>

companial.com

## Contents

- [Currently in R&D](#currently-in-rd)
- [Version 2.0.0.0 – September 16th, 2026](#version-2000--september-16th-2026)
  - [Documentation](#documentation)
  - [Prerequisites](#prerequisites)
  - [Changes](#changes)

# Currently in R&D

The Companial Partner community plays a key role in shaping the future development of Localization for Ireland. Product investments and priorities are influenced by Partner feedback, market requirements, customer needs, and evolving regulatory requirements. The following areas are currently under active research and development and represent our present focus. Scope and priorities may evolve as new feedback and requirements emerge.

- Enhanced Reporting Requirements support

- Integration for Eircode look-up

- Relevant Contracts Tax support

- Professional Services Withholding tax support

If you or your Customers have feedback or feature suggestions on the Irish Localisation, please do not hesitate to reach out via <localizationIE@companial.com>.

# Version 2.0.0.0 – September 16<sup>th</sup>, 2026

This release introduces significant new capabilities that help Irish businesses improve company data accuracy and manage important VAT requirements directly in Business Central. Users can retrieve company information from the Companies Registration Office CORE service, configure accounting for VAT on moneys received, and manage Section 56 authorizations and zero-rated sales. Supporting documentation and setup materials have also been updated to support the new functionalities.

## Documentation

*The following documentation has been updated and is available on the Companial Partner Portal:*

- User manual updated to v2.0

- Testing manual updated to v2.0

- Rapid Start package updated to v2.0

- Feature list updated to v2.0

## Prerequisites

- Business Central version 28

## Changes

- **Integration with Companies Registration Office CORE system for business details look-up**

The CRO CORE integration is available from CRO Integration Setup and Assisted Setup. Users can configure the CRO service URL, API key, email address, logging level, default search type, name-match type, entity type, result limit, and overwrite behavior.

Customer, Vendor, and Contact cards include CRO lookup actions. Users can search by registration number or company name, select a result, and apply CRO data according to the configured field mappings and overwrite settings. CRO Integration Setup provides actions for Field Mapping, Log Entries, and Test Connection.

CRO field mappings display the CRO field, Business Central table and field details, mandatory status, and enabled status. The Reset to Default action replaces existing mappings with the default configuration. CRO requests and responses are available through CRO Log Entries.

- **Accounting for VAT on moneys received support**

User manual v2.0 introduces support for configuring Business Central to support application of accounting for VAT on moneys received in Ireland. In relation to this, the application is updated to support VAT3 Return marking including VAT entries whose Unrealized Amount is zero. VAT entries with unpaid unrealized VAT remain available for a later VAT3 Return.

- **Section 56 zero-ratings of goods and services (VAT 56B) support**

Customer Cards include a Section 56 Authorization section with Certificate No., Valid From, and Valid To fields. When Section 56 Authorization is turned off, the certificate number and validity dates are cleared. Valid From must be earlier than Valid To.

Sales Orders and Sales Invoices display the customer’s Section 56 authorization details. These details are copied when the Sell-to Customer No. is entered and remain available on posted sales invoices.

VAT Setup includes the VAT56B Default Product Posting Group and Update VAT56B Posting Group on Release settings. The configured VAT product posting group must have VAT Posting Setup records with VAT % set to zero. When a sales document for a customer with a valid Section 56 authorization is released, the sales lines are updated automatically or after confirmation, according to the VAT Setup setting.
