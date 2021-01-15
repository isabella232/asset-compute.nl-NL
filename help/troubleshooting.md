---
title: Problemen oplossen [!DNL Asset Compute Service].
description: Los en zuiver douanetoepassingen problemen op gebruikend [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 7e520921ebb459c963d61d70c66497b8e62521cf
workflow-type: tm+mt
source-wordcount: '288'
ht-degree: 1%

---


# Problemen oplossen {#troubleshoot}

Enkele algemene tips voor het oplossen van problemen die u kunnen helpen problemen op te lossen met de service Asset compute zijn:

* Zorg ervoor dat de JavaScript-toepassing niet vastloopt bij het opstarten. Dergelijke neerstortingen zijn gewoonlijk verwant aan een ontbrekende bibliotheek of een gebiedsdeel.
* Zorg ervoor dat in het `package.json`-bestand van de toepassing naar alle te installeren afhankelijkheden wordt verwezen.
* Verzeker om het even welke fouten die uit schoonmaakbeurt bij mislukking kunnen voortkomen niet hun eigen fouten produceren die het originele probleem verbergen.

* Wanneer het beginnen van het ontwikkelaarshulpmiddel voor het eerst met nieuwe [!DNL Asset Compute Service] integratie, kan het ontbreken het eerste verwerkingsverzoek omdat het Dagboek van de Gebeurtenissen van de Asset compute niet volledig opstelling kan zijn. Wacht enige tijd op het dagboek aan opstelling alvorens een ander verzoek te verzenden.
* Als er fouten optreden bij het verzenden van Asset compute `/register`- of `/process`-aanvragen, moet u ervoor zorgen dat alle benodigde API&#39;s worden toegevoegd aan het [!DNL Adobe I/O]-project en de werkruimte—dat wil zeggen, Asset compute, [!DNL Adobe I/O] Events, [!DNL Adobe I/O] Events Management en [!DNL Adobe I/O] Runtime.

## Problemen met aanmelden via [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Als u kwesties het programma openen aan [!DNL Adobe Developer Console] [door  [!DNL Adobe I/O] CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) hebt, dan voeg manueel de geloofsbrieven toe die voor het ontwikkelen, het testen, en het opstellen van uw douanetoepassing worden vereist:

1. Navigeer naar uw Firefly-project en -werkruimte op de [Adobe Developer Console](https://console.adobe.io/) en druk **[!UICONTROL Download]** in de rechterbovenhoek. Open dit bestand en sla deze JSON op een veilige plaats op uw computer op.

1. Navigeer naar het ENV-bestand in uw Firefly-toepassing.

1. Voeg de [!DNL Adobe I/O] Runtime Credentials toe. Download de [!DNL Adobe I/O] A Runtime geloofsbrieven van gedownloade JSON. De referenties zijn onder `project.workspace.services.runtime`. Voeg de [!DNL Adobe I/O] geloofsbrieven van Runtime in `AIO_runtime_XXX` variabelen toe:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Voeg het absolute pad toe aan de gedownloade JSON in Stap 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Stel de overige [vereiste referenties](develop-custom-application.md) in die nodig zijn voor het ontwikkelaarsgereedschap.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
