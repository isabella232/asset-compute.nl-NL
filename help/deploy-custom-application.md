---
title: Implementeer [!DNL Asset Compute Service] aangepaste toepassing.
description: Implementeer [!DNL Asset Compute Service] aangepaste toepassing.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# Een aangepaste toepassing {#deploy-custom-application} implementeren

Als u uw toepassing wilt implementeren, gebruikt u de opdracht [aio-app implementeren](https://github.com/adobe/aio-cli#aio-appdeploy). In de terminal geeft de opdracht een URL weer voor toegang tot de aangepaste toepassing. De URL heeft de notatie `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Als u dezelfde URL wilt ophalen zonder de toepassing opnieuw te implementeren, gebruikt u de opdracht [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action).

Gebruik URL in [het Profiel van de Verwerking in Experience Manager als Cloud Service ](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) om uw toepassing met [!DNL Experience Manager] als Cloud Service te integreren.

Zorg ervoor dat uw Firefly-project en -werkruimte overeenkomen met [!DNL Experience Manager] als een Cloud Service-omgeving waarin u de handeling wilt gebruiken. Het heeft verschillende omgevingen voor ontwikkeling, staging en productie. U kunt de omgeving controleren door `AIO_runtime_*` referenties te controleren die in het ENV-bestand in de hoofdmap van uw Firefly-toepassing zijn gedefinieerd. Als u bijvoorbeeld wilt implementeren in een `Stage`-werkruimte, heeft de `AIO_runtime_namespace` de notatie `xxxxxx_xxxxxxxxx_stage`. Om met [!DNL Experience Manager] als milieu van de Productie van de Cloud Service te integreren, gebruik toepassings URLs van uw Vuurwerk `Production`.

>[!CAUTION]
>
>Gebruik geen persoonlijke werkruimte op kritieke [!DNL Experience Manager] milieu&#39;s.

>[!MORELIKETHIS]
>
>* [Begrijp en beheer milieu&#39;s in Experience Manager als Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

