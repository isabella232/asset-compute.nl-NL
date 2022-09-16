---
title: Ontwikkelen voor [!DNL Asset Compute Service]
description: Aangepaste toepassingen maken met [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: a121b48d480b45405259c2061ac86b9ab46b89cb
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Een aangepaste toepassing ontwikkelen {#develop}

Voordat u begint met het ontwikkelen van een aangepaste toepassing:

* Zorg ervoor dat alle [voorwaarden](/help/understand-extensibility.md#prerequisites-and-provisioning) is voldaan.
* Installeer de [vereiste softwaretools](/help/setup-environment.md#create-dev-environment).
* Zie [uw omgeving instellen](setup-environment.md) om er zeker van te zijn dat u klaar bent om een aangepaste toepassing te maken.

## Een aangepaste toepassing maken {#create-custom-application}

Zorg ervoor dat u de [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) lokaal geïnstalleerd.

1. Als u een aangepaste toepassing wilt maken, [een App Builder-project maken](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Hiertoe voert u uit `aio app init <app-name>` in uw terminal.

   Als u zich nog niet hebt aangemeld, wordt u met deze opdracht gevraagd zich aan te melden bij de [Adobe Developer Console](https://console.adobe.io/) met uw Adobe ID. Zie [hier](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) voor meer informatie over aanmelden via de cli.

   Adobe raadt u aan zich aan te melden. Als u problemen hebt, volgt u de instructies [om een app te maken zonder u aan te melden](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Na het programma openen, volg de herinneringen in CLI en selecteer `Organization`, `Project`, en `Workspace` gebruiken voor de toepassing. Kies het project en de werkruimte u creeerde toen u [uw omgeving instellen](setup-environment.md). Als dit wordt gevraagd `Which extension point(s) do you wish to implement ?`, zorg ervoor dat u `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Als u hierom wordt gevraagd `Which Adobe I/O App features do you want to enable for this project?`, selecteert u `Actions`. Zorg ervoor dat u de selectie opheft `Web Assets` gebruiken verschillende verificatie- en autorisatiecontroles.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Als dit wordt gevraagd `Which type of sample actions do you want to create?`, zorg ervoor dat u `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Volg de rest herinneringen en open de nieuwe toepassing in de Code van Visual Studio (of uw favoriete coderedacteur). Het bevat de basiscode en voorbeeldcode voor een aangepaste toepassing.

   Lees hier over de [hoofdcomponenten van een App Builder-app](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   De sjabloontoepassing gebruikt onze [asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) voor het uploaden, downloaden en ordenen van toepassingsuitvoeringen hoeven ontwikkelaars alleen de aangepaste toepassingslogica te implementeren. Binnen de `actions/<worker-name>` map, de `index.js` In dit bestand kunt u de aangepaste toepassingscode toevoegen.

Zie [voorbeeld aangepaste toepassingen](#try-sample) voor voorbeelden en ideeën voor aangepaste toepassingen.

### Referenties toevoegen {#add-credentials}

Terwijl u zich aanmeldt bij het maken van de toepassing, worden de meeste gegevens van App Builder verzameld in uw ENV-bestand. Voor het gebruik van het ontwikkelaarsgereedschap zijn echter aanvullende gegevens vereist.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Inloggegevens voor opslag van het gereedschap Ontwikkelaar {#developer-tool-credentials}

Het hulpprogramma voor ontwikkelaars dat wordt gebruikt om aangepaste toepassingen te testen met het daadwerkelijke [!DNL Asset Compute service] vereist een cloudopslagcontainer voor het hosten van testbestanden en voor het ontvangen en weergeven van uitvoeringen die door toepassingen worden gegenereerd.

>[!NOTE]
>
>Dit staat los van de cloudopslag van [!DNL Adobe Experience Manager] als [!DNL Cloud Service]. Dit geldt alleen voor het ontwikkelen en testen met het Asset compute developer tool.

Zorg ervoor dat u toegang hebt tot een [ondersteunde cloudopslagcontainer](https://github.com/adobe/asset-compute-devtool#prerequisites). Deze container kan door veelvoudige ontwikkelaars over verschillende projecten worden gedeeld zoals nodig.

#### Referenties toevoegen aan ENV-bestand {#add-credentials-env-file}

Voeg de volgende geloofsbrieven voor het ontwikkelaarshulpmiddel aan het ENV dossier in de wortel van uw project App Builder toe:

1. Voeg het absolute pad toe aan het gemaakte persoonlijke sleutelbestand terwijl u services toevoegt aan uw App Builder-project:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Download het bestand uit de Adobe Developer-console. Ga naar de hoofdmap van het project en klik op Alles downloaden rechtsboven in het scherm. Het bestand wordt gedownload met `<namespace>-<workspace>.json` als de bestandsnaam. Voer een van de volgende handelingen uit:

   * De naam van het bestand wijzigen als `console.json` en verplaatst u het in de hoofdmap van uw project.
   * U kunt desgewenst het absolute pad toevoegen aan het JSON-bestand voor integratie in Adobe Developer Console. Dit is hetzelfde [`console.json`](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) bestand dat wordt gedownload in uw projectwerkruimte.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. Voeg S3 of Azure opslaggeloofsbrieven toe. U hebt slechts toegang tot één cloudopslagoplossing nodig.

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

>[!TIP]
>
>De `config.json` bestand bevat referenties. Voeg vanuit uw project het JSON-bestand toe aan uw `.gitignore` om delen te voorkomen. Hetzelfde geldt voor de .env- en .aio-bestanden.

## De toepassing uitvoeren {#run-custom-application}

Voordat u de toepassing uitvoert met het Asset compute Developer Tool, moet u de [geloofsbrieven](#developer-tool-credentials).

Als u de toepassing wilt uitvoeren in het hulpprogramma voor ontwikkelaars, gebruikt u `aio app run` gebruiken. De handeling wordt geïmplementeerd op [!DNL Adobe I/O] Runtime en begin het ontwikkelingshulpmiddel op uw lokale machine. Dit hulpmiddel wordt gebruikt om toepassingsverzoeken tijdens ontwikkeling te testen. Hier volgt een voorbeeld van een verzoek om uitvoering:

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
>Gebruik de `--local` met de `run` gebruiken. Het werkt niet met [!DNL Asset Compute] aangepaste toepassingen en het Asset compute Developer tool. Aangepaste toepassingen worden aangeroepen door de [!DNL Asset Compute Service] die geen toegang hebben tot handelingen die op de lokale machines van de ontwikkelaar worden uitgevoerd.

Zie [hier](test-custom-application.md) hoe te om uw toepassing te testen en te zuiveren. Wanneer u klaar bent met het ontwikkelen van uw aangepaste toepassing, [uw aangepaste toepassing implementeren](deploy-custom-application.md).

## Probeer de voorbeeldtoepassing van Adobe {#try-sample}

Hieronder vindt u voorbeelden van aangepaste toepassingen:

* [basisch voor werknemers](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [dierenfoto&#39;s van werknemers](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aangepaste toepassing sjabloon {#template-custom-application}

De [basisch voor werknemers](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) is een sjabloontoepassing. Het produceert een vertoning door het brondossier eenvoudig te kopiëren. De inhoud van deze toepassing is de sjabloon die wordt ontvangen bij het kiezen van `Adobe Asset Compute` bij het maken van de aio-app.

Het toepassingsbestand, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) gebruikt de [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) om het bronbestand te downloaden, elke verwerking van de vertoning te ordenen en de resulterende uitvoeringen weer naar de cloudopslag te uploaden.

De [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) is gedefinieerd in de toepassingscode, waar alle logica voor toepassingsverwerking wordt uitgevoerd. De renditiecallback in `worker-basic` kopieert eenvoudig de inhoud van het bronbestand naar het weergavebestand.

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

De [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) doet een ophaalverzoek aan een statische URL van Wikimedia gebruikend [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) bibliotheek.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Aangepaste parameters doorgeven {#pass-custom-parameters}

U kunt aangepaste gedefinieerde parameters doorgeven via de weergaveobjecten. Er kan binnen de toepassing naar worden verwezen in [`rendition` instructies](https://github.com/adobe/asset-compute-sdk#rendition). Een voorbeeld van een weergaveobject is:

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

Door gebrek, komen de de douanetoepassingen van de Asset compute met de controles van de Vergunning en van de Authentificatie voor het project van de Bouwer van de App. Dit wordt ingeschakeld door het instellen van de `require-adobe-auth` aantekening `true` in de `manifest.yml`.

### Andere Adobe-API&#39;s openen {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

De API-services toevoegen aan de [!DNL Asset Compute] Console-werkruimte gemaakt in Setup. Deze diensten maken deel uit van het JWT toegangstoken dat door wordt geproduceerd [!DNL Asset Compute Service]. De token en andere gegevens zijn toegankelijk binnen de handeling van de toepassing `params` object.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Referenties doorgeven voor systemen van derden {#pass-credentials-for-tp}

Om geloofsbrieven voor andere externe diensten te behandelen, ga deze als standaardparameters over de acties. Deze worden automatisch tijdens de doorvoer versleuteld. Zie voor meer informatie [handelingen maken in de handleiding voor ontwikkelaars van runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). Stel ze vervolgens in met behulp van omgevingsvariabelen tijdens de implementatie. Deze parameters zijn toegankelijk in het dialoogvenster `params` in de handeling.

Stel de standaardparameters in de `inputs` in de `manifest.yml`:

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

De `$VAR` expressie leest de waarde van een omgevingsvariabele met de naam `VAR`.

Tijdens de ontwikkeling kan de waarde als volgt in het lokale ENV-bestand worden ingesteld `aio` Hiermee worden omgevingsvariabelen van ENV-bestanden automatisch gelezen naast de variabelen die zijn ingesteld vanuit de aanroepende shell. In dit voorbeeld ziet het ENV-bestand er als volgt uit:

```CONF
#...
SECRET_KEY=secret-value
```

Voor productieplaatsing zou men de milieuvariabelen in het systeem van CI kunnen plaatsen, bijvoorbeeld gebruikend geheimen in Acties GitHub. Tot slot hebt u als volgt toegang tot de standaardparameters binnen de toepassing:

```javascript
const key = params.secretKey;
```

## Toepassingen vergroten/verkleinen {#sizing-workers}

Een toepassing wordt uitgevoerd in een container in [!DNL Adobe I/O] Runtime met [limieten](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) dat door kan worden gevormd `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Wegens de meer uitgebreide verwerking typisch die door de toepassingen van de Asset compute wordt gedaan, is het waarschijnlijker dat men deze grenzen voor optimale prestaties (groot genoeg om binaire activa te behandelen) en efficiency (niet verspilend middelen wegens ongebruikt containergeheugen) moet aanpassen.

De standaardtime-out voor acties in Runtime is een minuut, maar deze kan worden verhoogd door het instellen van de opdracht `timeout` limiet (in milliseconden). Verhoog deze tijd als u grotere bestanden wilt verwerken. Houd rekening met de totale tijd die nodig is om de bron te downloaden, het bestand te verwerken en de vertoning te uploaden. Als een handeling uitvalt, d.w.z. de activering niet vóór de opgegeven time-outlimiet retourneert, verwijdert Runtime de container en gebruikt deze niet opnieuw.

De toepassingen van de asset compute door aard neigen om netwerk en schijfInput of output gebonden te zijn. Het bronbestand moet eerst worden gedownload, de verwerking is vaak bronintensief en vervolgens worden de resulterende uitvoeringen opnieuw geüpload.

Het geheugen dat beschikbaar is voor een handelingencontainer wordt opgegeven door `memorySize` in MB. Momenteel bepaalt dit ook hoeveel toegang van cpu tot de container krijgt, en het belangrijkste is het een zeer belangrijk element van de kosten om Runtime (grotere containers kosten meer) te gebruiken. Gebruik hier een grotere waarde wanneer uw verwerking meer geheugen of cpu vereist maar ben voorzichtig om geen middelen te verspillen aangezien groter de containers zijn, lager de algemene productie is.

Bovendien is het mogelijk de gelijktijdige handelingen binnen een container te controleren met behulp van de `concurrency` instellen. Dit is het aantal gelijktijdige activeringen dat één container (van dezelfde handeling) krijgt. In dit model, is de actiecontainer als server Node.js die veelvoudige gezamenlijke verzoeken, tot die grens ontvangt. Als niet geplaatst, is het gebrek in Runtime 200, dat voor kleinere acties App Builder, maar gewoonlijk te groot voor Asset compute toepassingen gezien hun intensievere lokale verwerking en schijfactiviteit groot is. Sommige toepassingen, afhankelijk van hun implementatie, werken mogelijk ook niet goed met gelijktijdige activiteit. De Asset compute-SDK zorgt ervoor dat de activeringen worden gescheiden door bestanden naar verschillende unieke mappen te schrijven.

Test toepassingen om de optimale getallen te vinden voor `concurrency` en `memorySize`. Grotere containers = de hogere geheugengrens kon voor meer gelijktijdige uitvoering toestaan maar kon ook verkwistend voor lager verkeer zijn.
