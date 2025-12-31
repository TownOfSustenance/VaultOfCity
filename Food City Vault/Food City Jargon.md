**LV-** Logistic Variant-> How we pack items on a pallet

**SV**- Sales Variant-> How the same item is sold in different ways
	Ex. Coke can be sold as a single, case, or pack of six

**Procs- Stored Procedures** -> pre-written, reusable block of SQL code that runs on the database server to perform a specific task whenever you call it.

**KVAT Components**-> [http://sup-ap-dwvgold1/kvatcomponents/inputs/#multi-dropdown](http://sup-ap-dwvgold1/kvatcomponents/inputs/#multi-dropdown)

**Location Guide** (possibly outdated- will update when more is known) -> ![[Pasted image 20251231121218.png]]

**Gold**-> The replacement for the HQPM Database; Holds retail, costs, deals, manages the items and everything related to them, their package information, the different SVs. How we ship it through LVs, etc. It also handles store assortment, item assortment, has commercial contract information, vendor information as it relates to the stores

**HQPM**-> Replaced by Gold same functionality but some redundancy dropped.

**SQL Agent Job**-> A job that can be scheduled to run an SSIS package or proc.

**GasBuddy**-> A company that Food City has a subscription with to display our gas prices hourly to draw in customers with up-to-date cheap gas. We have a program that pulls a CSV and uploads it to their amazon S3 bucket hourly.

**App Hub**-> An internal webpage used by store, district, and warehouse managers to display information and alerts tailored to their stores and warehouses.

[**Consolidated Screen**->](https://foodcity.sharepoint.com/:w:/r/sites/ConsolidatedScreen/Shared%20Documents/Requirements%20Gathering/KVAT-RFE002-BR-Consolidated%20Screen.docx?d=w94e45f8cfa004adea8375604bfeae2a2&csf=1&web=1&e=F55jMI) A consolidated screen containing a sortable and filterable view of retail, cost, and deal information pertaining to an item.

[**Symphony Wiki**](https://dev.azure.com/foodcity/Symphony%20ETL/_wiki/wikis/Symphony-ETL.wiki/3/Symphony-Wiki)-> Translations of the crazy tables from symphony that need to be transferred to gold- shows their equivalents

 **artcexr** -> external root code (UPC)- find its equivalent on item master- parent flag=1 is the currently active type of item- there are multiple because we switch vendors- USE FOR SSRS VNEDOR ITEM MISMATCH

**ParentFlag**-> Code on label, what they scan in stores

**artuc, artrac, artvl->** StoreAssortment (contains shippers so use for Vendor/Item Mismatch) or ItemAssortment can be used in it’s place

**tarprix**-> cost Master equivalent