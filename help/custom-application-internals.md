---
title: De werking van een aangepaste toepassing begrijpen
description: Het interne werk van [!DNL Asset Compute Service] douanetoepassing helpen begrijpen hoe het werkt.
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---


# Intern van een aangepaste toepassing {#how-custom-application-works}

Gebruik de volgende illustratie om inzicht te krijgen in de end-to-end workflow wanneer een digitaal element wordt verwerkt met een aangepaste toepassing van een client.

![Aangepaste workflow voor toepassingen](assets/customworker.png)

*Afbeelding: Stappen die nodig zijn om een middel te verwerken met behulp van  [!DNL Asset Compute Service].*

## Registratie {#registration}

De client moet [`/register`](api.md#register) één keer aanroepen voor het eerste verzoek aan [`/process`](api.md#process-request) om de journaal-URL voor het ontvangen van [!DNL Adobe I/O] gebeurtenissen voor Adobe-Asset compute in te stellen en op te halen.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

De JavaScript-bibliotheek [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) kan in NodeJS-toepassingen worden gebruikt om alle noodzakelijke stappen af te handelen, van registratie tot asynchrone gebeurtenisafhandeling. Voor meer informatie over de vereiste kopballen, zie [Authentificatie en Authorization](api.md).

## Bezig met verwerken {#processing}

De client verzendt een [processing](api.md#process-request)-verzoek.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

De client is verantwoordelijk voor de juiste opmaak van de uitvoeringen met vooraf ondertekende URL&#39;s. De JavaScript-bibliotheek [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) kan in NodeJS-toepassingen worden gebruikt om URL&#39;s vooraf te ondertekenen. Momenteel ondersteunt de bibliotheek alleen Azure Blob Storage en AWS S3 Containers.

Het verwerkingsverzoek keert `requestId` terug die voor het opiniepeilen [!DNL Adobe I/O] Gebeurtenissen kan worden gebruikt.

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

[!DNL Asset Compute Service] verzendt de de vertoningsverzoeken van de douanetoepassing naar de douanetoepassing. Er wordt een HTTP-POST gebruikt naar de opgegeven toepassings-URL, de beveiligde webactie-URL van Project Firefly. Voor alle aanvragen wordt het HTTPS-protocol gebruikt om de gegevensbeveiliging te maximaliseren.

De [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) die door een douanetoepassing wordt gebruikt behandelt het verzoek van de POST van HTTP. Het behandelt ook het downloaden van de bron, het uploaden van uitvoeringen, het verzenden van [!DNL Adobe I/O] gebeurtenissen en fout behandeling.

<!-- TBD: Add the application diagram. -->

### Toepassingscode {#application-code}

De code van de douane moet slechts callback verstrekken die het plaatselijk beschikbare brondossier (`source.path`) neemt. `rendition.path` is de plaats om het definitieve resultaat van een verzoek van de activaverwerking te plaatsen. De douanetoepassing gebruikt callback om de plaatselijk beschikbare brondossiers in een vertoningsdossier te veranderen gebruikend de binnen overgegaan naam (`rendition.path`). Een aangepaste toepassing moet naar `rendition.path` schrijven om een vertoning te maken:

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

### Bronbestanden {#download-source} downloaden

Een aangepaste toepassing heeft alleen betrekking op lokale bestanden. Het downloaden van het bronbestand wordt afgehandeld door de [Asset compute-SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Vertoning maken {#rendition-creation}

De SDK roept een asynchrone [callback functie van de vertoning](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) voor elke vertoning aan.

De callback functie heeft toegang tot [source](https://github.com/adobe/asset-compute-sdk#source) en [rendition](https://github.com/adobe/asset-compute-sdk#rendition) voorwerpen. De `source.path` bestaat al en is het pad naar de lokale kopie van het bronbestand. De `rendition.path` is het pad waar de verwerkte vertoning moet worden opgeslagen. Tenzij de [disableSourceDownload vlag](https://github.com/adobe/asset-compute-sdk#worker-options-optional) wordt geplaatst, moet de toepassing precies `rendition.path` gebruiken. Anders kan de SDK het weergavebestand niet vinden of identificeren en mislukt.

De overdreven vereenvoudiging van het voorbeeld wordt gedaan om de anatomie van een douanetoepassing te illustreren en te concentreren. De toepassing kopieert het bronbestand alleen naar de weergavebestemming.

Voor meer informatie over de parameters van de vertoningscallback, zie [Asset compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### Uitvoeringen {#upload-rendition} uploaden

Nadat elke vertoning is gemaakt en opgeslagen in een bestand met het pad dat is opgegeven door `rendition.path`, uploadt de [Asset compute-SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) elke uitvoering naar een cloudopslag (AWS of Azure). Een aangepaste toepassing krijgt meerdere uitvoeringen tegelijkertijd als en alleen als de binnenkomende aanvraag meerdere uitvoeringen bevat die verwijzen naar dezelfde toepassings-URL. Het uploaden naar de cloudopslag vindt plaats na elke uitvoering en voordat de callback voor de volgende uitvoering wordt uitgevoerd.

De `batchWorker()` heeft een ander gedrag, aangezien het eigenlijk alle vertoningen verwerkt en slechts nadat allen zijn verwerkt uploadt die.

## [!DNL Adobe I/O] Gebeurtenissen {#aio-events}

De SDK verzendt [!DNL Adobe I/O]-gebeurtenissen voor elke uitvoering. Deze gebeurtenissen zijn van het type `rendition_created` of `rendition_failed` afhankelijk van het resultaat. Zie [Asset compute asynchrone gebeurtenissen](api.md#asynchronous-events) voor gebeurtenisdetails.

## [!DNL Adobe I/O] Gebeurtenissen {#receive-aio-events} ontvangen

De client pollt het [[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) volgens de logica van zijn verbruik. De eerste journaal-URL is de URL die wordt opgegeven in de API-reactie `/register`. Gebeurtenissen kunnen worden geïdentificeerd met de `requestId` die aanwezig is in de gebeurtenissen en die hetzelfde is als geretourneerd in `/process`. Elke vertoning heeft een aparte gebeurtenis die wordt verzonden zodra de vertoning is geüpload (of mislukt). Nadat de client een overeenkomende gebeurtenis heeft ontvangen, kan de client de resulterende uitvoeringen weergeven of op een andere manier afhandelen.

De bibliotheek [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) maakt de dagboekopiniepeiling eenvoudig gebruikend de `waitActivation()` methode om alle gebeurtenissen te krijgen.

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

Zie [[!DNL Adobe I/O] Gebeurtenissen API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml) voor meer informatie over hoe u journaalgebeurtenissen kunt ophalen.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
