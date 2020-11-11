---
title: Inleiding tot het [!DNL Asset Compute Service].
description: '[!DNL Asset Compute Service] is een service voor de verwerking van eigen middelen in de cloud die de complexiteit vermindert en de schaalbaarheid verbetert.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '321'
ht-degree: 0%

---


# Overzicht van [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] is een schaalbare en uitbreidbare service voor het verwerken van digitale elementen [!DNL Adobe Experience Cloud] . Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Ontwikkelaars kunnen aangepaste middelentoepassingen (ook wel aangepaste workers genoemd) insluiten om aangepaste gebruiksgevallen aan te pakken. De service werkt op de [!DNL Adobe I/O] runtime. De functie kan worden uitgebreid met toepassingen zonder [!DNL Project Firefly] koppen die zijn geschreven in Node.js. Deze kunnen aangepaste bewerkingen uitvoeren, zoals het aanroepen van externe API&#39;s om afbeeldingsbewerkingen uit te voeren of [!DNL Adobe Sensei] ondersteuning te bieden.

[!DNL Project Firefly] is een raamwerk voor het ontwikkelen en implementeren van aangepaste webtoepassingen op [!DNL Adobe I/O] runtime voor het uitbreiden van Adobe Experience Cloud-oplossingen. Om douanetoepassingen tot stand te brengen, kunnen de ontwikkelaars hefboomwerking [!DNL React Spectrum] (toolkit UI van Adobe), microdiensten tot stand brengen, douanegebeurtenissen tot stand brengen, en APIs orchestreren. Zie [documentatie van Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Momenteel [!DNL Asset Compute Service] kan het alleen via [!DNL Experience Manager] een Cloud Service worden gebruikt. Beheerders maken verwerkingsprofielen die de profielen kunnen aanroepen [!DNL Asset Compute Service] om elementen door te geven voor verwerking. Zie [Elementmicroservices en verwerkingsprofielen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)gebruiken.

## Ondersteund gebruik van [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] ondersteunt een aantal gangbare gevallen voor zakelijk gebruik, zoals eenvoudige beeldverwerking; specifieke omzettingen van Adobe-toepassingen; en de verwezenlijking van douanetoepassingen die complexe bedrijfsvereisten ordenen.

U kunt de [!DNL Asset Compute] webservice gebruiken om miniaturen te genereren voor verschillende bestandstypen, kwalitatief hoogstaande afbeeldingsrenderingen voor de [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Zie [gebruiksgevallen die worden ondersteund via aangepaste configuratie](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>De service biedt geen opslag van bedrijfsmiddelen. Gebruikers geven deze informatie en geven verwijzingen naar de locatie van bron- en uitvoerbestanden in de cloudopslag.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use Adobe I/O Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Overzicht van asset processing met asset microservices in Adobe Experience Manager als Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentatie van het project zonder problemen](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Ondersteunde bestandsindelingen voor verwerking](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).
>* [Opmerkingen bij de release van de Asset compute Service](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
