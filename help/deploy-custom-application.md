---
title: Aangepaste toepassing [!DNL Asset Compute Service] implementeren.
description: Aangepaste toepassing [!DNL Asset Compute Service] implementeren.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# Een aangepaste toepassing implementeren {#deploy-custom-application}

Als u uw toepassing wilt implementeren, gebruikt u de [opdracht Implementeren](https://github.com/adobe/aio-cli#aio-appdeploy) van een AIR-toepassing. In de terminal geeft de opdracht een URL weer voor toegang tot de aangepaste toepassing. De URL heeft de notatie `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Als u dezelfde URL wilt ophalen zonder de toepassing opnieuw te implementeren, gebruikt u de [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) opdracht.

Gebruik de URL in een [verwerkingsprofiel in Experience Manager als Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) om uw toepassing te integreren met [!DNL Experience Manager] als Cloud Service.

Zorg ervoor dat uw Firefly-project en -werkruimte overeenkomen met de omgeving [!DNL Experience Manager] als Cloud Service waarin u de handeling wilt gebruiken. Het heeft verschillende omgevingen voor ontwikkeling, staging en productie. U kunt de omgeving controleren door de `AIO_runtime_*` referenties te controleren die in het ENV-bestand zijn gedefinieerd in de hoofdmap van uw Firefly-toepassing. Als u bijvoorbeeld wilt implementeren in een `Stage` werkruimte, `AIO_runtime_namespace` heeft de werkruimte de indeling `xxxxxx_xxxxxxxxx_stage`. Als u wilt integreren met [!DNL Experience Manager] een productieomgeving van een Cloud Service, gebruikt u de toepassings-URL&#39;s van uw Firefly- `Production` werkruimte.

>[!CAUTION]
>
>Gebruik geen persoonlijke werkruimte in kritieke [!DNL Experience Manager] omgevingen.

>[!MORELIKETHIS]
>
>* [Begrijp en beheer milieu&#39;s in Experience Manager als Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

