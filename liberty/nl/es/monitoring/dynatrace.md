---

copyright:
  years: 2016, 2017
lastupdated: "2017-11-08"

---

{:new_window: target="_blank"}
{:codeblock: .codeblock}

#Utilización de Dynatrace para supervisar Liberty en {{site.data.keyword.cloud_notm}}
{: #using_dynatrace}

Dynatrace es un servicio de otro proveedor que proporciona supervisión de la app.

Para obtener más información acerca de lo que proporciona el servicio de Dynatrace, consulte [Supervisión de la aplicación Dynatrace](http://www.dynatrace.com/en/products/application-monitoring.html).

Cuando la aplicación Liberty se configura para utilizar Dynatrace, el comportamiento predeterminado es que el tiempo de ejecución de Liberty adquirirá un archivo jar de agente de Dynatrace desde un sitio Dynatrace y ejecutará dicho agente Dynatrace con la app.  Con este comportamiento predeterminado la configuración mínima necesaria para utilizar Dynatrace es crear un servicio proporcionado por el usuario que apunte al recopilador
Dynatrace.

## Creación de un servicio proporcionado por el usuario que apunte al recopilador de Dynatrace

En primer lugar será necesario configurar un recopilador de Dynatrace.  A continuación, deberá crear un servicio proporcionado por el usuario para pasar información para el agente
de Dynatrace para conectarse con el recopilador de Dynatrace. Consulte [Arquitectura de Dynatrace](https://community.dynatrace.com/community/display/DOCDT63/Architecture) para comprender mejor la relación entre componentes de Dynatrace.

1. Configure un recopilador de Dynatrace.
  * Consulte el [sitio web de la comunidad de Dynatrace](https://community.dynatrace.com/community/display/EVAL/Step+3+-+Connect+Agent+to+Dynatrace) para obtener instrucciones sobre la descarga y configuración del recopilador de Dynatrace.
  * Asegúrese de que el recopilador se configura en una ubicación que sea accesible para el agente de Dynatrace que se ejecuta con la app en {{site.data.keyword.Bluemix_notm}}.
2. Cree un servicio proporcionado por el usuario que apunte al recopilador de Dynatrace en ejecución. **NOTA** El nombre del servicio proporcionado por el usuario debe contener **dynatrace**. No se distingue entre mayúsculas y minúsculas. Por ejemplo, utilice el siguiente mandato, donde **my-dynatrace-collector** contiene **dynatrace**:

        $ cf cups my-dynatrace-collector -p '{"server":"DynatraceCollectorIPaddress","profile":"Monitoring"}'
        {: codeblock}

    En este ejemplo, my-dynatrace-collector es el nombre proporcionado al servicio, DynatraceCollectorIPaddress es la dirección IP del recopilador de Dynatrace que ha configurado, y el perfil es el nombre de perfil de Dynatrace opcional asociado con esta app supervisada. El valor predeterminado del perfil es Monitoring. Puede especificar los parámetros opcionales como en el ejemplo que se indica a continuación.

        $ cf cups my-dynatrace-collector -p '{"server":"DynatraceCollectorIPaddress","profile":"Monitoring",
                                              "options" : {"dynatrace-parameter-1": "value",
                                                           "dynatrace-parameter-2": "value"}}'
        {: codeblock}

    Consulte [Sección establecimiento del agente de la configuración del agente](https://community.dynatrace.com/community/display/DOCDT62/Agent+Configuration) en el sitio web de la comunidad de Dynatrace para obtener más información sobre las opciones disponibles. Por ejemplo, utilizando la opción de exclusión, puede excluir las clases que supervisa Dynatrace. Consulte [Infraestructura del agente de DynaTrace](https://github.com/cloudfoundry/ibm-websphere-liberty-buildpack/blob/master/docs/framework-dynatrace-agent.md) para obtener más detalles sobre cómo configurar el servicio proporcionado por el usuario.

3. Una vez que haya enviado la app a {{site.data.keyword.Bluemix_notm}}, enlace el servicio proporcionado por el usuario que ha creado a la app. Por ejemplo, utilice el mandato

        $ cf bs myApp my-dynatrace-collector
        {: codeblock}

    **Nota**: Debe volver a transferir la aplicación después de enlazar el servicio.

## Configuración opcional
{: #optional_configuration}

Puede elegir adquirir y alojar usted mismo el archivo jar del agente de Dynatrace.  En ese caso, son necesarios los siguientes pasos de configuración adicionales.
1. Adquiera y aloje el archivo jar del agente de Dynatrace para que el paquete de compilación de Liberty pueda descargarlo.
2. Configure la app de Liberty de modo que pueda descargar el agente de Dynatrace.

### Alojamiento del agente de Dynatrace
{: #hosting_dynatrace_agent}
El agente de Dynatrace debe estar alojado en un servidor web, y el paquete de compilación de Liberty debe ser capaz de descargar el jar del agente desde dicho servidor. El servidor debe estar configurado con un archivo `index.yml` que especifique los detalles sobre el jar del agente. Complete los pasos que se muestran a continuación para configurar el agente de Dynatrace:
  1. Descargue el jar del agente de Dynatrace. Consulte [Instaladores de la plataforma del servidor de Dynatrace](https://community.dynatrace.com/community/display/EVAL/Step+1+-+Download+and+install+Dynatrace) en el sitio web de la comunidad de Dynatrace para obtener instrucciones sobre la descarga del jar del agente de Dynatrace. El archivo jar del agente apropiado para ejecutar en {{site.data.keyword.Bluemix_notm}} es **dynatrace-agent-unix.jar** versión **6.+**.
  2. Aloje el archivo jar del agente en una ubicación desde la que el paquete de compilación de Liberty pueda descargarlo. Puede alojarlo en el mismo {{site.data.keyword.Bluemix_notm}} utilizando cualquiera de los recursos del servidor disponibles, o puede alojarlo en alguna ubicación disponible de forma pública.  
     * Asegúrese de que se proporciona un archivo `index.yml` en la ubicación de alojamiento. El archivo `index.yml` debe contener una entrada que conste del ID de versión del jar del agente seguido por dos puntos y el URL completo de la ubicación de ese jar de agente. Por ejemplo:

            ---
               6.3.0: https://my-dynatrace-agent.mybluemix.net/dynatrace-agent-6.3.0-unix.jar
            {: codeblock}

     * El archivo **dynatrace-agent-6.3.0-unix.jar** debe estar disponible en la ubicación especificada en el archivo `index.yml`. La ubicación para el archivo jar y el `index.yml` puede ser el mismo directorio.

### Configuración de la app de Liberty
{: #configuring_liberty_app}

La app de Liberty que desea supervisar deben estar configurada para localizar el servidor en el que se encuentra el jar del agente que ha establecido previamente. Puede configurar la app con la variable de entorno de **JBP_CONFIG_DYNATRACEAPPMONAGENT**. La variable de entorno de **JBP_CONFIG_DYNATRACEAPPMONAGENT** indica al paquete de compilación desde dónde se debe descargar el agente de Dynatrace. Para establecer la variable de entorno, siga estos pasos:

1. Establezca la variable **JBP_CONFIG_DYNATRACEAPPMONAGENT** para que tenga el valor
*"repository_root: URL_of_server_hosting_index.yml"*. Por ejemplo, después de enviar la aplicación, emita el siguiente mandato:

        $ cf se myApp JBP_CONFIG_DYNATRACEAPPMONAGENT 'repository_root: https://my-dynatrace-agent-host.mybluemix.net'
        {: codeblock}

    En este ejemplo, *my-dynatrace-agent-host.mybluemix.net* es el URL del archivo `index.yml` alojado por el servidor que ha configurado previamente.

2. Después de establecer la variable de entorno, vuelva a transferir la app. Consulte el registro de transferencias para ver si hay un mensaje que indica que se ha descargado correctamente el agente de Dynatrace desde el servidor de alojamiento del agente. Por ejemplo:
```
    Descarga de dynatrace-agent-6.3.0-unix.jar 6.3.0 de https://my-dynatrace-agent-host.mybluemix.net/dynatrace-agent-6.3.0-unix.jar (17,8 s)
```
{: codeblock}

# rellinks
{: #rellinks notoc}
## general
{: #general notoc}
* [Tiempo de ejecución de Liberty](index.html)
* [Visión general del perfil de Liberty](http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.wlp.nd.doc/ae/cwlp_about.html)
