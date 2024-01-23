# Guía de Instalación del Toolkit de Prevención de VIH en DHIS2 para OPS { #hiv-prev-paho-installation }

Versión del Paquete 1.0.0

Idioma predeterminado del sistema: Inglés, Español

## Instalación

La instalación del módulo consta de varios pasos:

1. [Preparación](#preparando-el-archivo-de-metadatos) del archivo de metadatos.
2. [Importación](#importando-metadatos) del archivo de metadatos en DHIS2.
3. [Configuración](#configuración) de los metadatos importados.
4. [Adaptación](#adaptar-el-programa) del programa después de la importación.

Se recomienda leer primero cada sección de la guía de instalación antes de comenzar el proceso de instalación y configuración en DHIS2. Identifique las secciones aplicables según el tipo de importación:

1. Importar en una instancia en blanco de DHIS2.
2. Importar en una instancia de DHIS2 con metadatos existentes (sin importación previa de otras versiones del rastreador de prevención del VIH).
3. Actualizar la versión existente/anterior del rastreador de prevención del VIH.

Las medidas descritas en este documento deben probarse en una instancia de DHIS2 de prueba antes de aplicarse en un entorno de producción.

## Requisitos

Para instalar el módulo, se requiere una cuenta de usuario administrador en DHIS2. El procedimiento detallado en este documento debe probarse en un entorno de prueba antes de realizarse en una instancia de producción de DHIS2.

Se debe tener mucho cuidado para asegurarse de que el servidor y la aplicación DHIS2 estén bien seguros para restringir el acceso a los datos que se recopilan. Los detalles sobre cómo asegurar un sistema DHIS2 están fuera del alcance de este documento, y remitimos a la [documentación de DHIS2](http://dhis2.org/documentation).

## Preparando el archivo de metadatos

**NOTA:** Si está instalando el paquete en una nueva instancia de DHIS2, puede omitir la sección "Preparación del archivo de metadatos" y pasar directamente a la sección [Importar un archivo de metadatos en DHIS2](#importando-metadatos)

Aunque no siempre es necesario, a menudo es ventajoso hacer ciertas modificaciones al archivo de metadatos antes de importarlo a DHIS2.

### Dimensión de datos predeterminada (si es aplicable)

En las primeras versiones de DHIS2, la UID de la dimensión de datos predeterminada se generaba automáticamente. Por lo tanto, aunque todas las instancias de DHIS2 tienen una opción de categoría predeterminada, una categoría de elemento de datos, una combinación de categorías y una combinación de opciones de categorías predeterminadas, las UID de estos valores predeterminados pueden ser diferentes. Las versiones posteriores de DHIS2 tienen UID codificadas para la dimensión predeterminada, y estas UID se utilizan en los paquetes de configuración.

Para evitar conflictos al importar los metadatos, se recomienda buscar y reemplazar el archivo .json completo en todas las instancias de estos objetos predeterminados, reemplazando las UID del archivo .json con las UID de la base de datos en la que se importará el archivo. La Tabla 1 muestra las UID que deben reemplazarse, así como los puntos finales de la API para identificar las UID existentes.

| Objeto                               | UID         | API endpoint                                              |
|:-------------------------------------|:------------|:----------------------------------------------------------|
| Categoría                            | GLevLNI9wkl | `../api/categories.json?filter=name:eq:default`           |
| Opción de Categoría                  | xYerKDKCefk | `../api/categoryOptions.json?filter=name:eq:default`      |
| Combinación de Categorías            | bjDvmb4bfuf | `../api/categoryCombos.json?filter=name:eq:default`       |
| Combinación de Opciones de Categoría | HllvX50cXC0 | `../api/categoryOptionCombos.json?filter=name:eq:default` |

Por ejemplo, si está importando un paquete de configuración en <https://play.dhis2.org/demo>, la UID de la combinación de opciones de categoría predeterminada podría identificarse a través de <https://play.dhis2.org/demo/api/categoryOptionCombos.json?filter=name:eq:default> como bRowv6yZOF2.

Luego podría buscar y reemplazar todas las instancias de HllvX50cXC0 con bRowv6yZOF2 en el archivo .json, ya que es la ID predeterminada en el sistema que está importando. **_Tenga en cuenta que esta operación de búsqueda y reemplazo debe realizarse con un editor de texto plano_**, no con un procesador de texto como Microsoft Word.

### Tipos de indicadores

El tipo de indicador es otro tipo de objeto que puede generar conflictos durante la importación porque se utilizan ciertos nombres en diferentes bases de datos de DHIS2 (por ejemplo, "Porcentaje"). Dado que los tipos de indicadores se definen simplemente por su factor y si son números simples sin denominador, son inequívocos y se pueden reemplazar mediante una búsqueda y reemplazo de las UID. Esto evita conflictos de importación potenciales y evita la creación de tipos de indicadores duplicados. La siguiente tabla muestra las UID que podrían reemplazarse, así como los puntos finales de la API para identificar las UID existentes.

| Objeto                  | UID         | API endpoint                                                               |
|:------------------------|:------------|:---------------------------------------------------------------------------|
| Porcentaje              | hmSnCXmLYwt | `../api/indicatorTypes.json?filter=number:eq:false&filter=factor:eq:100`   |
| Tasa (factor=1)         | k4RGC3sMTzO | `../api/indicatorTypes.json?filter=number:eq:false&filter=factor:eq:1`     |
| Por 10 000              | FWTvArgP0jt | `../api/indicatorTypes.json?filter=number:eq:false&filter=factor:eq:10000` |
| Solo numerador (número) | kHy61PbChXr | `..api/indicatorTypes.json?filter=number:eq:true`                          |

### Tipo de entidad rastreada

Al igual que los tipos de indicadores, es posible que ya tenga tipos de entidad rastreada existentes en su base de datos de DHIS2. Las referencias al tipo de entidad rastreada deben cambiarse para reflejar lo que hay en su sistema para no crear duplicados. La Tabla 3 muestra las UID que podrían reemplazarse, así como los puntos finales de la API para identificar las UID existentes.

| Objeto  | UID         | API endpoint                                           |
|:--------|:------------|:-------------------------------------------------------|
| Persona | MCPQUTHX1Ze | `../api/trackedEntityTypes.json?filter=name:eq:Person` |

### Visualizaciones que utilizan la UID de la Unidad Organizativa Raíz

Las visualizaciones, informes de eventos, tablas de informes y mapas que se asignan a un nivel específico de unidad organizativa o grupo de unidades organizativas tienen una referencia a la unidad organizativa raíz (nivel 1). Dichos objetos, si están presentes en el archivo de metadatos, contienen un marcador de posición `<OU_ROOT_UID>`. Utilice la función de búsqueda en el editor de archivos .json para identificar posiblemente este marcador de posición y reemplácelo con la UID de la unidad organizativa de nivel 1 en la instancia de destino.

Algunas visualizaciones y mapas pueden contener referencias a niveles de unidades organizativas. Los mapas que constan de varias vistas de mapas pueden contener diversas referencias a niveles de unidades organizativas según la configuración de la capa de mapas. Ajuste las referencias a los niveles de unidades organizativas en el archivo json de metadatos para que coincidan con la estructura de unidades organizativas en la instancia de destino antes de importar el archivo de metadatos.

### Actualización del paquete de metadatos

El proceso de actualizar un paquete existente a una versión más reciente en una instancia de DHIS2 operativa es una operación compleja que debe realizarse con precaución. Este proceso debe ejecutarse primero en una instancia de prueba antes de actualizar la configuración en el servidor de producción. Dado que los objetos de metadatos pueden haberse eliminado, agregado o modificado, es importante asegurarse de que:

   - el formato de los datos existentes se pueda mapear y ajustar a la nueva configuración;
   - los objetos de metadatos descontinuados se eliminen de la instancia;
   - los objetos existentes se actualicen;
   - los nuevos objetos se creen;
   - se revise la asignación de usuarios a grupos de usuarios relevantes.

## Importando metadatos

El archivo .json de metadatos se encuentra en 2 idiomas de base: Inglés y Español. Los mismos pueden encontrarse en la carpeta [config](../config) del directorio raíz.

Cada archivo contiene su idioma de base identificado en el nombre, por ejemplo "xxxxxx-**es**.json", y su traducción al otro idioma de base, en este caso, al Inglés.

> NOTA: Para una completa traducción del sistema y programa al idioma de base preferido, por ejemplo Español, se recomienda importar el arhcivo "xxxxxx-**es**.json"


El archivo .json de metadatos se importa a través de la aplicación [Importar/Exportar](https://docs.dhis2.org/master/en/user/html/import_export.html) de DHIS2. Se recomienda utilizar la función "Start dry run" para identificar problemas antes de intentar realizar una importación real de los metadatos. Si la importación en "dry run" informa problemas o conflictos, consulte la [sección de conflictos de importación](#manejo-de-conflictos-de-importación) a continuación.

Si la importación en dry run/validación funciona sin errores, intente importar los metadatos. Si la importación tiene éxito sin errores, puede proceder a [configurar](#configuración) el módulo. En algunos casos, los conflictos de importación o problemas no se muestran durante la importación en "dry run", sino que aparecen cuando se intenta la importación real. En este caso, el resumen de importación enumerará cualquier error que deba resolverse.

### Manejo de conflictos de importación

> NOTA: Si está importando en una nueva instancia de DHIS2, no tendrá que preocuparse por conflictos de importación, ya que no hay nada en la base de datos a la que está importando que pueda entrar en conflicto. Siga las instrucciones para importar los metadatos y luego continúe con la sección "[Configuración](#configuración)".

Existen varios tipos de conflictos que pueden ocurrir, aunque el más común es que hay objetos de metadatos en el paquete de configuración con un nombre, un nombre corto y/o un código que ya existe en la base de datos de destino. Hay un par de soluciones alternativas para estos problemas, cada una con diferentes ventajas y desventajas. Cuál es más apropiada dependerá, por ejemplo, del tipo de objeto para el cual ocurre un conflicto.

#### Alternativa 1

Cambie el nombre del objeto existente en su base de datos DHIS2 para el cual hay un conflicto. La ventaja de este enfoque es que no es necesario modificar el archivo .json, ya que los cambios se realizan a través de la interfaz de usuario de DHIS2. Es probable que sea menos propenso a errores. También significa que el paquete de configuración se deja como está, lo que puede ser una ventaja, por ejemplo, cuando se utilizarán materiales de capacitación y documentación basados en el paquete de configuración.

#### Alternativa 2

Cambie el nombre del objeto para el cual hay un conflicto en el archivo .json. La ventaja de este enfoque es que los metadatos existentes de DHIS2 se mantienen como están. Esto puede ser un factor cuando hay material de capacitación o documentación, como SOP o diccionarios de datos, vinculado al objeto en cuestión, y no implica ningún riesgo de confundir a los usuarios al modificar los metadatos con los que están familiarizados.

Tenga en cuenta que tanto para la alternativa 1 como para la 2, la modificación puede ser tan simple como agregar un pequeño prefijo/sufijo al nombre, para minimizar el riesgo de confusión.

#### Alternativa 3

Un tercer enfoque más complicado es modificar el archivo .json para reutilizar los metadatos existentes. Por ejemplo, en casos donde ya existe un conjunto de opciones para un cierto concepto (por ejemplo, "sexo"), ese conjunto de opciones podría eliminarse del archivo .json y todas las referencias a su UID reemplazarse por el conjunto de opciones correspondiente ya en la base de datos. La gran ventaja de esto (que no se limita a los casos donde hay un conflicto directo de importación) es evitar la creación de metadatos duplicados en la base de datos. Hay algunas consideraciones clave que hacer al realizar este tipo de modificación:

* requiere conocimientos expertos de la estructura detallada de metadatos de DHIS2
* el enfoque no funciona para todos los tipos de objetos. En particular, ciertos tipos de objetos tienen dependencias que son complicadas de resolver de esta manera, por ejemplo, relacionadas con desgloses.
* las actualizaciones futuras del paquete de configuración serán complicadas.

## Configuración

Una vez que se haya importado exitosamente toda la metainformación, hay algunos pasos que deben realizarse antes de que el módulo sea funcional.

### Compartir

En primer lugar, deberás utilizar la funcionalidad de _Compartir_ de DHIS2 para configurar qué usuarios (grupos de usuarios) deben ver la metainformación y los datos asociados al programa, así como quiénes pueden registrar/ingresar datos en el programa. Por defecto, el compartir se ha configurado para lo siguiente:

* Tipo de entidad
* Programa
* Etapas del programa
* Tableros

Un paquete de metadatos generalmente contiene varios grupos de usuarios:

* HIVp PAHO Acceso
* HIVp PAHO Administrador
* HIVp PAHO Captura de datos

Por defecto, lo siguiente está asignado a estos grupos de usuarios

| Objeto              | Grupos de Usuarios                         | Grupos de Usuarios                                  | Grupos de Usuarios                                    |
|---------------------|--------------------------------------------|-----------------------------------------------------|-------------------------------------------------------|
|                     | HIVp PAHO Acceso                           | HIVp PAHO Administrador                             | Captura de Datos HIVp PAHO                            |
| Tipo de entidad     | Metadatos: puede ver <br> Datos: puede ver | Metadatos: puede editar y ver <br> Datos: puede ver | Metadatos: puede ver <br> Datos: puede capturar y ver |
| Programa            | Metadatos: puede ver <br> Datos: puede ver | Metadatos: puede editar y ver <br> Datos: puede ver | Metadatos: puede ver <br> Datos: puede capturar y ver |
| Etapas del programa | Metadatos: puede ver <br> Datos: puede ver | Metadatos: puede editar y ver <br> Datos: puede ver | Metadatos: puede ver <br> Datos: puede capturar y ver |
| Tableros            | Metadatos: puede ver <br> Datos: puede ver | Metadatos: puede editar y ver <br> Datos: puede ver | Metadatos: puede ver <br> Datos: puede ver            |

Deberás asignar tus usuarios al grupo de usuarios apropiado según su rol dentro del sistema. Puede que desees habilitar el compartir para otros objetos en el paquete dependiendo de tu configuración. Consulta la [Documentación de DHIS2](https://docs.dhis2.org/master/en/dhis2_user_manual_en/about-sharing-of-objects.html) para obtener más información sobre la configuración de compartir.

### Roles de Usuario

Los usuarios necesitarán roles de usuario para interactuar con las diversas aplicaciones dentro de DHIS2. Se recomiendan los siguientes roles mínimos:

1. Análisis de datos de rastreo: Puede ver análisis de eventos y acceder a tableros, informes de eventos, visualizador de eventos, visualizador de datos, tablas dinámicas, informes y mapas de eventos.
2. Captura de datos de rastreo: Puede agregar valores de datos, actualizar entidades rastreadas, buscar entidades rastreadas en unidades organizativas y acceder a la captura de rastreo.

Consulta la [Documentación de DHIS2](http://dhis2.org/documentation) para obtener más información sobre la configuración de roles de usuario.

### Unidades Organizativas

Debes asignar el programa a las unidades organizativas dentro de tu propia jerarquía para poder ver el programa en la captura de rastreo.

### Metadatos Duplicados

> **NOTA**
>
> Esta sección solo se aplica si estás importando en una base de datos DHIS2 en la que ya hay metadatos presentes. Si estás trabajando con una nueva instancia de DHIS2, por favor omite esta sección y ve a [Adaptar el programa de rastreo](#adaptar-el-programa).
> Si estás utilizando aplicaciones de terceros que dependen de los metadatos actuales, ten en cuenta que esta actualización podría afectarlas.

Incluso cuando la metainformación se ha importado con éxito sin conflictos de importación, puede haber duplicados en la metainformación: elementos de datos, atributos de entidad rastreada o conjuntos de opciones que ya existen. Como se señaló en la sección anterior sobre la resolución de conflictos, un problema importante a tener en cuenta es que las decisiones sobre realizar cambios en la metainformación en DHIS2 también deben tener en cuenta otros documentos y recursos que están de diferentes maneras asociados tanto con la metainformación existente como con la metainformación que se ha importado a través del paquete de configuración. La resolución de duplicados no es solo una cuestión de "limpiar la base de datos", sino también asegurarse de que esto se haga sin romper posibles integraciones con otros sistemas, la posibilidad de utilizar material de capacitación, romper procedimientos operativos estándar, etc. Esto dependerá en gran medida del contexto.

### [CONFIG] Metadatos

Los siguientes metadatos deben configurarse después de la importación.

| Tipo de Metadato | Nombre                                                    |
|------------------|-----------------------------------------------------------|
| Option Sets      | [CONFIG]_HIVp_PAHO - PrEP reason discontinuation          |
| Option Sets      | [CONFIG]_HIVp_PAHO - PrEP adverse effect                  |
| Option Sets      | [CONFIG]_HIVp_PAHO - STI: syndromic treatment             |
| Option Sets      | [CONFIG]_HIVp_PAHO - STI: Syphilis treatment              |
| Option Sets      | [CONFIG]_HIVp_PAHO - STI: Neisseria gonorrhoeae treatment |
| Option Sets      | [CONFIG]_HIVp_PAHO - STI: Chlamydia treatment             |
| Option Sets      | [CONFIG]_HIVp_PAHO - STI: Monkeypox treatment             |

**Ten en cuenta que esto podría afectar a los Indicadores del Programa y las Reglas del Programa.**

## Adaptar el Programa

Una vez que el programa haya sido importado, es posible que desees realizar ciertas modificaciones al programa. Ejemplos de adaptaciones locales que _podrían_ realizarse incluyen:

* Agregar variables adicionales al formulario.
* Adaptar nombres de elementos de datos/opciones según las convenciones nacionales.
* Agregar traducciones a variables y/o al formulario de entrada de datos.
* Modificar indicadores de programa basados en definiciones de casos locales.

Sin embargo, se recomienda encarecidamente tener mucho cuidado si decides cambiar o eliminar alguno de los metadatos incluidos. Existe el peligro de que las modificaciones puedan afectar la funcionalidad, por ejemplo, las reglas del programa e indicadores del programa.