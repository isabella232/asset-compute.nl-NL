---
title: Begrijp het werk van een douanetoepassing.
description: Interne werking [!DNL Asset Compute Service] van een aangepaste toepassing voor een beter begrip van de werking ervan.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# Interfaces van een aangepaste toepassing {#how-custom-application-works}

Gebruik de volgende illustratie om inzicht te krijgen in de end-to-end workflow wanneer een digitaal element wordt verwerkt met een aangepaste toepassing van een client.

![Aangepaste workflow voor toepassingen](assets/customworker.png)

*Afbeelding: Stappen die nodig zijn om een middel te verwerken met behulp van [!DNL Asset Compute Service].*

## Registratie {#registration}

De cliënt moet [`/register`](api.md#register) één keer vóór het eerste verzoek roepen om [`/process`](api.md#process-request) de dagboek URL voor het ontvangen van de Gebeurtenissen van Adobe I/O voor de Compute van de Activa van Adobe op te zetten en terug te winnen.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

De [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript-bibliotheek kan in NodeJS-toepassingen worden gebruikt om alle noodzakelijke stappen af te handelen, van registratie tot asynchrone gebeurtenisafhandeling. Voor meer informatie over de vereiste kopballen, zie [Authentificatie en Vergunning](api.md).

## Verwerking {#processing}

De client verzendt een [verwerkingsverzoek](api.md#process-request) .

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

De client is verantwoordelijk voor de juiste opmaak van de uitvoeringen met vooraf ondertekende URL&#39;s. De [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript-bibliotheek kan in NodeJS-toepassingen worden gebruikt om URL&#39;s vooraf te ondertekenen. Momenteel ondersteunt de bibliotheek alleen Azure Blob Storage en AWS S3 Containers.

Het verwerkingsverzoek keert een terug `requestId` die voor het opiniepeilen van Adobe I/O Gebeurtenissen kan worden gebruikt.

Hieronder ziet u een voorbeeld van een aangepaste aanvraag voor de verwerking van toepassingen.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

De [!DNL Asset Compute Service] gebruiker verzendt de aanvragen voor de aangepaste uitvoering van de toepassing naar de aangepaste toepassing. Er wordt een HTTP-POST gebruikt naar de opgegeven toepassings-URL, de beveiligde webactie-URL van Project Firefly. Voor alle aanvragen wordt het HTTPS-protocol gebruikt om de gegevensbeveiliging te maximaliseren.

De [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) die door een aangepaste toepassing wordt gebruikt, handelt de HTTP-aanvraag af. Het behandelt ook het downloaden van de bron, het uploaden van uitvoeringen, het verzenden van I/O-gebeurtenissen en foutafhandeling.

<!-- TBD: Add the application diagram. -->

### Toepassingscode {#application-code}

De code van de douane moet slechts callback verstrekken die het plaatselijk beschikbare brondossier (`source.path`) neemt. Het `rendition.path` is de locatie waar het eindresultaat van een aanvraag voor de verwerking van bedrijfsmiddelen moet worden geplaatst. De aangepaste toepassing gebruikt de callback om de lokaal beschikbare bronbestanden om te zetten in een weergavebestand met de naam die wordt doorgegeven (`rendition.path`). Een aangepaste toepassing moet schrijven naar `rendition.path` om een uitvoering te maken:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Bronbestanden downloaden {#download-source}

Een aangepaste toepassing heeft alleen betrekking op lokale bestanden. Het downloaden van het bronbestand wordt afgehandeld door de SDK [Asset Compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Vertoning maken {#rendition-creation}

De SDK roept een asynchrone callback-functie [voor](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) uitvoering aan voor elke uitvoering.

De callback-functie heeft toegang tot de [bron](https://github.com/adobe/asset-compute-sdk#source) - en [weergaveobjecten](https://github.com/adobe/asset-compute-sdk#rendition) . Het `source.path` bestaat al en is het pad naar de lokale kopie van het bronbestand. Het `rendition.path` is het pad waar de verwerkte vertoning moet worden opgeslagen. Tenzij de markering [disableSourceDownload is ingesteld](https://github.com/adobe/asset-compute-sdk#worker-options-optional) , moet de toepassing exact de markering gebruiken `rendition.path`. Anders kan de SDK het weergavebestand niet vinden of identificeren en mislukt.

De overdreven vereenvoudiging van het voorbeeld wordt gedaan om de anatomie van een douanetoepassing te illustreren en te concentreren. De toepassing kopieert het bronbestand alleen naar de weergavebestemming.

Zie [Asset Compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details)voor meer informatie over de callback-parameters van de uitvoering.

### Uitvoeringen uploaden {#upload-rendition}

Nadat elke vertoning is gemaakt en opgeslagen in een bestand met het pad dat is opgegeven door `rendition.path`, uploadt de [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) elke uitvoering naar een cloudopslag (AWS of Azure). Een aangepaste toepassing krijgt meerdere uitvoeringen tegelijkertijd als en alleen als de binnenkomende aanvraag meerdere uitvoeringen bevat die verwijzen naar dezelfde toepassings-URL. Het uploaden naar de cloudopslag vindt plaats na elke uitvoering en voordat de callback voor de volgende uitvoering wordt uitgevoerd.

Het `batchWorker()` heeft een ander gedrag, aangezien het eigenlijk alle vertoningen verwerkt en slechts nadat al verwerkt is uploadt die.

## Adobe I/O-gebeurtenissen {#aio-events}

De SDK verzendt Adobe I/O-gebeurtenissen voor elke uitvoering. Deze gebeurtenissen zijn van het type `rendition_created` of `rendition_failed` afhankelijk van het resultaat. Zie Asynchrone gebeurtenissen [voor het berekenen van](api.md#asynchronous-events) middelen voor gebeurtenisdetails.

## Adobe I/O-gebeurtenissen ontvangen {#receive-aio-events}

De klant pollt het [Adobe I/O Events Journal](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) volgens de logica van zijn verbruik. De eerste journaal-URL is de URL die wordt opgegeven in de API-reactie. `/register` Gebeurtenissen kunnen worden geïdentificeerd met de gebeurtenissen `requestId` die aanwezig zijn in de gebeurtenissen en zijn gelijk aan de gebeurtenissen die worden geretourneerd in `/process`. Elke vertoning heeft een aparte gebeurtenis die wordt verzonden zodra de vertoning is geüpload (of mislukt). Nadat de client een overeenkomende gebeurtenis heeft ontvangen, kan de client de resulterende uitvoeringen weergeven of op een andere manier afhandelen.

De JavaScript-bibliotheek [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) maakt het opiniepeilen van het tijdschrift eenvoudig met de `waitActivation()` methode om alle gebeurtenissen op te halen.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Zie [Adobe I/O Events API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)voor meer informatie over hoe u journaalgebeurtenissen kunt ophalen.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
