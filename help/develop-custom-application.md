---
title: Ontwikkelen voor [!DNL Asset Compute Service].
description: Aangepaste toepassingen maken met [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 127895cf1bab59546f9ba0be2b3b7a935627effb
workflow-type: tm+mt
source-wordcount: '1496'
ht-degree: 0%

---


# Een aangepaste toepassing ontwikkelen {#develop}

Voordat u begint met het ontwikkelen van een aangepaste toepassing:

* Zorg ervoor dat aan alle [voorwaarden](/help/understand-extensibility.md#prerequisites-and-provisioning) is voldaan.
* Installeer de [vereiste softwaretools](/help/setup-environment.md#create-dev-environment).
* Zie [opstelling uw milieu](setup-environment.md) om ervoor te zorgen u bereid bent om een douanetoepassing tot stand te brengen.

## Een aangepaste toepassing maken {#create-custom-application}

Zorg ervoor dat de [Adobe I/O CLI](https://github.com/adobe/aio-cli) lokaal is geïnstalleerd.

1. Als u een aangepaste toepassing wilt maken, [maakt u een Firefly-app](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli). Om dit te doen, voer `aio app init <app-name>` in uw terminal uit.

   Als u zich nog niet hebt aangemeld, wordt u met deze opdracht gevraagd zich aan te melden bij de [Adobe Developer Console](https://console.adobe.io/) met uw Adobe ID. Zie [hier](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) voor meer informatie over aanmelden vanuit de clip.

   Adobe raadt u aan zich aan te melden. Als u problemen ondervindt, volgt u de instructies [om een app te maken zonder u aan te melden](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Na het programma openen, volg de herinneringen in CLI en selecteer `Organization`, `Project`, en `Workspace` voor de toepassing te gebruiken. Kies het project en de werkruimte u creeerde toen u [opstelling uw milieu](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Selecteer ten minste `Which Adobe I/O App features do you want to enable for this project?``Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Selecteer desgevraagd `Which type of sample actions do you want to create?`het volgende `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Volg de rest herinneringen en open de nieuwe toepassing in de Code van Visual Studio (of uw favoriete coderedacteur). Het bevat de basiscode en voorbeeldcode voor een aangepaste toepassing.

   Lees hier de [hoofdcomponenten van een Firefly-app](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

   De sjabloontoepassing gebruikt onze SDK [voor](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) Asset Compute voor het uploaden, downloaden en ordenen van toepassingsuitvoeringen, zodat ontwikkelaars alleen de aangepaste toepassingslogica hoeven te implementeren. In de `actions/<worker-name>` map is het `index.js` bestand de locatie waar de aangepaste toepassingscode moet worden toegevoegd.

Zie [voorbeelden van aangepaste toepassingen](#try-sample) voor voorbeelden en ideeën voor aangepaste toepassingen.

### Referenties toevoegen {#add-credentials}

Terwijl u zich aanmeldt bij het maken van de toepassing, worden de meeste Firefly-referenties verzameld in uw ENV-bestand. Voor het gebruik van het ontwikkelaarsgereedschap zijn echter aanvullende gegevens vereist.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Inloggegevens voor opslag van het gereedschap Ontwikkelaar {#developer-tool-credentials}

Het hulpprogramma voor ontwikkelaars dat wordt gebruikt om aangepaste toepassingen te testen met de werkelijke code, [!DNL Asset Compute service] vereist een cloudopslagcontainer voor het hosten van testbestanden en voor het ontvangen en weergeven van uitvoeringen die door toepassingen worden gegenereerd.

>[!NOTE]
>
>Dit staat los van de cloudopslag van [!DNL Adobe Experience Manager] als Cloud Service. Het is alleen van toepassing voor het ontwikkelen en testen met het hulpprogramma Asset Compute Developer.

Zorg ervoor dat u toegang hebt tot een [ondersteunde container](https://github.com/adobe/asset-compute-devtool#prerequisites)voor cloudopslag. Deze container kan door veelvoudige ontwikkelaars over verschillende projecten worden gedeeld zoals nodig.

#### Referenties toevoegen aan ENV-bestand {#add-credentials-env-file}

Voeg de volgende geloofsbrieven voor het ontwikkelaarshulpmiddel aan het ENV dossier in de wortel van uw Vuurwerk project toe:

1. Voeg het absolute pad toe aan het bestand met de persoonlijke sleutel dat u hebt gemaakt terwijl u services toevoegt aan uw project Firefly:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Voeg de referenties S3 of Azure Storage toe. U hebt slechts toegang tot één cloudopslagoplossing nodig.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

## De toepassing uitvoeren {#run-custom-application}

Voordat u de toepassing uitvoert met het Hulpprogramma voor het berekenen van bedrijfsmiddelen, moet u de [referenties](#developer-tool-credentials)op de juiste wijze configureren.

Gebruik de `aio app run` opdracht om de toepassing uit te voeren in het gereedschap Ontwikkelaar. De toepassing implementeert de actie op Adobe I/O Runtime en start het hulpprogramma voor ontwikkeling op uw lokale computer. Dit hulpmiddel wordt gebruikt om toepassingsverzoeken tijdens ontwikkeling te testen. Hier volgt een voorbeeld van een verzoek om uitvoering:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Gebruik de `--local` markering niet met de `run` opdracht. De functie werkt niet met [!DNL Asset Compute] aangepaste toepassingen en het hulpprogramma Asset Compute Developer. De toepassingen van de douane worden geroepen door [!DNL Asset Compute Service] die tot geen acties kunnen toegang hebben die op de lokale machines van de ontwikkelaar lopen.

Zie [hier](test-custom-application.md) hoe u uw toepassing kunt testen en er fouten in kunt opsporen. Wanneer u klaar bent met het ontwikkelen van uw douanetoepassing, [stel uw douanetoepassing](deploy-custom-application.md)op.

## Probeer de voorbeeldtoepassing van Adobe {#try-sample}

Hieronder vindt u voorbeelden van aangepaste toepassingen:

* [basisch voor werknemers](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [dierenfoto&#39;s van werknemers](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aangepaste toepassing sjabloon {#template-custom-application}

De [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) is een sjabloontoepassing. Het produceert een vertoning door het brondossier eenvoudig te kopiëren. De inhoud van deze toepassing is de sjabloon die wordt ontvangen wanneer u kiest `Adobe Asset Compute` voor het maken van de audio-app.

Het toepassingsbestand, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) gebruikt het [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) om het bronbestand te downloaden, elke verwerking van de vertoning te ordenen en de resulterende uitvoeringen weer naar de cloudopslag te uploaden.

In de toepassingscode is [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) gedefinieerd waar alle verwerkingslogica van de toepassing wordt uitgevoerd. De rendercallback in kopieert `worker-basic` eenvoudig de inhoud van het bronbestand naar het weergavebestand.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Een externe API aanroepen {#call-external-api}

In de toepassingscode kunt u externe API-aanroepen maken die u helpen bij de verwerking van toepassingen. Hieronder ziet u een voorbeeld van een toepassingsbestand waarin externe API wordt aangeroepen.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Bijvoorbeeld, [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) doet het een haalverzoek aan een statische URL van Wikimedia gebruikend de [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) bibliotheek.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Aangepaste parameters doorgeven {#pass-custom-parameters}

U kunt aangepaste gedefinieerde parameters doorgeven via de weergaveobjecten. In de toepassing kunt u hier in [`rendition` instructies](https://github.com/adobe/asset-compute-sdk#rendition)naar verwijzen. Een voorbeeld van een weergaveobject is:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Een voorbeeld van een toepassingsbestand dat aangepaste parameters benadert, is:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

De `example-worker-animal-pictures` geeft een aangepaste parameter door [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) om te bepalen welk bestand moet worden opgehaald van Wikimedia.

## Ondersteuning voor verificatie en autorisatie {#authentication-authorization-support}

Standaard worden aangepaste toepassingen voor middelenberekening geleverd met verificatie- en verificatiecontroles voor probleemtoepassingen. Dit wordt ingeschakeld door de `require-adobe-auth` annotatie in te stellen op `true` in de `manifest.yml`.

### Andere Adobe-API&#39;s openen {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Voeg de API-services toe aan de [!DNL Asset Compute] Console-werkruimte die in Setup is gemaakt. Deze diensten maken deel uit van het JWT toegangstoken dat door wordt geproduceerd [!DNL Asset Compute Service]. Het token en andere gegevens zijn toegankelijk binnen het `params` object Handeling van de toepassing.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Referenties doorgeven voor systemen van derden {#pass-credentials-for-tp}

Om geloofsbrieven voor andere externe diensten te behandelen, ga deze als standaardparameters over de acties. Deze worden automatisch tijdens de doorvoer versleuteld. Zie Handelingen [maken in de handleiding](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)voor ontwikkelaars van runtimeprogramma voor meer informatie. Stel ze vervolgens in met behulp van omgevingsvariabelen tijdens de implementatie. Deze parameters zijn toegankelijk in het `params` object binnen de handeling.

Stel de standaardparameters in de `inputs` map in `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

De `$VAR` expressie leest de waarde uit een omgevingsvariabele met de naam `VAR`.

Tijdens de ontwikkeling, kan de waarde in het lokale ENV dossier worden geplaatst aangezien `aio` automatisch omgevingsvariabelen van ENV dossiers naast de variabelen die van het aanhalen shell worden geplaatst. In dit voorbeeld ziet het ENV-bestand er als volgt uit:

```CONF
#...
SECRET_KEY=secret-value
```

Voor productieplaatsing zou men de milieuvariabelen in het systeem van CI kunnen plaatsen, bijvoorbeeld gebruikend geheimen in Acties GitHub. Tot slot hebt u als volgt toegang tot de standaardparameters binnen de toepassing:

```javascript
const key = params.secretKey;
```

## Toepassingen vergroten/verkleinen {#sizing-workers}

Een toepassing wordt uitgevoerd in een container in Adobe I/O Runtime met [grenzen](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) die door `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Wegens de meer uitgebreide verwerking typisch die door de toepassingen van de Compute van Activa wordt gedaan, is het waarschijnlijker dat men deze grenzen voor optimale prestaties (groot genoeg om binaire activa te behandelen) en efficiency (niet verspilend middelen wegens ongebruikt containergeheugen) moet aanpassen.

De standaardtime-out voor acties in Runtime is een minuut, maar deze kan worden verhoogd door de `timeout` limiet in milliseconden in te stellen. Verhoog deze tijd als u grotere bestanden wilt verwerken. Houd rekening met de totale tijd die nodig is om de bron te downloaden, het bestand te verwerken en de vertoning te uploaden. Als een handeling uitvalt, d.w.z. de activering niet vóór de opgegeven time-outlimiet retourneert, verwijdert Runtime de container en gebruikt deze niet opnieuw.

De activa verwerken toepassingen door aard neigen om netwerk en schijfIO verbindend te zijn. Het bronbestand moet eerst worden gedownload, de verwerking is vaak IO zwaar en de resulterende uitvoeringen worden opnieuw geüpload.

Het geheugen beschikbaar aan een actiecontainer wordt gespecificeerd door `memorySize` in MB. Momenteel bepaalt dit ook hoeveel toegang van cpu tot de container krijgt, en het belangrijkste is het een zeer belangrijk element van de kosten om Runtime (grotere containers kosten meer) te gebruiken. Gebruik hier een grotere waarde wanneer uw verwerking meer geheugen of cpu vereist maar ben voorzichtig om geen middelen te verspillen aangezien groter de containers zijn, lager de algemene productie is.

Bovendien is het mogelijk om handelingsgelijktijdig binnen een container te controleren gebruikend het `concurrency` plaatsen. Dit is het aantal gelijktijdige activeringen dat één container (van dezelfde handeling) krijgt. In dit model, is de actiecontainer als server Node.js die veelvoudige gezamenlijke verzoeken, tot die grens ontvangt. Als niet geplaatst, is het gebrek in Runtime 200, dat voor kleinere acties Firefly, maar gewoonlijk te groot voor de toepassingen van de Compute van Activa gezien hun intensievere lokale verwerking en schijfactiviteit groot is. Sommige toepassingen, afhankelijk van hun implementatie, werken mogelijk ook niet goed met gelijktijdige activiteit. De SDK voor middelenberekening zorgt ervoor dat de actities worden gescheiden door bestanden naar verschillende unieke mappen te schrijven.

Test toepassingen om de optimale getallen voor `concurrency` en `memorySize`. Grotere containers = de hogere geheugengrens kon voor meer gelijktijdige uitvoering toestaan maar kon ook verkwistend voor lager verkeer zijn.
