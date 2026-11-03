# **Feature list**

Localization for Ireland

<figure>
<img src="images/Localization-for-Ireland_Feature-List/image1.png" style="width:2.25in;height:0.57686in" alt="A blue and black logo Description automatically generated" />
<figcaption><p>Microsoft Dynamics 365 Business Central</p></figcaption>
</figure>

companial.com

# Feature list

Companial’s Localization for Ireland is designed to support Microsoft Dynamics 365 Business Central users operating in Ireland with core missing features for local compliance and ease of use. The table below provides an overview of the currently available features included in the solution as of version 2.0.0.0.

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 30%" />
<col style="width: 54%" />
</colgroup>
<thead>
<tr>
<th style="text-align: center;">Area</th>
<th style="text-align: center;">Functionality</th>
<th style="text-align: center;">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: center;"><blockquote>
<p>Master Data &amp; User Experience</p>
</blockquote></td>
<td style="text-align: center;">Dynamic Post Code Caption and Eircode Support</td>
<td style="text-align: left;">Dynamically adapts postal code field captions throughout Business Central based on country/region configuration. For Ireland, users can work with “Eircode” terminology across customer, vendor, contact, document, banking, company, and other records, improving local usability and consistency across the system.</td>
</tr>
<tr>
<td rowspan="3" style="text-align: center;"><blockquote>
<p>MAINTAINING VAT</p>
</blockquote></td>
<td style="text-align: center;">VAT Reverse Charge</td>
<td style="text-align: left;">Supports Irish VAT reverse charge scenarios through VAT posting setup and VAT statement configuration for goods and services. The functionality helps ensure correct treatment of purchase and sales VAT entries, including scenarios where foreign-currency transactions require VAT calculation based on local-currency rules.</td>
</tr>
<tr>
<td style="text-align: center;">VAT Statement for Ireland</td>
<td style="text-align: left;">Provides an Irish VAT statement structure aligned to the VAT3 return, including dedicated VAT business and product posting groups and box-based setup for T1, T2, E1, E2, ES1, and ES2 values. This creates the accounting foundation used for Irish VAT reporting and return preparation.</td>
</tr>
<tr>
<td style="text-align: center;">Accounting for VAT on Moneys Received</td>
<td style="text-align: left;">Supports Irish cash-basis VAT requirements by allowing VAT to be reported when customer payments are received rather than when invoices are issued. This helps businesses align VAT reporting with cash collection and manage payment-based VAT obligations within Business Central.</td>
</tr>
<tr>
<td rowspan="4" style="text-align: center;"><blockquote>
<p>VAT Reporting &amp; Compliance</p>
</blockquote></td>
<td style="text-align: center;">VAT3 Return Export</td>
<td style="text-align: left;">Enables preparation and export of VAT3 return data in XML format for submission to the Revenue Online Service (ROS). The functionality extends standard VAT reporting with Ireland-specific return handling, including Standard, Corrective, and Supplementary return types, configurable filing frequency, language selection, and export tracking on VAT entries. Generated XML files are stored as document attachments for traceability.</td>
</tr>
<tr>
<td style="text-align: center;">VAT3 Return Mapping</td>
<td style="text-align: left;">Includes configurable VAT3 Return Mapping that links Irish VAT3 box numbers to the XML elements expected by ROS. The setup supports one-to-one mapping validation and an auto-fill action for the standard Irish mapping model, helping ensure the exported return uses the correct structure.</td>
</tr>
<tr>
<td style="text-align: center;">VAT Audit Report</td>
<td style="text-align: left;">Provides a structured CSV export of VAT-related data for audit preparation in Ireland. The report supports export of customer and vendor master data, open payments, late invoicing scenarios, and VAT entries for the reviewed period, helping organizations provide clear and extractable records when requested by auditors.</td>
</tr>
<tr>
<td style="text-align: center;">VAT Entry Exception Report</td>
<td style="text-align: left;">Adds a VAT Entry Exception Report for quality control and review of VAT data before return submission or audit preparation. The report highlights exceptions only and can validate VAT base discount, manual VAT differences, VAT calculation type mismatches, and VAT rate differences against configurable thresholds.</td>
</tr>
<tr>
<td style="text-align: center;"><blockquote>
<p>SALES Documents</p>
</blockquote></td>
<td style="text-align: center;">Customer VAT Registration Number on Sales Documents</td>
<td style="text-align: left;">Displays the customer VAT registration number on printed sales invoices and credit memos using Irish-specific Word layouts. This helps support document requirements for B2B transactions while keeping the printing process aligned with standard sales document handling in Business Central.</td>
</tr>
<tr>
<td style="text-align: center;"><blockquote>
<p>Controls &amp; Validation</p>
</blockquote></td>
<td style="text-align: center;">Mandatory VAT3 Export Before Submission</td>
<td style="text-align: left;">Supports controlled VAT return processing through an optional mandatory-export setting. When enabled, users must generate the VAT3 XML file before a VAT return can be marked as submitted, helping enforce a more disciplined reporting process.</td>
</tr>
<tr>
<td rowspan="3" style="text-align: center;"><blockquote>
<p>Controls &amp; Validation</p>
</blockquote></td>
<td style="text-align: center;">VAT Entry Export Tracking</td>
<td style="text-align: left;">Tracks which VAT return each VAT entry has been exported in, including support for supplementary return tracking. This improves visibility and helps prevent accidental double-reporting when working with standard, corrective, and supplementary VAT3 returns.</td>
</tr>
<tr>
<td style="text-align: center;">Corrective and Supplementary Return Handling</td>
<td style="text-align: left;">Supports corrective and supplementary VAT3 return workflows in addition to standard returns. The functionality includes rules for linking returns, controlling correction chains, and filtering transactions so that additional returns only include relevant entries.</td>
</tr>
<tr>
<td style="text-align: center;">Flexible VAT3 Validation</td>
<td style="text-align: left;">Allows organizations to use either Ireland-specific VAT3 controls or standard Business Central VAT reporting processes. This provides greater flexibility for businesses with different reporting arrangements while keeping VAT configuration within a familiar Business Central experience.</td>
</tr>
<tr>
<td style="text-align: center;"><blockquote>
<p>Company Data &amp; Administration</p>
</blockquote></td>
<td style="text-align: center;">Companies Registration Office Integration</td>
<td style="text-align: left;">Connects Business Central with Ireland’s Companies Registration Office via the CORE system, allowing users to look up official company and business-name information directly from customer, vendor, and contact records. This reduces manual data entry, improves master-data accuracy, and provides controls for reviewing updates and tracking lookup activity.</td>
</tr>
<tr>
<td style="text-align: center;"><blockquote>
<p>VAT Relief &amp; Sales</p>
</blockquote></td>
<td style="text-align: center;">Section 56 Authorization and VAT Processing</td>
<td style="text-align: left;">Supports the management of Section 56 customer authorizations and the zero-rating of qualifying goods and services. Authorization details and validity periods are maintained in Business Central, while built-in controls help apply the appropriate VAT treatment consistently during sales processing.</td>
</tr>
</tbody>
</table>
