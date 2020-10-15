---
title: '[!DNL Asset Compute Service] HTTP-API.'
description: '[!DNL Asset Compute Service] HTTP-API om aangepaste toepassingen te maken.'
translation-type: tm+mt
source-git-commit: 18e97e544014933e9910a12bc40246daa445bf4f
workflow-type: tm+mt
source-wordcount: '2931'
ht-degree: 1%

---


# [!DNL Asset Compute Service] HTTP-API {#asset-compute-http-api}

Het gebruik van de API is beperkt tot ontwikkelingsdoeleinden. De API wordt als context verstrekt wanneer het ontwikkelen van douanetoepassingen. [!DNL Adobe Experience Manager] als Cloud Service de API gebruikt om de verwerkingsgegevens door te geven aan een aangepaste toepassing. Zie [Elementmicroservices en Verwerkingsprofielen](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)gebruiken voor meer informatie.

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als Cloud Service.

Elke client van de [!DNL Asset Compute Service] HTTP API moet deze flow op hoog niveau volgen:

1. Een cliënt wordt provisioned als [!DNL Adobe Developer Console] project in een organisatie IMS. Elke afzonderlijke client (systeem of omgeving) vereist een eigen afzonderlijk project om de gegevensstroom van de gebeurtenis te scheiden.

1. Een cliënt produceert een toegangstoken voor de technische rekening gebruikend de Authentificatie [van](https://www.adobe.io/authentication/auth-methods.html)JWT (van de Rekening van de Dienst).

1. Een cliënt roept [`/register`](#register) slechts eenmaal om dagboek URL terug te winnen.

1. Een client roept [`/process`](#process-request) voor elk element aan waarvoor het rendities wil genereren. De vraag is asynchroon.

1. Een cliënt opinieert regelmatig het dagboek om gebeurtenissen [te](#asynchronous-events)ontvangen. Er worden gebeurtenissen voor elke aangevraagde uitvoering ontvangen wanneer de uitvoering is verwerkt (`rendition_created` gebeurtenistype) of wanneer er een fout (`rendition_failed` gebeurtenistype) is.

Met de [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) -module kunt u de API in Node.js-code eenvoudig gebruiken.

## Verificatie en autorisatie {#authentication-and-authorization}

Voor alle API&#39;s is verificatie van toegangstoken vereist. De verzoeken moeten de volgende kopballen plaatsen:

1. `Authorization` header met token voor toonder, dit is het token voor een technische account, ontvangen via [JWT-uitwisseling](https://www.adobe.io/authentication/auth-methods.html) van het project Adobe Developer Console. Het [bereik](#scopes) wordt hieronder beschreven.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` met de IMS-organisatie-id.

1. `x-api-key` met de client-id van het [!DNL Adobe Developers Console] project.

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

Deze vereisen dat het [!DNL Adobe Developer Console] project wordt ingetekend aan `Asset Compute`, `I/O Events`en de `I/O Management API` diensten. De uitsplitsing van individuele scènes is:

* Basis
   * bereik: `openid,AdobeID`

* Verwerking van middelen
   * metareaal: `asset_compute_meta`
   * bereik: `asset_compute,read_organizations`

* Adobe I/O-gebeurtenissen
   * metareaal: `event_receiver_api`
   * bereik: `event_receiver,event_receiver_api`

* API voor Adobe I/O-beheer
   * metareaal: `ent_adobeio_sdk`
   * bereik: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registratie {#register}

Elke cliënt van het [!DNL Asset Compute service] - een uniek [!DNL Adobe Developer Console] project dat aan de dienst wordt ingetekend - moet [registreren](#register-request) alvorens verwerkingsverzoeken te maken. De registratiestap retourneert het unieke gebeurtenisjournaal dat vereist is om de asynchrone gebeurtenissen van de uitvoering op te halen.

Aan het eind van zijn levenscyclus, kan een cliënt [unregister](#unregister-request).

### Aanvraag registreren {#register-request}

Met deze API-aanroep wordt een [!DNL Asset Compute] client ingesteld en wordt de URL van het gebeurtenisdagboek weergegeven. Dit is een epidemische bewerking die slechts één keer voor elke client hoeft te worden uitgevoerd. Het kan opnieuw worden geroepen om het dagboek URL terug te winnen.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/register` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde kopteksten](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet leeg zijn. |

### Reactie registreren {#register-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Hetzelfde als de `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `journal`, `ok` en/of `requestId` -velden. |

De HTTP-statuscodes zijn:

* **200 succesvol**: Wanneer het verzoek succesvol is. Het bevat de `journal` URL die op de hoogte wordt gebracht van de resultaten van de asynchrone verwerking die via `/process` (als gebeurtenistype `rendition_created` wanneer succesvol, of `rendition_failed` wanneer mislukt) worden geactiveerd.

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek geen geldige [authentificatie](#authentication-and-authorization)heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: zich voordoet wanneer de aanvraag geen geldige [vergunning](#authentication-and-authorization)heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

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

### Aanvraag ongedaan maken {#unregister-request}

Met deze API-aanroep wordt de registratie van een [!DNL Asset Compute] client ongedaan gemaakt. Hierna is het niet meer mogelijk om te bellen `/process`. Als u de API-aanroep voor een niet-geregistreerde client of een nog te registreren client gebruikt, wordt een `404` fout geretourneerd.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/unregister` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde kopteksten](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Leeg. |

### Reactie ongedaan maken {#unregister-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Hetzelfde als de `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` en `requestId` velden. |

De statuscodes zijn:

* **200 succesvol**: treedt op wanneer de registratie en het dagboek worden gevonden en verwijderd.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Niet-geautoriseerd**: komt voor wanneer het verzoek geen geldige [authentificatie](#authentication-and-authorization)heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: zich voordoet wanneer de aanvraag geen geldige [vergunning](#authentication-and-authorization)heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

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

De `process` bewerking verzendt een taak die een bronelement in meerdere uitvoeringen omzet, op basis van de instructies in de aanvraag. Meldingen over geslaagde voltooiing (gebeurtenistype `rendition_created`) of over eventuele fouten (gebeurtenistype `rendition_failed`) worden verzonden naar een gebeurtenisdagboek dat moet worden opgehaald met [/register](#register) voordat een aantal `/process` aanvragen kan worden gedaan. Onjuist gevormde verzoeken ontbreken onmiddellijk met een 400 foutencode.

Er wordt verwezen naar binaire URL&#39;s, zoals vooraf ondertekende URL&#39;s van Amazon AWS S3 of Azure Blob Storage SAS URL&#39;s, voor zowel het lezen van het `source` element (`GET` URL&#39;s) als het schrijven van de uitvoeringen (`PUT` URL&#39;s). De klant is verantwoordelijk voor het genereren van deze vooraf ondertekende URL&#39;s.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/process` |
| MIME-type | `application/json` |
| Koptekst `Authorization` | Alle [machtigingsgerelateerde kopteksten](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel kunnen clients een unieke end-to-end-id instellen voor de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Moet de JSON-indeling van het procesverzoek hebben, zoals hieronder wordt beschreven. Er worden instructies gegeven over welk element moet worden verwerkt en welke uitvoeringen moeten worden gegenereerd. |

### Verzoek om verwerking JSON {#process-request-json}

De aanvraaginstantie van `/process` is een JSON-object met dit schema op hoog niveau:

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
| `source` | `object` | Beschrijf het bronelement dat moet worden verwerkt. Zie de beschrijving van de [bronobjectvelden](#source-object-fields) hieronder. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Uitvoeringen die moeten worden gegenereerd op basis van het bronbestand. Elk weergaveobject ondersteunt [vertoningsinstructie](#rendition-instructions). Vereist. | `[{ "target": "https://....", "fmt": "png" }]` |

Het `source` kan een URL zijn `<string>` die als een URL wordt gezien of het kan een veld `<object>` met een extra veld zijn. De volgende varianten zijn vergelijkbaar:

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
| `name` | `string` | Bestandsnaam van bronelement. De bestandsextensie in de naam kan worden gebruikt als er geen MIME-type kan worden gedetecteerd. Heeft voorrang op bestandsnaam in URL-pad of bestandsnaam in `content-disposition` header van binaire bron. De standaardwaarde is &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Bestandsgrootte van bronelement in bytes. Heeft voorrang op `content-length` kopbal van het binaire middel. | `10234` |
| `mimetype` | `string` | MIME-type voor bronelementbestand. Heeft voorrang op de `content-type` header van de binaire resource. | `"image/jpeg"` |

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
| Koptekst `X-Request-Id` | Hetzelfde als de `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen en/of steunverzoeken. |
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

* **401 Niet-geautoriseerd**: Wanneer de aanvraag geen geldige [verificatie](#authentication-and-authorization)heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.
* **403 Verboden**: Wanneer het verzoek geen geldige [vergunning](#authentication-and-authorization)heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het project van de Console van de Ontwikkelaar van Adobe (technische rekening) wordt niet ingetekend aan alle vereiste diensten.
* **Te veel verzoeken**: Wanneer het systeem door deze client of in het algemeen wordt overbelast. De klanten kunnen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff)opnieuw proberen. Het lichaam is leeg.
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

Alle JSON-reacties (indien aanwezig) bevatten de waarde `requestId` die gelijk is aan de `X-Request-Id` koptekst. Het wordt aanbevolen om van de koptekst te lezen, aangezien deze altijd aanwezig is. Het `requestId` wordt ook geretourneerd in alle gebeurtenissen die betrekking hebben op de verwerking van aanvragen als `requestId`. Clients mogen geen aanname maken over de opmaak van deze tekenreeks, het is een ondoorzichtige tekenreeks-id.

## Inschakelen naar naverwerking {#opt-in-to-post-processing}

De SDK [voor](https://github.com/adobe/asset-compute-sdk) Asset Compute ondersteunt een aantal basisopties voor nabewerking van afbeeldingen. Aangepaste workers kunnen zich expliciet aanmelden bij nabewerking door het veld `postProcess` op het weergaveobject in te stellen op `true`.

De ondersteunde gebruiksgevallen zijn:

* Een vertoning uitsnijden naar een rechthoek waarvan de grenzen worden gedefinieerd door crop.w, crop.h, crop.x en crop.y. Deze wordt gedefinieerd door `instructions.crop` in het weergaveobject.
* Wijzig de grootte van afbeeldingen met breedte, hoogte of beide. Deze wordt door `instructions.width` `instructions.height` en in het weergaveobject gedefinieerd. Als u alleen de breedte of hoogte wilt wijzigen, stelt u slechts één waarde in. Met Compute Service behoudt u de hoogte-breedteverhouding.
* Stel de kwaliteit in voor een JPEG-afbeelding. Deze wordt gedefinieerd door `instructions.quality` in het weergaveobject. De beste kwaliteit wordt aangegeven met `100` en kleinere waarden geven een lagere kwaliteit aan.
* Maak geïnterlinieerde afbeeldingen. Deze wordt gedefinieerd door `instructions.interlace` in het weergaveobject.
* Stel DPI in om de gerenderde grootte aan te passen voor publicatie op het bureaublad door de schaal aan te passen die op de pixels wordt toegepast. De eigenschap wordt gedefinieerd door `instructions.dpi` in het weergaveobject om de dpi-resolutie te wijzigen. Als u de afbeelding echter wilt vergroten of verkleinen zodat deze dezelfde grootte heeft bij een andere resolutie, gebruikt u de `convertToDpi` instructies.
* Wijzig de grootte van de afbeelding zodanig dat de weergegeven breedte of hoogte gelijk blijft aan het origineel bij de opgegeven doelresolutie (DPI). Deze wordt gedefinieerd door `instructions.convertToDpi` in het weergaveobject.

## Watermerkelementen {#add-watermark}

De SDK [voor](https://github.com/adobe/asset-compute-sdk) Asset Compute ondersteunt het toevoegen van een watermerk aan PNG-, JPEG-, TIFF- en GIF-afbeeldingsbestanden. Het watermerk wordt toegevoegd na de vertoningsinstructies in het `watermark` object op de vertoning.

Watermerken worden uitgevoerd tijdens de nabewerking van de vertoning. Als u elementen van een watermerk wilt voorzien, [gaat de aangepaste worker naar een nabewerking](#opt-in-to-post-processing) door het veld `postProcess` op het weergaveobject in te stellen op `true`. Als de worker niet de optie Weigeren inschakelt, wordt het watermerk niet toegepast, zelfs niet als het watermerkobject is ingesteld op het weergaveobject in de aanvraag.

## Vertoningsinstructies {#rendition-instructions}

Dit zijn de beschikbare opties voor de `renditions` array in [/proces](#process-request).

### Algemene velden {#common-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | De doelindeling van uitvoeringen kan ook worden gebruikt `text` voor het extraheren van tekst en `xmp` voor het extraheren van XMP metagegevens als XML. Zie [ondersteunde indelingen](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL van een [aangepaste toepassing](develop-custom-application.md). Dit moet een `https://` URL zijn. Als dit veld aanwezig is, wordt de vertoning gemaakt door een aangepaste toepassing. Een ander setveld voor uitvoering wordt vervolgens gebruikt in de aangepaste toepassing. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL waarnaar de gegenereerde uitvoering moet worden geüpload met HTTP-PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Vooraf ondertekende URL met meerdere delen uploadgegevens voor de gegenereerde uitvoering. Dit is voor [AEM/Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) met dit [multipart uploadgedrag](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fields:<ul><li>`urls`: array van tekenreeksen, één voor elke vooraf ondertekende deel-URL</li><li>`minPartSize`: minimumgrootte voor één onderdeel = url</li><li>`maxPartSize`: de maximumgrootte voor één onderdeel = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optionele gereserveerde ruimte die door de client wordt beheerd en net als weergavegebeurtenissen wordt doorgegeven. Staat cliënten toe om douaneinformatie toe te voegen om vertoningsgebeurtenissen te identificeren. Moet niet worden gewijzigd of vertrouwd op in douanetoepassingen, aangezien de cliënten vrij zijn om dit op elk ogenblik te veranderen. | `{ ... }` |

### Vertoningsspecifieke velden {#rendition-specific-fields}

Zie [Ondersteunde bestandsindelingen](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html)voor een lijst met momenteel ondersteunde bestandsindelingen.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `*` | `*` | Geavanceerde, aangepaste gebieden kunnen worden toegevoegd die een [douanetoepassing](develop-custom-application.md) begrijpt. |  |
| `embedBinaryLimit` | `number` in bytes | Als deze waarde is ingesteld en het bestand van de vertoning kleiner is dan deze waarde, wordt de vertoning ingesloten in de gebeurtenis die wordt verzonden zodra de renditie is voltooid. De maximaal toegestane grootte voor insluiten is 32 kB (32 x 1024 bytes). Als een vertoning groter is dan de `embedBinaryLimit` limiet, wordt deze op een locatie in de cloudopslag geplaatst en wordt deze niet ingesloten in de gebeurtenis. | `3276` |
| `width` | `number` | Breedte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
| `height` | `number` | Hoogte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
|  |  | De verhouding blijft altijd behouden als: <ul> <li> Zowel `width` als `height` worden opgegeven, dan past de afbeelding in de grootte terwijl de hoogte-breedteverhouding behouden blijft </li><li> Alleen `width` of alleen `height` opgegeven. De resulterende afbeelding gebruikt de corresponderende dimensie terwijl de hoogte-breedteverhouding behouden blijft</li><li> Als dit niet het geval is `width` of `height` wordt opgegeven, wordt de pixelgrootte van de oorspronkelijke afbeelding gebruikt. Het hangt van het brontype af. Voor sommige indelingen, zoals PDF-bestanden, wordt een standaardgrootte gebruikt. Er kan een maximumgrootte zijn.</li></ul> |  |
| `quality` | `number` | Geef JPEG-kwaliteit op in het bereik van `1` tot `100`. Alleen van toepassing op afbeeldingsuitvoeringen. | `90` |
| `xmp` | `string` | Wordt alleen gebruikt door XMP terugverwijzing van metagegevens. Het is XMP base64 gecodeerd om terug te schrijven naar de opgegeven uitvoering. |  |
| `interlace` | `bool` | Maak geïnterlinieerde PNG- of GIF- of progressieve JPEG-bestanden door deze in te stellen op `true`. Dit heeft geen invloed op andere bestandsindelingen. |  |
| `jpegSize` | `number` | De grootte van het JPEG-bestand in bytes benaderen. Alle `quality` instellingen worden genegeerd. Heeft geen effect op andere indelingen. |  |
| `dpi` | `number` or `object` | Stel de DPI voor x en y in. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. Dit heeft geen invloed op de afbeelding zelf. | `96` or `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` or `object` | x- en y-DPI-waarden opnieuw samplen terwijl de fysieke grootte behouden blijft. Voor de eenvoud kan de waarde ook worden ingesteld op één getal dat voor zowel x als y wordt gebruikt. | `96` or `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lijst met bestanden die moeten worden opgenomen in het ZIP-archief (`fmt=zip`). Elk item kan een URL-tekenreeks zijn of een object met de velden:<ul><li>`url`: URL om bestand te downloaden</li><li>`path`: Bestand onder dit pad opslaan in ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Dubbele verwerking voor ZIP-archieven (`fmt=zip`). Standaard wordt een fout gegenereerd bij meerdere bestanden die in het ZIP-bestand onder hetzelfde pad zijn opgeslagen. Als u deze instelling instelt `duplicate` `ignore` op, wordt alleen het eerste element opgeslagen en de rest genegeerd. | `ignore` |
| `watermark` | `object` | Bevat instructies over het [watermerk](#watermark-specific-fields). |  |

### Watermerkspecifieke velden {#watermark-specific-fields}

De PNG-indeling wordt gebruikt als een watermerk.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Schaal van het watermerk tussen `0.0` en `1.0`. `1.0` betekent dat het watermerk de oorspronkelijke schaal (1:1) heeft en de laagste waarden de grootte van het watermerk verminderen. | Een waarde van `0.5` betekent de helft van de oorspronkelijke grootte. |
| `image` | `url` | URL naar het PNG-bestand dat moet worden gebruikt voor het watermerk. |  |

## Asynchrone gebeurtenissen {#asynchronous-events}

Zodra de verwerking van een vertoning is voltooid of wanneer een fout optreedt, wordt een gebeurtenis verzonden naar een [Adobe I/O Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Clients moeten luisteren naar de journaal-URL die via [/register](#register)wordt aangeboden. De journaalreactie bevat een `event` array die bestaat uit één object voor elke gebeurtenis, waarvan het `event` veld de werkelijke gebeurtenislading bevat.

Het type Adobe I/O-gebeurtenis voor alle gebeurtenissen van het [!DNL Asset Compute Service] is `asset_compute`. Het dagboek wordt automatisch ingetekend op dit gebeurtenistype slechts en er is geen verdere vereiste om te filtreren die op het type van Gebeurtenis Adobe I/O wordt gebaseerd. De service-specifieke gebeurtenistypen zijn beschikbaar in de `type` eigenschap van de gebeurtenis.

### Gebeurtenistypen {#event-types}

| Gebeurtenis | Beschrijving |
|---------------------|-------------|
| `rendition_created` | Verzonden voor elke correct verwerkte en geüploade vertoning. |
| `rendition_failed` | Verzonden voor elke uitvoering die niet kon worden verwerkt of geüpload. |

### Gebeurteniskenmerken {#event-attributes}

| Kenmerk | Type | Gebeurtenis | Beschrijving |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tijdstempel wanneer de gebeurtenis is verzonden in de vereenvoudigde uitgebreide [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) -indeling, zoals gedefinieerd door JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | De aanvraag-id van de oorspronkelijke aanvraag naar `/process`, net als de `X-Request-Id` header. |
| `source` | `object` | `*` | Het `source` verzoek `/process` . |
| `userData` | `object` | `*` | De waarde `userData` van de vertoning van het `/process` verzoek, indien ingesteld. |
| `rendition` | `object` | `rendition_*` | Het overeenkomende weergaveobject dat wordt doorgegeven `/process`. |
| `metadata` | `object` | `rendition_created` | De [metagegevenseigenschappen](#metadata) van de vertoning. |
| `errorReason` | `string` | `rendition_failed` | Eventuele [oorzaak](#error-reasons) van een renderingsfout. |
| `errorMessage` | `string` | `rendition_failed` | Tekst die meer details geeft over de eventuele uitvoerfout. |

### Metadata {#metadata}

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
