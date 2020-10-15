---
title: Architectuur van [!DNL Asset Compute Service].
description: De [!DNL Asset Compute Service] HoeAPI, toepassingen, en SDK werken samen om de cloud-native dienst voor middelenverwerking te verlenen.
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# Architectuur van [!DNL Asset Compute Service] {#overview}

Het [!DNL Asset Compute Service] is gebouwd op het serverloze Adobe I/O Runtime-platform. Het biedt ondersteuning voor Adobe Sensei-inhoudsservices voor elementen. De aanroepende client (alleen [!DNL Experience Manager] als een Cloud Service wordt ondersteund) wordt voorzien van de door Adobe Sensei gegenereerde informatie die hij voor het element heeft aangevraagd. De geretourneerde informatie heeft de JSON-indeling.

[!DNL Asset Compute Service] is uitbreidbaar door aangepaste toepassingen te maken op basis van [!DNL Project Firefly]. Deze aangepaste toepassingen zijn toepassingen zonder [!DNL Project Firefly] kop en voeren bijvoorbeeld aangepaste conversieprogramma&#39;s toe of roepen externe API&#39;s op om afbeeldingsbewerkingen uit te voeren.

[!DNL Project Firefly] is een framework om aangepaste webtoepassingen op [!DNL Adobe I/O] runtime te maken en te implementeren. Om douanetoepassingen tot stand te brengen, kunnen de ontwikkelaars hefboomwerking [!DNL React Spectrum] (toolkit UI van Adobe), microdiensten tot stand brengen, douanegebeurtenissen tot stand brengen, en APIs orchestreren. Zie [documentatie van Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

De basis waarop de architectuur is gebaseerd, is onder meer:

* De modulariteit van toepassingen - die slechts wat voor een bepaalde taak vereist bevat - staat toe om toepassingen van elkaar te loskoppelen en hen lichtgewicht te houden.

* Het serverless concept van Adobe I/O Runtime biedt tal van voordelen: asynchrone, hoogst scalable, geïsoleerde, op baan-gebaseerde verwerking, die perfect voor activaverwerking past.

* Binaire cloudopslag biedt de benodigde functies voor het afzonderlijk opslaan en openen van elementbestanden en uitvoeringen, zonder dat volledige toegangsrechten voor de opslag vereist zijn, met behulp van vooraf ondertekende URL-verwijzingen. Dankzij overdrachtsversnelling, CDN-caching en colocatie van computertoepassingen met cloudopslag hebt u optimale toegang tot content met lage latentie. Zowel AWS- als Azure-wolken worden ondersteund.

![Architectuur van Asset Compute Service](assets/architecture-diagram.png)

*Afbeelding: Architectuur van [!DNL Asset Compute Service] en hoe het met [!DNL Experience Manager], opslag, en verwerkingstoepassing integreert.*

De architectuur bestaat uit de volgende onderdelen:

* **Een API- en organisatielaag** ontvangt aanvragen (in JSON-indeling) die de service de opdracht geven een bronelement om te zetten in meerdere uitvoeringen. Deze aanvragen zijn asynchroon en worden geretourneerd met een activerings-id (ook wel &#39;job id&#39; genoemd). Instructies zijn louter declaratief en voor alle standaardverwerkingswerkzaamheden (bijv. miniatuurproductie, tekstextractie) geven consumenten alleen het gewenste resultaat aan, maar niet de toepassingen die bepaalde uitvoeringen verwerken. Algemene API-functies zoals verificatie, analyse en snelheidsbeperking worden vóór de service afgehandeld met de Adobe API-gateway en beheren alle aanvragen die naar I/O-runtime gaan. De toepassing die verplettert wordt dynamisch gedaan door de orchestratielaag. Aangepaste toepassingen kunnen door clients worden opgegeven voor specifieke uitvoeringen en aangepaste parameters bevatten. De uitvoering van de toepassing kan volledig parallel lopen aangezien zij afzonderlijke serverfuncties in I/O Runtime zijn.

* **Toepassingen om elementen** te verwerken die zich specialiseren op bepaalde bestandsindelingen of doeluitvoeringen. Conceptueel, is een toepassing als het Unix pijpconcept: een invoerbestand wordt omgezet in een of meer uitvoerbestanden.

* **Een [algemene toepassingsbibliotheek](https://github.com/adobe/asset-compute-sdk)** verwerkt algemene taken zoals het downloaden van het bronbestand, het uploaden van de uitvoeringen, het rapporteren van fouten, het verzenden en controleren van gebeurtenissen. Dit is zo ontworpen dat het ontwikkelen van een toepassing zo eenvoudig mogelijk blijft, na het serverloze idee, en kan worden beperkt tot lokale bestandssysteeminteracties.

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->