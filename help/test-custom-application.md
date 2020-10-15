---
title: Aangepaste toepassing testen en [!DNL Asset Compute Service] fouten opsporen.
description: Aangepaste toepassing testen en [!DNL Asset Compute Service] fouten opsporen.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# Een aangepaste toepassing testen en fouten hierin opsporen {#test-debug-custom-worker}

## Eenheidstests uitvoeren voor een aangepaste toepassing {#test-custom-worker}

Installeer [Docker Desktop](https://www.docker.com/get-started) op uw computer. Als u een aangepaste worker wilt testen, voert u de volgende opdracht uit in de hoofdmap van de toepassing:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Dit stelt een kader van de de testtest van de douaneeenheid voor de toepassingsacties van de Compute van Activa in het project in werking zoals hieronder beschreven. Deze wordt door een configuratie in het `package.json` bestand gekoppeld. Het is ook mogelijk om JavaScript eenheidstests zoals Jest te hebben. `aio app test` voert beide uit.

De [insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule voor de insteekmodule](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) voor de ontwikkeling wordt ingesloten in de aangepaste toepassing, zodat deze niet hoeft te worden geïnstalleerd op systemen voor samenstellen/testen.

### Testframework voor toepassingseenheid {#unit-test-framework}

Met het testframework voor de toepassing Asset Compute kunt u toepassingen testen zonder code te schrijven. Het is afhankelijk van het principe van de bron naar het weergavebestand van toepassingen. Er moet een bepaalde bestands- en mapstructuur worden ingesteld om testcase te definiëren met testbronbestanden, optionele parameters, verwachte uitvoeringen en aangepaste validatiescripts. Standaard worden de uitvoeringen vergeleken voor de gelijkheid van bytes. Bovendien kunnen externe HTTP-services eenvoudig worden gecontroleerd met behulp van eenvoudige JSON-bestanden.

### Tests toevoegen {#add-tests}

Tests worden verwacht in de `test` map op het hoofdniveau van het AIO-project. De testgevallen voor elke toepassing moeten in het pad staan `test/asset-compute/<worker-name>`, met één map voor elk testgeval:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Bekijk [voorbeelden van aangepaste toepassingen](https://github.com/adobe/asset-compute-example-workers/) . Hieronder vindt u een gedetailleerde referentie.

### Uitvoer testen {#test-output}

De gedetailleerde testuitvoer, inclusief de logboeken van de aangepaste toepassing, wordt beschikbaar gesteld in de `build` map in de hoofdmap van de Firefly-app, zoals wordt getoond in de `aio app test` uitvoer.

### Externe diensten koppelen {#mock-external-services}

Het is mogelijk om externe de dienstvraag in uw acties te controleren door `mock-<HOST_NAME>.json` dossiers in uw testgevallen te bepalen, waar HOST_NAME de gastheer is u zou willen controleren. Een geval van het voorbeeldgebruik is een toepassing die een afzonderlijke vraag aan S3 maakt. De nieuwe teststructuur ziet er als volgt uit:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Het mock-bestand is een http-reactie met JSON-indeling. Zie [deze documentatie](https://www.mock-server.com/mock_server/creating_expectations.html)voor meer informatie. Als er meerdere hostnamen zijn om te controleren, definieert u meerdere `mock-<mocked-host>.json` bestanden. Hieronder ziet u een voorbeeldmodelbestand voor de `google.com` naam `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

Het voorbeeld `worker-animal-pictures` bevat een [modelbestand](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) voor de Wikimedia-service waarmee het communiceert.

#### Bestanden delen over testgevallen {#share-files-across-test-cases}

U wordt aangeraden relatieve symmetrieën te gebruiken als u `file.*`of `params.json` `validate` scripts in meerdere tests deelt. Ze worden ondersteund met git. Geef uw gedeelde bestanden een unieke naam, omdat u mogelijk andere bestanden hebt. In het onderstaande voorbeeld mengen en vergelijken de tests een paar gedeelde bestanden en hun eigen bestanden:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Verwachte fouten testen {#test-unexpected-errors}

De testgevallen van de fout zouden geen verwacht `rendition.*` dossier moeten bevatten en zouden de verwachte `errorReason` binnen het `params.json` dossier moeten bepalen.

Fout bij testen hoofdletterstructuur:

```json
<error_test_case>/
    file.jpg
    params.json
```

Parameterbestand met reden van fout:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Zie volledige lijst en beschrijving van de [oorzaak](https://github.com/adobe/asset-compute-commons#error-reasons)van fouten bij het berekenen van bedrijfsmiddelen.

## Fouten opsporen in een aangepaste toepassing {#debug-custom-worker}

De volgende stappen tonen hoe u uw douanetoepassing kunt zuiveren gebruikend de Code van Visual Studio. Het staat voor het zien van levende logboeken, raakbreekpunten en stap door code evenals het levende opnieuw laden van lokale codeveranderingen na elke activering toe.

Veel van deze stappen worden gewoonlijk geautomatiseerd door `aio` uit de doos, zie sectie het Zuiveren van de Toepassing in de [Firefly documentatie](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Tot nu toe bevatten de onderstaande stappen een tijdelijke oplossing.

1. Installeer de recentste [wskdebug](https://github.com/apache/openwhisk-wskdebug) van GitHub en facultatieve [ingrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Voeg aan uw gebruikersinstellingen JSON-bestand toe. Het blijft het oude debugger van de Code van VS gebruiken, nieuwe heeft [sommige kwesties](https://github.com/apache/openwhisk-wskdebug/issues/74) met wskdebug: `"debug.javascript.usePreview": false`.
1. Sluit alle exemplaren van apps die via `aio app run`het venster zijn geopend.
1. De meest recente code implementeren met `aio app deploy`.
1. Alleen de Asset compute Devtool uitvoeren met `npx adobe-asset-compute devtool`. Houd het open.
1. In de redacteur van de Code van VS, voeg hieronder configuratie zuivert aan uw `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Haal de naam van de handeling op uit de uitvoer van `aio app deploy`. Het ziet er zo uit `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`.

1. Selecteer `wskdebug worker` van de looppas/zuivert configuratie en druk het speel pictogram. Wacht tot het begint tot het **[!UICONTROL Klaar voor activering]** in het venster van de Console **** Debug toont.

1. Klik op **[!UICONTROL Uitvoeren]** in het gereedschap Ontwikkelen. U kunt de acties zien die in de redacteur van de Code van VS worden uitgevoerd en het logboeken begint te tonen.

1. Stel een onderbrekingspunt in de code in, voer het opnieuw uit en druk op.

Eventuele codewijzigingen worden in real-time geladen en worden van kracht zodra de volgende activering plaatsvindt.

>[!NOTE]
>
>Er zijn twee activeringen voor elke aanvraag in aangepaste toepassingen. Het eerste verzoek is een webactie die zichzelf asynchroon aanroept in de SDK-code. De tweede activering is de activering die raakt aan uw code.
