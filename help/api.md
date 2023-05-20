---
title: "[!DNL Asset Compute Service] HTTP-API"
description: "[!DNL Asset Compute Service] HTTP-API om aangepaste toepassingen te maken."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 0%

---

# [!DNL Asset Compute Service] HTTP-API {#asset-compute-http-api}

Het gebruik van de API is beperkt tot ontwikkelingsdoeleinden. De API wordt als context verstrekt wanneer het ontwikkelen van douanetoepassingen. [!DNL Adobe Experience Manager] als [!DNL Cloud Service] gebruikt de API om de verwerkingsgegevens door te geven aan een aangepaste toepassing. Zie voor meer informatie [Middelenmicroservices en verwerkingsprofielen gebruiken](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als [!DNL Cloud Service].

Elke client van de [!DNL Asset Compute Service] HTTP API moet deze stroom op hoog niveau volgen:

1. Een client is ingericht als [!DNL Adobe Developer Console] in een IMS-organisatie. Elke afzonderlijke client (systeem of omgeving) vereist een eigen afzonderlijk project om de gegevensstroom van de gebeurtenis te scheiden.

1. Een cliënt produceert een toegangstoken voor de technische rekening gebruikend [JWT-verificatie (serviceaccount)](https://www.adobe.io/authentication/auth-methods.html).

1. Een clientaanroep [`/register`](#register) slechts eenmaal om de journaal-URL op te halen.

1. Een clientaanroep [`/process`](#process-request) voor elk element waarvoor het uitvoeringen wil genereren. De vraag is asynchroon.

1. Een klant pollt regelmatig het dagboek naar [ontvangen, gebeurtenissen](#asynchronous-events). Er worden gebeurtenissen voor elke aangevraagde uitvoering ontvangen wanneer de uitvoering is verwerkt (`rendition_created` gebeurtenistype) of als er een fout is (`rendition_failed` gebeurtenistype).

De [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) maakt het gemakkelijk om API in code Node.js te gebruiken.

## Verificatie en autorisatie {#authentication-and-authorization}

Voor alle API&#39;s is verificatie van toegangstoken vereist. De verzoeken moeten de volgende kopballen plaatsen:

1. `Authorization` header met token aan toonder, de token van een technische account, ontvangen via [JWT-uitwisseling](https://www.adobe.io/authentication/auth-methods.html) uit Adobe Developer Console-project. De [bereik](#scopes) worden hieronder beschreven.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` met de IMS-organisatie-id.

1. `x-api-key` met de client-id van de [!DNL Adobe Developers Console] project.

### Segmenten {#scopes}

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

Hiervoor zijn [!DNL Adobe Developer Console] project waarop moet worden geabonneerd `Asset Compute`, `I/O Events`, en `I/O Management API` diensten. De uitsplitsing van individuele scènes is:

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

Elke client van de [!DNL Asset Compute service] - een unieke [!DNL Adobe Developer Console] project geabonneerd op de dienst - most [registreren](#register-request) voordat u verzoeken om verwerking indient. De registratiestap retourneert het unieke gebeurtenisjournaal dat vereist is om de asynchrone gebeurtenissen van de uitvoering op te halen.

Aan het einde van de levenscyclus kan een client [unregister](#unregister-request).

### Aanvraag registreren {#register-request}

Deze API-aanroep stelt een [!DNL Asset Compute] en geeft de URL van het gebeurtenisjournaal weer. Dit is een epidemische bewerking die slechts één keer voor elke client hoeft te worden uitgevoerd. Het kan opnieuw worden geroepen om het dagboek URL terug te winnen.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/register` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet leeg zijn. |

### Reactie registreren {#register-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `journal`, `ok` en/of `requestId` velden. |

De HTTP-statuscodes zijn:

* **200 succesvol**: Wanneer het verzoek succesvol is. Het bevat de `journal` URL die op de hoogte wordt gesteld van de resultaten van de asynchrone verwerking die via `/process` (als gebeurtenistype) `rendition_created` wanneer succesvol, of `rendition_failed` bij falen).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek ongeldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: komt voor wanneer het verzoek ongeldig is [autorisatie](#authentication-and-authorization). Een voorbeeld kan een geldig toegangstoken zijn, maar het project van de Console van Adobe Developer (technische rekening) wordt niet geabonneerd aan alle vereiste diensten.

* **429 Te veel verzoeken**: treedt op wanneer het systeem door deze cliënt of anders wordt overbelast. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.
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

### Aanvraag ongedaan maken {#unregister-request}

Met deze API-aanroep wordt de registratie van [!DNL Asset Compute] client. Hierna is het niet meer mogelijk `/process`. Wanneer de API-aanroep voor een niet-geregistreerde client wordt gebruikt of een nog te registreren client een `404` fout.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/unregister` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Leeg. |

### Reactie ongedaan maken {#unregister-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` en `requestId` velden. |

De statuscodes zijn:

* **200 succesvol**: treedt op wanneer de registratie en het dagboek worden gevonden en verwijderd.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek ongeldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: komt voor wanneer het verzoek ongeldig is [autorisatie](#authentication-and-authorization). Een voorbeeld kan een geldig toegangstoken zijn, maar het project van de Console van Adobe Developer (technische rekening) wordt niet geabonneerd aan alle vereiste diensten.

* **404 Niet gevonden**: komt voor wanneer er geen huidige registratie voor de bepaalde geloofsbrieven is.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Te veel verzoeken**: treedt op wanneer het systeem wordt overbelast. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.

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

De `process` bewerking verzendt een taak die een bronelement in meerdere uitvoeringen omzet, op basis van de instructies in de aanvraag. Meldingen over geslaagde voltooiing (gebeurtenistype) `rendition_created`) of fouten (gebeurtenistype) `rendition_failed`) worden verzonden naar een gebeurtenisjournaal dat moet worden opgehaald met [/register](#register) om het even welk aantal `/process` verzoeken. Onjuist gevormde verzoeken ontbreken onmiddellijk met een 400 foutencode.

Er wordt verwezen naar binaire URL&#39;s, zoals vooraf ondertekende URL&#39;s van Amazon AWS S3 of Azure Blob Storage SAS URL&#39;s, voor beide leesbewerkingen van de `source` activa (`GET` URL&#39;s) en de vertoningen schrijven (`PUT` URL&#39;s). De klant is verantwoordelijk voor het genereren van deze vooraf ondertekende URL&#39;s.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/process` |
| MIME-type | `application/json` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet de JSON-indeling van het procesverzoek hebben, zoals hieronder wordt beschreven. Er worden instructies gegeven over welk element moet worden verwerkt en welke uitvoeringen moeten worden gegenereerd. |

### Verzoek om verwerking JSON {#process-request-json}

De verzoekende instantie `/process` is een JSON-object met dit schema op hoog niveau:

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
| `source` | `object` | Beschrijf het bronelement dat moet worden verwerkt. Zie beschrijving van [Bronobjectvelden](#source-object-fields) hieronder. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Uitvoeringen die moeten worden gegenereerd op basis van het bronbestand. Elk weergaveobject ondersteunt [renderinstructie](#rendition-instructions). Vereist. | `[{ "target": "https://....", "fmt": "png" }]` |

De `source` kan `<string>` die als een URL wordt beschouwd of een `<object>` met een extra veld. De volgende varianten zijn vergelijkbaar:

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
| `name` | `string` | Bestandsnaam van bronelement. De bestandsextensie in de naam kan worden gebruikt als er geen MIME-type kan worden gedetecteerd. Heeft voorrang op bestandsnaam in URL-pad of bestandsnaam in `content-disposition` header van de binaire bron. De standaardwaarde is &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Bestandsgrootte van bronelement in bytes. Heeft voorrang op `content-length` header van de binaire bron. | `10234` |
| `mimetype` | `string` | MIME-type voor bronelementbestand. Heeft voorrang op de `content-type` header van de binaire bron. | `"image/jpeg"` |

### Een complete `process` aanvraagvoorbeeld {#complete-process-request-example}

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

De `/process` het verzoek keert onmiddellijk met een succes of een mislukking terug die op de basisverzoekbevestiging wordt gebaseerd. De werkelijke verwerking van elementen gebeurt asynchroon.

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` en `requestId` velden. |

Statuscodes:

* **200 succesvol**: Als het verzoek is verzonden. Response JSON omvat `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Ongeldig verzoek**: Als de aanvraag onjuist is samengesteld, ontbreken bijvoorbeeld vereiste velden in de aanvraag-JSON. Response JSON omvat `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Niet-geautoriseerd**: Wanneer de aanvraag niet geldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.
* **403 Verboden**: Wanneer de aanvraag niet geldig is [autorisatie](#authentication-and-authorization). Een voorbeeld kan een geldig toegangstoken zijn, maar het project van de Console van Adobe Developer (technische rekening) wordt niet geabonneerd aan alle vereiste diensten.
* **429 Te veel verzoeken**: Wanneer het systeem door deze client of in het algemeen wordt overbelast. De clients kunnen het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.
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

De meeste klanten zijn waarschijnlijk geneigd om exact hetzelfde verzoek opnieuw uit te voeren met [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff) bij elke fout *behalve* configuratieproblemen zoals 401 of 403 of ongeldige verzoeken zoals 400. Afgezien van een reguliere tariefbeperking via 429 reacties, kan een tijdelijke onderbreking of beperking van de service leiden tot 5x fouten. Het zou dan raadzaam zijn om na een bepaalde periode opnieuw te proberen.

Alle JSON-reacties (indien aanwezig) bevatten de `requestId` dat dezelfde waarde heeft als de `X-Request-Id` header. Het wordt aanbevolen om van de koptekst te lezen, aangezien deze altijd aanwezig is. De `requestId` wordt ook geretourneerd in alle gebeurtenissen met betrekking tot verwerkingsverzoeken als `requestId`. Clients mogen geen aanname maken over de opmaak van deze tekenreeks, het is een ondoorzichtige tekenreeks-id.

## Inschakelen naar naverwerking {#opt-in-to-post-processing}

De [asset compute SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt een aantal basisopties voor nabewerking van afbeeldingen. Aangepaste workers kunnen zich expliciet aanmelden bij naverwerking door het veld in te stellen `postProcess` op het weergaveobject naar `true`.

De ondersteunde gebruiksgevallen zijn:

* Een vertoning uitsnijden naar een rechthoek waarvan de grenzen worden gedefinieerd door crop.w, crop.h, crop.x en crop.y. Het wordt gedefinieerd door `instructions.crop` in het weergaveobject.
* Wijzig de grootte van afbeeldingen met breedte, hoogte of beide. Het wordt gedefinieerd door `instructions.width` en `instructions.height` in het weergaveobject. Als u alleen de breedte of hoogte wilt wijzigen, stelt u slechts één waarde in. Met Compute Service behoudt u de hoogte-breedteverhouding.
* Stel de kwaliteit in voor een JPEG-afbeelding. Het wordt gedefinieerd door `instructions.quality` in het weergaveobject. De beste kwaliteit wordt aangegeven door `100` en lagere waarden duiden op een lagere kwaliteit.
* Maak geïnterlinieerde afbeeldingen. Het wordt gedefinieerd door `instructions.interlace` in het weergaveobject.
* Stel DPI in om de gerenderde grootte aan te passen voor publicatie op het bureaublad door de schaal aan te passen die op de pixels wordt toegepast. Het wordt gedefinieerd door `instructions.dpi` in het weergaveobject om de dpi-resolutie te wijzigen. Als u de afbeelding echter zo wilt vergroten of verkleinen dat deze dezelfde grootte heeft bij een andere resolutie, gebruikt u de opdracht `convertToDpi` instructies.
* Wijzig de grootte van de afbeelding zodanig dat de weergegeven breedte of hoogte gelijk blijft aan het origineel bij de opgegeven doelresolutie (DPI). Het wordt gedefinieerd door `instructions.convertToDpi` in het weergaveobject.

## Watermerkelementen {#add-watermark}

De [asset compute SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt het toevoegen van een watermerk aan PNG-, JPEG-, TIFF- en GIF-afbeeldingsbestanden. Het watermerk wordt toegevoegd na de vertoningsinstructies in het dialoogvenster `watermark` object op de vertoning.

Watermerken worden uitgevoerd tijdens de nabewerking van de vertoning. De aangepaste worker voor watermerkelementen [na verwerking](#opt-in-to-post-processing) door het veld in te stellen `postProcess` op het weergaveobject naar `true`. Als de worker niet de optie Weigeren inschakelt, wordt het watermerk niet toegepast, zelfs niet als het watermerkobject is ingesteld op het weergaveobject in de aanvraag.

## Vertoningsinstructies {#rendition-instructions}

Dit zijn de beschikbare opties voor de `renditions` array in [/process](#process-request).

### Algemene velden {#common-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | De doelindeling van uitvoeringen kan ook `text` voor tekstextractie en `xmp` voor het extraheren van XMP metagegevens als xml. Zie [ondersteunde indelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL van een [aangepaste toepassing](develop-custom-application.md). Moet een `https://` URL. Als dit veld aanwezig is, wordt de vertoning gemaakt door een aangepaste toepassing. Een ander setveld voor uitvoering wordt vervolgens gebruikt in de aangepaste toepassing. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL waarnaar de gegenereerde uitvoering moet worden geüpload met HTTP-PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Vooraf ondertekende URL met meerdere delen uploadgegevens voor de gegenereerde uitvoering. Dit is bedoeld voor [Binair uploaden van AEM/Eak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) met [uploadgedrag voor meerdere delen](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Velden:<ul><li>`urls`: array van tekenreeksen, één voor elke vooraf ondertekende deel-URL</li><li>`minPartSize`: minimumgrootte voor één onderdeel = url</li><li>`maxPartSize`: de maximumgrootte voor één onderdeel = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optionele gereserveerde ruimte die door de client wordt beheerd en net als weergavegebeurtenissen wordt doorgegeven. Staat cliënten toe om douaneinformatie toe te voegen om vertoningsgebeurtenissen te identificeren. Moet niet worden gewijzigd of vertrouwd op in douanetoepassingen, aangezien de cliënten vrij zijn om dit op elk ogenblik te veranderen. | `{ ... }` |

### Vertoningsspecifieke velden {#rendition-specific-fields}

Voor een lijst met momenteel ondersteunde bestandsindelingen raadpleegt u [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `*` | `*` | Geavanceerde aangepaste velden kunnen worden toegevoegd aan een [aangepaste toepassing](develop-custom-application.md) begrijpt. |  |
| `embedBinaryLimit` | `number` in bytes | Als deze waarde is ingesteld en het bestand van de vertoning kleiner is dan deze waarde, wordt de vertoning ingesloten in de gebeurtenis die wordt verzonden zodra de renditie is voltooid. De maximaal toegestane grootte voor insluiten is 32 kB (32 x 1024 bytes). Als een vertoning groter is dan de `embedBinaryLimit` beperkt, wordt deze op een locatie in de cloudopslag geplaatst en wordt niet ingesloten in de gebeurtenis. | `3276` |
| `width` | `number` | Breedte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
| `height` | `number` | Hoogte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
|  |  | De verhouding blijft altijd behouden als: <ul> <li> Beide `width` en `height` worden opgegeven, past de afbeelding in de grootte aan terwijl de hoogte-breedteverhouding behouden blijft </li><li> Alleen `width` of alleen `height` is opgegeven, gebruikt de resulterende afbeelding de corresponderende dimensie terwijl de hoogte-breedteverhouding behouden blijft</li><li> Als beide `width` noch `height` wordt opgegeven, wordt de oorspronkelijke pixelgrootte van de afbeelding gebruikt. Het hangt van het brontype af. Voor sommige indelingen, zoals PDF-bestanden, wordt een standaardgrootte gebruikt. Er kan een maximumgrootte zijn.</li></ul> |  |
| `quality` | `number` | JPEG-kwaliteit opgeven in het bereik van `1` tot `100`. Alleen van toepassing op afbeeldingsuitvoeringen. | `90` |
| `xmp` | `string` | Wordt alleen gebruikt door XMP terugverwijzing van metagegevens. Het is XMP base64 gecodeerd om terug te schrijven naar de opgegeven uitvoering. |  |
| `interlace` | `bool` | Geïnterlinieerde PNG- of GIF- of progressieve JPEG maken door deze in te stellen op `true`. Dit heeft geen invloed op andere bestandsindelingen. |  |
| `jpegSize` | `number` | De grootte van het JPEG-bestand in bytes wordt benaderd. Alle `quality` instellen. Heeft geen effect op andere indelingen. |  |
| `dpi` | `number` of `object` | Stel de DPI voor x en y in. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. Dit heeft geen invloed op de afbeelding zelf. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` of `object` | x- en y-DPI-waarden opnieuw samplen terwijl de fysieke grootte behouden blijft. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lijst met bestanden die moeten worden opgenomen in het ZIP-archief (`fmt=zip`). Elk item kan een URL-tekenreeks zijn of een object met de velden:<ul><li>`url`: URL om bestand te downloaden</li><li>`path`: Bestand onder dit pad opslaan in ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Dubbele verwerking voor ZIP-archieven (`fmt=zip`). Standaard wordt een fout gegenereerd bij meerdere bestanden die in het ZIP-bestand onder hetzelfde pad zijn opgeslagen. Instelling `duplicate` tot `ignore` resulteert alleen in het eerste element dat moet worden opgeslagen en de rest die moet worden genegeerd. | `ignore` |
| `watermark` | `object` | Bevat instructies over [watermerk](#watermark-specific-fields). |  |

### Watermerkspecifieke velden {#watermark-specific-fields}

De PNG-indeling wordt gebruikt als een watermerk.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Schaal van het watermerk tussen `0.0` en `1.0`. `1.0` betekent dat het watermerk de oorspronkelijke schaal (1:1) heeft en de laagste waarden de grootte van het watermerk verminderen. | Een waarde van `0.5` de helft van de oorspronkelijke grootte. |
| `image` | `url` | URL naar het PNG-bestand dat moet worden gebruikt voor het watermerk. |  |

## Asynchrone gebeurtenissen {#asynchronous-events}

Nadat een vertoning is verwerkt of een fout optreedt, wordt een gebeurtenis verzonden naar een [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Clients moeten luisteren naar de journaal-URL die via [/register](#register). De dagboekreactie omvat een `event` array die bestaat uit één object voor elke gebeurtenis, waarvan de `event` bevat het daadwerkelijke gebeurtenislaadveld.

De [!DNL Adobe I/O] Gebeurtenistype voor alle gebeurtenissen van het dialoogvenster [!DNL Asset Compute Service] is `asset_compute`. Het dagboek wordt automatisch ingetekend op dit gebeurtenistype slechts en er is geen verdere vereiste om te filtreren die op [!DNL Adobe I/O] Type gebeurtenis. De de dienstspecifieke gebeurtenistypes zijn beschikbaar in `type` eigenschap van de gebeurtenis.

### Gebeurtenistypen {#event-types}

| Gebeurtenis | Beschrijving |
|---------------------|-------------|
| `rendition_created` | Verzonden voor elke correct verwerkte en geüploade vertoning. |
| `rendition_failed` | Verzonden voor elke uitvoering die niet kon worden verwerkt of geüpload. |

### Gebeurteniskenmerken {#event-attributes}

| Kenmerk | Type | Gebeurtenis | Beschrijving |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tijdstempel wanneer de gebeurtenis in het vereenvoudigde verlengde is verzonden [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) indeling, zoals gedefinieerd door JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | De aanvraag-id van het oorspronkelijke verzoek aan `/process`, gelijk aan `X-Request-Id` header. |
| `source` | `object` | `*` | De `source` van de `/process` verzoek. |
| `userData` | `object` | `*` | De `userData` van de vertoning van de `/process` request if set. |
| `rendition` | `object` | `rendition_*` | Het overeenkomende weergaveobject dat wordt doorgegeven `/process`. |
| `metadata` | `object` | `rendition_created` | De [metagegevens](#metadata) eigenschappen van de vertoning. |
| `errorReason` | `string` | `rendition_failed` | Vertoning mislukt [reden](#error-reasons) indien van toepassing. |
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
| `RenditionTooLarge` | De vertoning kan niet worden geüpload met de vooraf ondertekende URL(&#39;s) in `target`. De werkelijke weergavegrootte is beschikbaar als metagegevens in `repo:size` en kan door de client worden gebruikt om deze vertoning opnieuw te verwerken met het juiste aantal vooraf ondertekende URL&#39;s. |
| `GenericError` | Een andere onverwachte fout. |
