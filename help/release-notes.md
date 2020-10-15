---
title: Release-aantekeningen van [!DNL Asset Compute Service].
description: Nieuwe functies, verbeteringen en bekende problemen in [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 0%

---


# Opmerkingen bij de release van [!DNL Asset Compute Service] {#release-notes}

De meest recente release van [!DNL Asset Compute Service] is uitgebracht op 30 juli 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Nieuwe functies {#what-is-new}

Dit is de eerste release van [!DNL Asset Compute Service]. Het is een schaalbare en uitbreidbare service voor het verwerken van digitale elementen [!DNL Adobe Experience Cloud] . Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Momenteel [!DNL Asset Compute Service] kan het alleen worden gebruikt in [!DNL Experience Manager] de vorm van een Cloud Service.

## Beperkingen en bekende problemen {#known-limitations}

Als u uw aangepaste toepassing wilt testen met het [ontwikkelprogramma](https://github.com/adobe/asset-compute-devtool), hebt u toegang nodig tot een [cloudopslagcontainer](https://github.com/adobe/asset-compute-devtool#prerequisites).

* De toegang tot cloudopslag (anders dan de [!DNL Experience Manager] blob-opslag) is alleen nodig voor de ontwikkelaar. U kunt nog steeds aangepaste toepassingen maken, testen en implementeren zonder het ontwikkelaarsgereedschap.
* Het kan een gedeelde container zijn die door veelvoudige ontwikkelaars over verschillende projecten wordt gebruikt.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] rekbaarheid wordt ontwikkeld in het kader van een open ontwikkelingsmodel op [github.com/adobe](https://github.com/adobe) , dat bijdragen van uitbreidingsontwikkelaars toejuicht . Alle componenten die relevant zijn voor het ontwikkelen, maken, testen en implementeren van aangepaste toepassingen zijn open source. Zie [hoe en waar om aan de Rekendienst](contribute-to-compute-service.md)bij te dragen.

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
