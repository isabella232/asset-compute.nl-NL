---
title: Release-aantekeningen van [!DNL Asset Compute Service].
description: Nieuwe functies, verbeteringen en bekende problemen in  [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: c57867cd896e4ccb9402e6eeb0eea133faaa0e5d
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---


# Opmerkingen bij de release van [!DNL Asset Compute Service] {#release-notes}

De meest recente release van [!DNL Asset Compute Service] wordt uitgebracht op 30 juli 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Nieuw {#what-is-new}

Dit is de eerste release van [!DNL Asset Compute Service]. Het is een schaalbare en uitbreidbare service van [!DNL Adobe Experience Cloud] voor het verwerken van digitale elementen. Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Momenteel kan [!DNL Asset Compute Service] alleen in [!DNL Experience Manager] als [!DNL Cloud Service] worden gebruikt.

## Beperkingen en bekende problemen {#known-limitations}

Als u uw aangepaste toepassing wilt testen met het [hulpprogramma voor ontwikkelaars](https://github.com/adobe/asset-compute-devtool), hebt u toegang nodig tot een [cloudopslagcontainer](https://github.com/adobe/asset-compute-devtool#prerequisites).

* De toegang tot cloudopslag (anders dan de [!DNL Experience Manager] blob store) is alleen nodig voor het ontwikkelaarsprogramma. U kunt nog steeds aangepaste toepassingen maken, testen en implementeren zonder het ontwikkelaarsgereedschap.
* Het kan een gedeelde container zijn die door veelvoudige ontwikkelaars over verschillende projecten wordt gebruikt.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] de uitbreidbaarheid wordt ontwikkeld in het kader van een open ontwikkelingsmodel op  [github.com/](https://github.com/adobe) adobeb , dat bijdragen van ontwikkelaars van extensies toejuicht . Alle componenten die relevant zijn voor het ontwikkelen, maken, testen en implementeren van aangepaste toepassingen zijn open source. Zie [Hoe en waar u wilt bijdragen aan de Compute Service](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
