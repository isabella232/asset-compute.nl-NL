---
title: De ontwikkelomgeving instellen die vereist is voor [!DNL Asset Compute Service]
description: Ontwikkelomgeving instellen voor [!DNL Asset Compute Service] om aangepaste code te maken en te testen.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 0%

---

# Een ontwikkelomgeving instellen {#create-dev-environment}

Om een opstelling tot stand te brengen die u toestaat te ontwikkelen voor [!DNL Asset Compute Service], volgt u deze vereisten en instructies.

1. [Toegang en referenties ophalen](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) for [!DNL Adobe Developer App Builder].

1. [De lokale omgeving instellen](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) en de vereiste gereedschappen.

1. Hier volgen nog enkele gereedschappen die u helpen probleemloos aan de slag te gaan met het ontwikkelen van:

   * [Git](https://git-scm.com/)
   * [Docker-bureaublad](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, oneven versies worden niet aanbevolen) en [NPM](https://www.npmjs.com). Gebruiker van OSX HomeBrew kan dit doen `brew install node` om beide te installeren. Anders downloadt u het bestand van het [Downloadpagina voor NodeJS](https://nodejs.org/en/)
   * Een winde die voor NodeJS goed is, adviseren wij [Visual Studio Code (VS Code)](https://code.visualstudio.com) aangezien het gesteunde winde voor debugger is. U kunt om het even welke andere winde als coderedacteur gebruiken, maar geavanceerd gebruik (b.v. debugger) wordt nog niet gesteund
   * De nieuwste[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Zorg ervoor dat u voldoet aan de [voorwaarden](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Een App Builder-project instellen {#create-App-Builder-project}

1. Zorgen voor de rol van systeembeheerder of ontwikkelaar in de [!DNL Experience Cloud] organisatie. Dit wordt ingesteld door een systeembeheerder in het [Admin Console](https://adminconsole.adobe.com/overview).

1. Aanmelden bij [Adobe Developer Console](https://console.adobe.io/). Zorg ervoor dat u deel uitmaakt van hetzelfde [!DNL Experience Cloud] organisatie als [!DNL Experience Manager] als [!DNL Cloud Service] integratie. Voor meer informatie over Adobe Developer Console raadpleegt u [Console-documentatie](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Een App Builder-project maken](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Klikken **[!UICONTROL Nieuw project maken]** > **[!UICONTROL Project uit sjabloon]**. Selecteer App Builder. Het leidt tot een nieuw Project van de Bouwer App met twee werkruimten: `Production` en `Stage`. Aanvullende werkruimten toevoegen, bijvoorbeeld `Development`, indien nodig.

1. In het Project van de Bouwer van de App, selecteer een werkruimte en teken aan de diensten nodig voor Asset compute in. Klikken **Toevoegen aan project** > **API** en toevoegen `Asset Compute`, `IO Events`, en `IO Events Management` diensten. Wanneer de eerste API wordt toegevoegd, wordt gevraagd om een persoonlijke sleutel te maken. Sla deze gegevens op uw computer op omdat u deze sleutel nodig hebt om uw aangepaste toepassing met het ontwikkelaarsgereedschap te testen.

## Volgende stap {#next-step}

Nu uw omgeving is ingesteld, kunt u [een aangepaste toepassing maken](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
