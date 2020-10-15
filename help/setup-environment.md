---
title: Stel de ontwikkelomgeving in die vereist is voor [!DNL Asset Compute Service].
description: Ontwikkelomgeving instellen [!DNL Asset Compute Service] voor het maken en testen van aangepaste code.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 0%

---


# Een ontwikkelomgeving instellen {#create-dev-environment}

Om een opstelling tot stand te brengen die u toestaat om voor te ontwikkelen [!DNL Asset Compute Service], volg deze vereisten en instructies.

1. [Verkrijg toegang en geloofsbrieven](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) voor het Vuurwerk van het Project.

1. [Stel de lokale omgeving](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) en de vereiste gereedschappen in.

1. Hier volgen nog enkele gereedschappen die u helpen probleemloos aan de slag te gaan met het ontwikkelen van:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 tot v12 LTS, oneven versies worden niet aanbevolen) en [NPM](https://www.npmjs.com). Gebruiker van OSX HomeBrew kan beide toepassingen installeren `brew install node` . Als dat niet het geval is, downloadt u deze van de [downloadpagina](https://nodejs.org/en/)NodeJS.
   * winde die voor NodeJS goed is, adviseren wij de Code van [Visual Studio (de Code van VS)](https://code.visualstudio.com) aangezien het gesteunde winde voor debugger is. U kunt om het even welke andere winde als coderedacteur gebruiken, maar geavanceerd gebruik (b.v. debugger) wordt nog niet gesteund.
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`) - installeren met `npm install -g @adobe/aio-cli`.

1. Zorg ervoor dat u aan de [voorwaarden](/help/understand-extensibility.md#prerequisites-and-provisioning)voldoet.

## Een Firefly-project instellen {#create-firefly-project}

1. Systeembeheerder of ontwikkelaarsrol toegang krijgen tot de Ervaring Organisatie. Dit kan door een Admin van het Systeem in de [Admin Console](https://adminconsole.adobe.com/overview)worden geplaatst.

1. Meld u aan bij de [Adobe Developer Console](https://console.adobe.io/). Zorg ervoor dat u deel uitmaakt van dezelfde Adobe Experience Cloud-organisatie als de AEM als integratie van Cloud Servicen. Raadpleeg de documentatie bij [Console voor meer informatie over Adobe Developer Console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Maak een Firefly-project](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Klik op **[!UICONTROL Nieuw project]** maken > **[!UICONTROL Project maken van sjabloon]**. Selecteer Firefly. Het leidt tot een nieuw Vuurwerk Project met twee werkruimten: `Production` en `Stage`. Voeg bijvoorbeeld desgewenst extra werkruimten toe `Development`.

1. Selecteer in het project Firefly een werkruimte en abonneer op de services die nodig zijn voor Asset Compute. Klik op **Toevoegen aan project** > **API** en voeg `Asset Compute`, `IO Events`en `IO Events Management` services toe. Wanneer de eerste API wordt toegevoegd, wordt gevraagd om een persoonlijke sleutel te maken. Sla deze gegevens op uw computer op omdat u deze sleutel nodig hebt om uw aangepaste toepassing met het ontwikkelaarsgereedschap te testen.

## Volgende stap {#next-step}

Nu de omgeving is ingesteld, kunt u een aangepaste toepassing [](develop-custom-application.md)maken.

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
