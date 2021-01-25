---
title: '[!DNL Asset Compute Service] HTTP-API'
description: '[!DNL Asset Compute Service] HTTP-API om aangepaste toepassingen te maken.'
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 1%

---


# [!DNL Asset Compute Service] HTTP-API  {#asset-compute-http-api}

Het gebruik van de API is beperkt tot ontwikkelingsdoeleinden. De API wordt als context verstrekt wanneer het ontwikkelen van douanetoepassingen. [!DNL Adobe Experience Manager] als  [!DNL Cloud Service] gebruikt de API om de verwerkingsgegevens door te geven aan een aangepaste toepassing. Zie [Elementmicroservices en verwerkingsprofielen gebruiken](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) voor meer informatie.

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met  [!DNL Experience Manager] als  [!DNL Cloud Service]een.

Elke client van de HTTP-API [!DNL Asset Compute Service] moet deze flow op hoog niveau volgen:

1. Een client wordt geleverd als [!DNL Adobe Developer Console]-project in een IMS-organisatie. Elke afzonderlijke client (systeem of omgeving) vereist een eigen afzonderlijk project om de gegevensstroom van de gebeurtenis te scheiden.

1. Een client genereert een toegangstoken voor de technische account met de [JWT (serviceaccount)-verificatie](https://www.adobe.io/authentication/auth-methods.html).

1. Een client roept [`/register`](#register) slechts eenmaal aan om de journaal-URL op te halen.

1. Een client roept [`/process`](#process-request) aan voor elk element waarvoor het uitvoeringen wil genereren. De vraag is asynchroon.

1. Een client pollt het dagboek regelmatig naar [receive-gebeurtenissen](#asynchronous-events). Het ontvangt gebeurtenissen voor elke gevraagde vertoning wanneer de vertoning met succes wordt verwerkt (`rendition_created` gebeurtenistype) of als er een fout (`rendition_failed` gebeurtenistype) is.

Met de [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client)-module kunt u de API in Node.js-code eenvoudig gebruiken.

## Verificatie en autorisatie {#authentication-and-authorization}

Voor alle API&#39;s is verificatie van toegangstoken vereist. De verzoeken moeten de volgende kopballen plaatsen:

1. `Authorization` header met token voor toonder, dit is het token voor een technische account, ontvangen via  [JWT ](https://www.adobe.io/authentication/auth-methods.html) Exchange van het project Adobe Developer Console. Het [bereik](#scopes) wordt hieronder beschreven.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` met de IMS-organisatie-id.

1. `x-api-key` met de client-id van het  [!DNL Adobe Developers Console] project.

### Scopes {#scopes}

Verzeker het volgende werkingsgebied voor het toegangstoken:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Hiervoor moet het [!DNL Adobe Developer Console]-project worden geabonneerd op `Asset Compute`-, `I/O Events`- en `I/O Management API`-services. De uitsplitsing van individuele scènes is:

* Basis
   * bereik: `openid,AdobeID`

* asset compute
   * metareaal: `asset_compute_meta`
   * bereik: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Gebeurtenissen
   * metareaal: `event_receiver_api`
   * bereik: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] Beheer-API
   * metareaal: `ent_adobeio_sdk`
   * bereik: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registratie {#register}

Elke client van het [!DNL Asset Compute service] - een uniek [!DNL Adobe Developer Console]-project dat op de service is geabonneerd - moet [register](#register-request) hebben voordat verwerkingsverzoeken worden ingediend. De registratiestap retourneert het unieke gebeurtenisjournaal dat vereist is om de asynchrone gebeurtenissen van de uitvoering op te halen.

Aan het eind van zijn levenscyclus, kan een cliënt [unregister](#unregister-request).

### Verzoek {#register-request} registreren

Met deze API-aanroep wordt een [!DNL Asset Compute]-client ingesteld en wordt de URL van het gebeurtenisdagboek weergegeven. Dit is een epidemische bewerking die slechts één keer voor elke client hoeft te worden uitgevoerd. Het kan opnieuw worden geroepen om het dagboek URL terug te winnen.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/register` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet leeg zijn. |

### Reactie registreren {#register-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Of het zelfde als `X-Request-Id` verzoekkopbal of uniek geproduceerde. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `journal`-, `ok`- en/of `requestId`-velden. |

De HTTP-statuscodes zijn:

* **200 succesvol**: Wanneer het verzoek succesvol is. Het bevat de `journal` URL die over om het even welke resultaten van de asynchrone verwerking wordt gemeld die via `/process` (als gebeurtenistype `rendition_created` wanneer succesvol, of `rendition_failed` wanneer mislukt) wordt teweeggebracht.

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek geen geldige  [authentificatie](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: zich voordoet wanneer de aanvraag geen geldige  [vergunning](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **Te veel verzoeken**: treedt op wanneer het systeem door deze cliënt of anders wordt overbelast. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.
* **4xx-fout**: Als er een andere clientfout is opgetreden en registratie is mislukt. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx-fout**: treedt op wanneer er een andere serverfout is opgetreden en de registratie is mislukt. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Registratieverzoek {#unregister-request} ongedaan maken

Met deze API-aanroep wordt de registratie van een [!DNL Asset Compute]-client ongedaan gemaakt. Hierna is het niet meer mogelijk `/process` aan te roepen. Als u de API-aanroep voor een niet-geregistreerde client of een nog te registreren client gebruikt, wordt een `404`-fout geretourneerd.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/unregister` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Leeg. |

### Reactie {#unregister-response} verwijderen

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Of het zelfde als `X-Request-Id` verzoekkopbal of uniek geproduceerde. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok`- en `requestId`-velden. |

De statuscodes zijn:

* **200 succesvol**: treedt op wanneer de registratie en het dagboek worden gevonden en verwijderd.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek geen geldige  [authentificatie](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: zich voordoet wanneer de aanvraag geen geldige  [vergunning](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **Niet gevonden**: komt voor wanneer er geen huidige registratie voor de bepaalde geloofsbrieven is.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **Te veel verzoeken**: treedt op wanneer het systeem wordt overbelast. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.

* **4xx-fout**: komt voor wanneer er een andere cliëntfout was en unregister ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx-fout**: treedt op wanneer er een andere serverfout is opgetreden en de registratie is mislukt. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Procesaanvraag {#process-request}

Met de bewerking `process` wordt een taak verzonden die een bronelement omzet in meerdere uitvoeringen, op basis van de instructies in het verzoek. Meldingen over een geslaagde voltooiing (gebeurtenistype `rendition_created`) of eventuele fouten (gebeurtenistype `rendition_failed`) worden naar een gebeurtenisdagboek verzonden dat moet worden opgehaald met [/register](#register) één keer voordat een aantal `/process`-aanvragen wordt ingediend. Onjuist gevormde verzoeken ontbreken onmiddellijk met een 400 foutencode.

Er wordt verwezen naar binaire URL&#39;s, zoals vooraf ondertekende URL&#39;s voor Amazon AWS S3 of Azure Blob Storage SAS URL&#39;s, voor zowel het lezen van het `source`-element (`GET` URL&#39;s) als het schrijven van de uitvoeringen (`PUT` URL&#39;s). De klant is verantwoordelijk voor het genereren van deze vooraf ondertekende URL&#39;s.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/process` |
| MIME-type | `application/json` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet de JSON-indeling van het procesverzoek hebben, zoals hieronder wordt beschreven. Er worden instructies gegeven over welk element moet worden verwerkt en welke uitvoeringen moeten worden gegenereerd. |

### Verzoek om verwerking JSON {#process-request-json}

De aanvraagtekst van `/process` is een JSON-object met dit schema op hoog niveau:

```json
{
    "source": "",
    "renditions" : []
}
```

De beschikbare velden zijn:

| Naam | Type | Beschrijving | Voorbeeld |
|--------------|----------|-------------|---------|
| `source` | `string` | URL van het bronelement dat moet worden verwerkt. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Beschrijf het bronelement dat moet worden verwerkt. Zie de beschrijving van [Bronobjectvelden](#source-object-fields) hieronder. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Uitvoeringen die moeten worden gegenereerd op basis van het bronbestand. Elk weergaveobject ondersteunt [vertoningsinstructie](#rendition-instructions). Vereist. | `[{ "target": "https://....", "fmt": "png" }]` |

De `source` kan een `<string>` zijn die als URL wordt gezien of het kan een `<object>` met een extra gebied zijn. De volgende varianten zijn vergelijkbaar:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Bronobjectvelden {#source-object-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-----------|----------|-------------|---------|
| `url` | `string` | URL van het bronelement dat moet worden verwerkt. Vereist. | `"http://example.com/image.jpg"` |
| `name` | `string` | Bestandsnaam van bronelement. De bestandsextensie in de naam kan worden gebruikt als er geen MIME-type kan worden gedetecteerd. Heeft voorrang op bestandsnaam in URL-pad of bestandsnaam in `content-disposition`-header van binaire bron. De standaardwaarde is &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Bestandsgrootte van bronelement in bytes. Neemt belangrijkheid over `content-length` kopbal van het binaire middel. | `10234` |
| `mimetype` | `string` | MIME-type voor bronelementbestand. Neemt belangrijkheid over de `content-type` kopbal van het binaire middel. | `"image/jpeg"` |

### Een volledig `process` aanvraagvoorbeeld {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Procesreactie {#process-response}

Het `/process` verzoek keert onmiddellijk met een succes of een mislukking terug die op de basisverzoekbevestiging wordt gebaseerd. De werkelijke verwerking van elementen gebeurt asynchroon.

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Of het zelfde als `X-Request-Id` verzoekkopbal of uniek geproduceerde. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok`- en `requestId`-velden. |

Statuscodes:

* **200 succesvol**: Als het verzoek is verzonden. Response JSON bevat `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Ongeldig verzoek**: Als de aanvraag onjuist is samengesteld, ontbreken bijvoorbeeld vereiste velden in de aanvraag-JSON. Response JSON bevat `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Niet-geautoriseerd**: Wanneer de aanvraag geen geldige  [verificatie](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.
* **403 Verboden**: Wanneer het verzoek geen geldige  [vergunning](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.
* **Te veel verzoeken**: Wanneer het systeem door deze client of in het algemeen wordt overbelast. De cliënten kunnen met [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff) opnieuw proberen. Het lichaam is leeg.
* **4xx-fout**: Wanneer er een andere clientfout is opgetreden. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx-fout**: Als er een andere serverfout is opgetreden. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

De meeste cliënten zijn waarschijnlijk geneigd om het nauwkeurige zelfde verzoek met [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff) op om het even welke fout *behalve* configuratiekwesties zoals 401 of 403, of ongeldige verzoeken zoals 400 opnieuw te proberen. Afgezien van een reguliere tariefbeperking via 429 reacties, kan een tijdelijke onderbreking of beperking van de service leiden tot 5x fouten. Het zou dan raadzaam zijn om na een bepaalde periode opnieuw te proberen.

Alle JSON-reacties (indien aanwezig) bevatten de `requestId` die dezelfde waarde heeft als de `X-Request-Id`-header. Het wordt aanbevolen om van de koptekst te lezen, aangezien deze altijd aanwezig is. `requestId` is ook teruggekeerd in alle gebeurtenissen met betrekking tot verwerkingsverzoeken zoals `requestId`. Clients mogen geen aanname maken over de opmaak van deze tekenreeks, het is een ondoorzichtige tekenreeks-id.

## Aanmelden bij nabewerking {#opt-in-to-post-processing}

De [Asset compute-SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt een set basisopties voor nabewerking van afbeeldingen. Aangepaste workers kunnen zich expliciet aanmelden bij naverwerking door het veld `postProcess` voor het weergaveobject in te stellen op `true`.

De ondersteunde gebruiksgevallen zijn:

* Een vertoning uitsnijden naar een rechthoek waarvan de grenzen worden gedefinieerd door crop.w, crop.h, crop.x en crop.y. Deze wordt gedefinieerd door `instructions.crop` in het weergaveobject.
* Wijzig de grootte van afbeeldingen met breedte, hoogte of beide. Deze wordt gedefinieerd door `instructions.width` en `instructions.height` in het weergaveobject. Als u alleen de breedte of hoogte wilt wijzigen, stelt u slechts één waarde in. Met Compute Service behoudt u de hoogte-breedteverhouding.
* Stel de kwaliteit in voor een JPEG-afbeelding. Deze wordt gedefinieerd door `instructions.quality` in het weergaveobject. De beste kwaliteit wordt aangegeven met `100` en kleinere waarden geven een lagere kwaliteit aan.
* Maak geïnterlinieerde afbeeldingen. Deze wordt gedefinieerd door `instructions.interlace` in het weergaveobject.
* Stel DPI in om de gerenderde grootte aan te passen voor publicatie op het bureaublad door de schaal aan te passen die op de pixels wordt toegepast. De eigenschap wordt gedefinieerd door `instructions.dpi` in het weergaveobject om de dpi-resolutie te wijzigen. Als u de afbeelding echter wilt vergroten of verkleinen zodat deze dezelfde grootte heeft bij een andere resolutie, gebruikt u de `convertToDpi`-instructies.
* Wijzig de grootte van de afbeelding zodanig dat de weergegeven breedte of hoogte gelijk blijft aan het origineel bij de opgegeven doelresolutie (DPI). Deze wordt gedefinieerd door `instructions.convertToDpi` in het weergaveobject.

## Watermerkelementen {#add-watermark}

De [Asset compute-SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt het toevoegen van een watermerk aan PNG-, JPEG-, TIFF- en GIF-afbeeldingsbestanden. Het watermerk wordt toegevoegd na de vertoningsinstructies in het `watermark`-object op de vertoning.

Watermerken worden uitgevoerd tijdens de nabewerking van de vertoning. Voor watermerkelementen kiest de aangepaste worker [ná verwerking](#opt-in-to-post-processing) door het veld `postProcess` op het weergaveobject in te stellen op `true`. Als de worker niet de optie Weigeren inschakelt, wordt het watermerk niet toegepast, zelfs niet als het watermerkobject is ingesteld op het weergaveobject in de aanvraag.

## Vertoningsinstructies {#rendition-instructions}

Dit zijn de beschikbare opties voor de `renditions`-array in [/process](#process-request).

### Algemene velden {#common-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | De doelindeling van uitvoeringen kan ook `text` zijn voor tekstextractie en `xmp` voor het extraheren van XMP metagegevens als xml. Zie [ondersteunde indelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL van een [aangepaste toepassing](develop-custom-application.md). Moet een `https://` URL zijn. Als dit veld aanwezig is, wordt de vertoning gemaakt door een aangepaste toepassing. Een ander setveld voor uitvoering wordt vervolgens gebruikt in de aangepaste toepassing. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL waarnaar de gegenereerde uitvoering moet worden geüpload met HTTP-PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Vooraf ondertekende URL met meerdere delen uploadgegevens voor de gegenereerde uitvoering. Dit is voor [AEM/Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) met dit [uploadgedrag voor meerdere delen](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fields:<ul><li>`urls`: array van tekenreeksen, één voor elke vooraf ondertekende deel-URL</li><li>`minPartSize`: minimumgrootte voor één onderdeel = url</li><li>`maxPartSize`: de maximumgrootte voor één onderdeel = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optionele gereserveerde ruimte die door de client wordt beheerd en net als weergavegebeurtenissen wordt doorgegeven. Staat cliënten toe om douaneinformatie toe te voegen om vertoningsgebeurtenissen te identificeren. Moet niet worden gewijzigd of vertrouwd op in douanetoepassingen, aangezien de cliënten vrij zijn om dit op elk ogenblik te veranderen. | `{ ... }` |

### Vertoningsspecifieke velden {#rendition-specific-fields}

Zie [Ondersteunde bestandsindelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) voor een lijst met momenteel ondersteunde bestandsindelingen.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `*` | `*` | Geavanceerde aangepaste velden kunnen worden toegevoegd die een [aangepaste toepassing](develop-custom-application.md) begrijpt. |  |
| `embedBinaryLimit` | `number` in bytes | Als deze waarde is ingesteld en het bestand van de vertoning kleiner is dan deze waarde, wordt de vertoning ingesloten in de gebeurtenis die wordt verzonden zodra de renditie is voltooid. De maximaal toegestane grootte voor insluiten is 32 kB (32 x 1024 bytes). Als een vertoning groter is dan de `embedBinaryLimit`-limiet, wordt deze op een locatie in de cloudopslag geplaatst en wordt deze niet ingesloten in de gebeurtenis. | `3276` |
| `width` | `number` | Breedte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
| `height` | `number` | Hoogte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
|  |  | De verhouding blijft altijd behouden als: <ul> <li> Zowel `width` als `height` worden gespecificeerd, dan past het beeld in het formaat terwijl het handhaven van aspectverhouding </li><li> Alleen `width` of alleen `height` is opgegeven, gebruikt de resulterende afbeelding de corresponderende dimensie terwijl de hoogte-breedteverhouding behouden blijft</li><li> Als noch `width` noch `height` is opgegeven, wordt de oorspronkelijke pixelgrootte van de afbeelding gebruikt. Het hangt van het brontype af. Voor sommige indelingen, zoals PDF-bestanden, wordt een standaardgrootte gebruikt. Er kan een maximumgrootte zijn.</li></ul> |  |
| `quality` | `number` | Geef jpeg-kwaliteit op in het bereik van `1` tot `100`. Alleen van toepassing op afbeeldingsuitvoeringen. | `90` |
| `xmp` | `string` | Wordt alleen gebruikt door XMP terugverwijzing van metagegevens. Het is XMP base64 gecodeerd om terug te schrijven naar de opgegeven uitvoering. |  |
| `interlace` | `bool` | Maak geïnterlinieerde PNG- of GIF-bestanden of progressieve JPEG door deze in te stellen op `true`. Dit heeft geen invloed op andere bestandsindelingen. |  |
| `jpegSize` | `number` | De grootte van het JPEG-bestand in bytes benaderen. Hiermee wordt elke `quality`-instelling genegeerd. Heeft geen effect op andere indelingen. |  |
| `dpi` | `number` or `object` | Stel de DPI voor x en y in. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. Dit heeft geen invloed op de afbeelding zelf. | `96` of  `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` of  `object` | x- en y-DPI-waarden opnieuw samplen terwijl de fysieke grootte behouden blijft. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. | `96` of  `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lijst met bestanden die moeten worden opgenomen in het ZIP-archief (`fmt=zip`). Elk item kan een URL-tekenreeks zijn of een object met de velden:<ul><li>`url`: URL om bestand te downloaden</li><li>`path`: Bestand onder dit pad opslaan in ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Dubbele verwerking voor ZIP-archieven (`fmt=zip`). Standaard wordt een fout gegenereerd bij meerdere bestanden die in het ZIP-bestand onder hetzelfde pad zijn opgeslagen. Als u `duplicate` instelt op `ignore`, wordt alleen het eerste element opgeslagen en de rest genegeerd. | `ignore` |
| `watermark` | `object` | Bevat instructies over [watermerk](#watermark-specific-fields). |  |

### Watermerkspecifieke velden {#watermark-specific-fields}

De PNG-indeling wordt gebruikt als een watermerk.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Schaal van het watermerk, tussen `0.0` en `1.0`. `1.0` betekent dat het watermerk de oorspronkelijke schaal (1:1) heeft en de laagste waarden de grootte van het watermerk verminderen. | De waarde `0.5` betekent de helft van de oorspronkelijke grootte. |
| `image` | `url` | URL naar het PNG-bestand dat moet worden gebruikt voor het watermerk. |  |

## Asynchrone gebeurtenissen {#asynchronous-events}

Zodra de verwerking van een vertoning is voltooid of wanneer een fout optreedt, wordt een gebeurtenis verzonden naar een [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). De cliënten moeten aan dagboek URL luisteren die door [/register](#register) wordt verstrekt. De dagboekreactie omvat een `event` serie die uit één voorwerp voor elke gebeurtenis bestaat, waarvan `event` gebied de daadwerkelijke gebeurtenislading omvat.

Het gebeurtenistype [!DNL Adobe I/O] voor alle gebeurtenissen van [!DNL Asset Compute Service] is `asset_compute`. Het dagboek wordt automatisch op dit gebeurtenistype slechts geabonneerd en er is geen verdere vereiste om te filtreren die op het [!DNL Adobe I/O] gebeurtenistype wordt gebaseerd. De service-specifieke gebeurtenistypen zijn beschikbaar in de eigenschap `type` van de gebeurtenis.

### Gebeurtenistypen {#event-types}

| Gebeurtenis | Beschrijving |
|---------------------|-------------|
| `rendition_created` | Verzonden voor elke correct verwerkte en geüploade vertoning. |
| `rendition_failed` | Verzonden voor elke uitvoering die niet kon worden verwerkt of geüpload. |

### Gebeurteniskenmerken {#event-attributes}

| Kenmerk | Type | Gebeurtenis | Beschrijving |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tijdstempel wanneer de gebeurtenis in de vereenvoudigde uitgebreide [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)-indeling is verzonden, zoals gedefinieerd door JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | De aanvraag-id van het oorspronkelijke verzoek aan `/process`, gelijk aan `X-Request-Id` header. |
| `source` | `object` | `*` | De `source` van het `/process` verzoek. |
| `userData` | `object` | `*` | De `userData` van de vertoning van het `/process` verzoek als reeks. |
| `rendition` | `object` | `rendition_*` | Het overeenkomende weergaveobject dat wordt doorgegeven in `/process`. |
| `metadata` | `object` | `rendition_created` | De [metadata](#metadata)-eigenschappen van de vertoning. |
| `errorReason` | `string` | `rendition_failed` | Renderingsfout [reason](#error-reasons) indien aanwezig. |
| `errorMessage` | `string` | `rendition_failed` | Tekst die meer details geeft over de eventuele uitvoerfout. |

### Metagegevens {#metadata}

| Eigenschap | Beschrijving |
|--------|-------------|
| `repo:size` | De grootte van de vertoning in bytes. |
| `repo:sha1` | De sha1-samenvatting van de uitvoering. |
| `dc:format` | Het MIME-type van de vertoning. |
| `repo:encoding` | De charsetcodering van de vertoning voor het geval dat deze een op tekst gebaseerde indeling is. |
| `tiff:ImageWidth` | De breedte van de vertoning in pixels. Alleen aanwezig voor afbeeldingsuitvoeringen. |
| `tiff:ImageLength` | De lengte van de vertoning in pixels. Alleen aanwezig voor afbeeldingsuitvoeringen. |

### Foutredenen {#error-reasons}

| Reden | Beschrijving |
|---------|-------------|
| `RenditionFormatUnsupported` | De aangevraagde renditie-indeling wordt niet ondersteund voor de opgegeven bron. |
| `SourceUnsupported` | De specifieke bron wordt niet ondersteund, ook al wordt het type ondersteund. |
| `SourceCorrupt` | De brongegevens zijn beschadigd. Bevat lege bestanden. |
| `RenditionTooLarge` | De vertoning kan niet worden geüpload met de vooraf ondertekende URL(&#39;s) in `target`. De werkelijke grootte van de vertoning is beschikbaar als metagegevens in `repo:size` en kan door de client worden gebruikt om deze vertoning opnieuw te verwerken met het juiste aantal vooraf ondertekende URL&#39;s. |
| `GenericError` | Een andere onverwachte fout. |
