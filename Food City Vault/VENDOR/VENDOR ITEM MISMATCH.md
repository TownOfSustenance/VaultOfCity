[Requirements/Technical Design Doc](https://foodcity.visualstudio.com/Symphony%20ETL/_workitems/edit/11644) - Explains what the purpose of the report is

[Assigned Board Item](https://dev.azure.com/foodcity/Store%20Ops%20(Application%20Development)/_workitems/edit/30032/) - The board item associated to it's progress, assigned to Eli Combs

[Informative Board Item](https://foodcity.visualstudio.com/Symphony%20ETL/_workitems/edit/11644)-  Board item containing to the information you'll use for rewrites



## Code to Change
**Tip:** It's easier to view these in a notepad- copy and paste them

### 1.  assort_cost.sql

```SQL Oracle
  select a.artcexr  root_code, /*SAM.UPC*/
  pkstrucobj.get_desc(1,a.artcinr, 'KV') root_code_description, /*IM.itemDescription*/
    pkparpostes.get_postlibl(1,0,1032,a.artgest,'KV') category_manager, /*IM.CategoryManager*/
    decode(pkartuv.get_ARVCEXV(1,l.arlcinluvc) ,-1,null, pkartuv.get_ARVCEXV(1,l.arlcinluvc)) sv_code, /*Sam.SalesVariantId_Intneral*/
    l.arlcexvl LV_code, /*CM.LogisticVariantID_Internal*/
    pkfoudgene.get_CNUF(1,o.aracfin) assortment_vendor_number, /*VM.MasterVendorID,*/
    pkfoudgene.get_DescriptionFournisseur(1,o.aracfin) assortment_vendor_description, /*VM.VendorName,*/
    o.aranfilf assortment_adch, /*SAM.MainVendorAddressChain*/
    pkfouccom.get_NumContrat(1,o.araccin) assortment_commercial_contract, /*SAM.CommercialContractNumber,*/
    o.arasite assortment_location, /* SAM.LocationNetworkCode,*/
    o.araddeb assortment_start_date, /*Sam.StartDate*/
    o.aradfin assortment_end_date, /*Sam.EndDate*/
    o.arautil assortment_last_user /*SAM.ChangedBy*/
  From dmr.artuc o, dmr.artrac a, dmr.artvl l 
  where trunc(sysdate) <= o.aradfin
  and o.aratcde = 1
  AND o.aracinr = a.artcinr
  AND l.arlseqvl = o.araseqvl 
  AND NOT EXISTS (SELECT 1 
				FROM tarprix t/*PROBABLY COST MASTER*/, reseau r 
				WHERE t.tapcfin/*CM.MasterVendorCode*/ = o.aracfin /*SAM. MainVendorID_Internal*/
				AND t.tapccin/*CM.CommercialContractID_Internal*/ = o.araccin /*SAM. CommercialContractID_Internal*/
				AND t.tapseqvl/*CM.LogisticVariantID_Internal*/ = o.araseqvl /*SAM.LogisticVariantID_Internal*/
				AND greatest(trunc(sysdate), o.araddeb/*SAM.StartDate*/)>= t.tapddeb /*CM.StartDate*/
				AND least(o.aradfin /*SAM.EndDate*/, trunc(sysdate)) <= t.tapdfin   /*CM.EndDate*/
				AND r.respere /*SAM.GoldLocationNetworkCode*/= t.tapsite /*LocationNetworkCode*/
				AND r.resresid = '1' 
				AND  r.ressite = o.arasite 
				AND greatest(trunc(sysdate), o.araddeb /*SAM.StartDate*/)>= r.resddeb 
				AND least(o.aradfin /*SAM.EndDate*/, trunc(sysdate)) <= r.resdfin);
```

### CURRENTLY REVISED VERSION: (IN-PROGRESS)

```SQL SSMS
  SELECT 
	SAM.UPC,
	IM.ItemDescription,
	IM.CategoryManager,
	SAM.SalesVariantID_Internal,
	CM.LogisticVariantID_Internal,
	VM.VendorName,
	VM.MasterVendorID,
	SAM.MainVendorAddressChain,
	SAM.CommercialContractNumber,
	SAM.LocationNetworkCode,
	SAM.StartDate,
	SAM.EndDate,
	SAM.ChangedBy
  FROM gmm.StoreAssortmentMaster SAM
	LEFT JOIN gmm.ItemMaster IM
		ON SAM.UPC = IM.SourceUPC
	LEFT JOIN gmm.CostMaster CM
		ON SAM.LogisticVariantID_Internal = CM.LogisticVariantID_Internal
	LEFT JOIN gmm.VendorMaster VM
		ON SAM.MainVendorID_Internal = VM.MasterVendorID
	LEFT JOIN gmm.ShipperMaster SM
		ON SAM.CommercialContractNumber = SM.CommercialContractNumber
	WHERE GETDATE() <= SAM.EndDate
		AND IM.ParentFlag = 1
		AND SAM.OrderableFlag = 1
		AND NOT EXISTS (
			SELECT 1
			FROM gmm.CostMaster CM
			WHERE CM.MasterVendorCode = SAM.MainVendorID_Internal
			AND CM.CommercialContractID_Internal = SAM.CommercialContractID_Internal
			AND CM.LogisticVariantID_Internal = SAM.LogisticVariantID_Internal
			AND GREATEST(GETDATE(), SAM.StartDate) >= CM.StartDate
			AND LEAST(SAM.EndDate, GETDATE()) <= CM.EndDate
			AND SAM.GoldLocationNetworkCode = CM.LocationNetworkCode
			/*MISSING EQUIVALENTS
				AND r.resresid = '1' 
				AND  r.ressite = o.arasite 
				AND greatest(trunc(sysdate), o.araddeb /*SAM.StartDate*/)>= r.resddeb 
				AND least(o.aradfin /*SAM.EndDate*/, trunc(sysdate)) <= r.resdfin);
			*/
		);
```


<hr>

### 2. cost_assort.sql

```SQL Oracle
  select r.artcexr root_code,
  pkstrucobj.get_desc(1,r.artcinr, 'KV') root_code_description,
  pkparpostes.get_postlibl(1,0,1032,r.artgest,'KV') category_manager,
  decode(pkartuv.get_ARVCEXV(1,l.arlcinluvc) ,-1,null, pkartuv.get_ARVCEXV(1,l.arlcinluvc)) sv_code,
  l.arlcexvl LV_code,
  pkfoudgene.get_CNUF(1,t.tapcfin) cost_vendor_number, 
  pkfoudgene.get_DescriptionFournisseur(1,t.tapcfin) cost_vendor_description,
  
  pkfouccom.get_NumContrat(1,t.tapccin) cost_commercial_contract, 
  t.tapddeb cost_start_date,
  t.tapdfin cost_end_date, 
  t.taputil cost_last_user
  From tarprix t, artrac r, artvl l
  where trunc(sysdate) <= t.tapdfin 
  and not exists (select 1 from artuc a where  a.aracfin = t.tapcfin and a.araccin = t.tapccin and  a.araddeb <= greatest(trunc(sysdate), t.tapddeb) and a.aradfin>= t.tapdfin and t.tapseqvl = a.araseqvl
  and a.arasite not in ( select n.ressite From reseau n where n.resresid = '1' and n.respere = t.tapsite and  n.resddeb<=greatest(trunc(sysdate), t.tapddeb) and n.resdfin>= t.tapdfin))
  and r.artcinr = t.tapcinr
  and l.arlseqvl = t.tapseqvl
  ; 
  
```

### CURRENTLY REVISED VERSION: (IN-PROGRESS)

```SQL
SELECT
	SAM.UPC,
	IM.ItemDescription,
	IM.CategoryManager,
	SAM.SalesVariantID_Internal,
	CM.LogisticVariantID_Internal,
	CM.MasterVendorCode,
	VM.VendorName, 
	CM.CommercialContractID_Internal,
	CM.StartDate,
	CM.EndDate,
	CM.LastUser
FROM gmm.StoreAssortmentMaster SAM
	LEFT JOIN gmm.ItemMaster IM
		ON SAM.UPC = IM.SourceUPC
	LEFT JOIN gmm.CostMaster CM
		ON SAM.LogisticVariantID_Internal = CM.LogisticVariantID_Internal
	LEFT JOIN gmm.VendorMaster VM
		ON SAM.MainVendorID_Internal = VM.MasterVendorID
	WHERE GETDATE() <= CM.EndDate
		AND NOT EXISTS(
			SELECT 1
			FROM gmm.StoreAssortmentMaster SAM
				WHERE SAM.MainVendorID_Internal = CM.MasterVendorCode
				AND SAM.CommercialContractID_Internal = CM.CommercialContractID_Internal
				AND SAM.StartDate <= GREATEST(GETDATE(), CM.StartDate)
				AND SAM.EndDate >= CM.EndDate
				AND CM.LogisticVariantID_Internal = SAM.LogisticVariantID_Internal
				/*MISSING 
				AND a.arasite
							NOT in ( select n.ressite 
					FROM reseau n /*DUNNO WHERE THIS GUY AT*/
					WHERE n.resresid = '1' 
					AND n.respere = t.tapsite /*CM.LocationNetworkCode*/
					and  n.resddeb<=greatest(trunc(sysdate), t.tapddeb) /*Cm.startDate*/
					AND n.resdfin>= t.tapdfin/*CM.EndDate*/))
				*/
				AND SAM.RootCode = CM.RootCode
				/*AND CM.LogisiticVariantID_Internal = MISSING*/

		);
```


<hr>

### 3. deal_assort.sql

```SQL Oracle
select a.artcexr root_code,
pkstrucobj.get_desc(1,a.artcinr, 'KV') root_code_description,
pkparpostes.get_postlibl(1,0,1032,a.artgest,'KV') category_manager,
decode(pkartuv.get_ARVCEXV(1,l.arlcinluvc) ,-1,null, pkartuv.get_ARVCEXV(1,l.arlcinluvc)) sv_code,
l.arlcexvl LV_code,
  pkfoudgene.get_CNUF(1,h.trecfin) deal_vendor_number, 
  pkfoudgene.get_DescriptionFournisseur(1,h.trecfin) deal_vendor_description,
    pkfouccom.get_NumContrat(1,h.treccin) deal_commercial_contract, 
 e.trxddeb deal_start_date,
  e.trxdfin deal_end_date, 
  e.trxutil deal_last_user
From taremise h, tarinco d, tarexar e, artrac a, artvl l
where e.trxdfin >= trunc(sysdate)
and h.trecinrem = d.ticcinrem
and d.ticcinrem = e.trxcinrem
and a.artcinr = e.trxcinr
and l.arlseqvl = e.trxseqvl
and not exists (select 1 from artuc o where o.araseqvl = e.trxseqvl and o.aracfin = h.trecfin and o.araccin = h.treccin and o.araddeb <=greatest(trunc(sysdate),e.trxddeb) and e.trxdfin <= o.aradfin)
```

### CURRENTLY REVISED VERSION: (N/A)