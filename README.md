# Processa.Utilities.Logging

Facilita el registro de información de diagnóstico (*logging*) para una aplicación. Expone métodos para generar tres tipos de información:

- **Debug**: Información de depuración. Básicamente se trata de mensajes de texto adicional a la fecha y hora en que se escribió el mensaje. Los archivos se rotan cada dos días.
 
- **Error**: Información de excepciones. Guarda el registro de excepciones (mensaje, tipo, pila, etc) generadas por la aplicación. Los archivos se rotan cada 31 días. Ademas de guardar la información en un archivo, deja una copia de la misma en el log de eventos del sistema operativo y si se establece un valor para la propiedad `SlackWebhookUri`, también envía un mensaje al Webhook configurado.
 
- **Info**: Información de seguimiento. Guarda un objeto en formato `JSON`. Si desea, puede excluir algunas propiedades del objeto antes de guardar. Los archivos se rotan cada dos días.

![Preview](image_preview.png)

## Instalación
El paquete `Processa.Utilities.Logging`, está disponible a través de Nuget o Proget

```powershell
Install-Package Processa.Utilities.Logging
```

## Versiones soportadas de .NET Framework:
- NET 4.6.0 o superior

## Conceptos básicos de configuración

`LogManager` utiliza un API de configuración simple a través de la clase `LoggerSettings`. El constructor predeterminado de `LoggerSettings` implementa una configuración básica que en muchos casos no tendrá que cambiar.

La configuración predeterminada establece una subcarpeta Logs en la misma carpeta del archivo binario (generalmente `bin`), reteniendo los últimos dos archivos `Debug` e `Info` y los últimos 31 archivos `Error`.

Si desea personalizar la configuración, basta con cambiar el valor de algunas de sus propiedades. Por ejemplo el siguiente código establece la carpeta donde se generan los archivos de logs a `C:\Temp`.

```c#
LoggerSettings settings = new LoggerSettings();
settings.LogFolderPath = @"C:\Temp";
var logManager = new LogManager(settings);
```
## Archivo de configuración
Logging permite la lectura de los datos de configuración desde un archivo de configuración con el nombre: `Processa.Utilities.Logging.dll.config` en el mismo directorio de los ensamblados de su aplicación.

Configuración predeterminada:
```xml
<configuration>
  <appSettings>
    <clear/>
    <add key="LogFolderPath"/>
    <add key="LogFolderName" value="Logs"/>
    <add key="PrefixFileName"/>
    <add key="DebugTemplate" value="{Username} - {Level} - {Timestamp:yyyy-MM-dd HH:mm:ss.fff} - {Message}{NewLine}"/>
    <add key="DebugFileName" value="Debug.txt"/>
    <add key="DebugRetainedFile" value="2"/>
    <add key="ErrorFileName" value="Error.txt"/>
    <add key="ErrorRetainedFile" value="32"/>
    <add key="ErrorTemplate" value="Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}Message: {Message}{NewLine}Exception: {Exception}Exception Data: {NewLine}{ExceptionData}{NewLine}{Separator}{NewLine}"/>
    <add key="InfoFileName" value="Info.txt"/>
    <add key="InfoRetainedFile" value="2"/>
    <add key="InfoTemplate" value="Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}{Message}{NewLine}{Separator}{NewLine}"/>
    <add key="EventLogName" value="Processa"/>
    <add key="EventLogSourceName"/>
    <add key="EventLogTemplate" value="Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}Message: {Message}{NewLine}Username: {Username}{NewLine}Exception: {Exception}Exception Data: {ExceptionData}"/>
    <add key="SlackWebhookUri"/>
    <add key="RollingInterval" value="Day"/>
    <add key="FlushToDiskInterval" value="60"/>
    <add key="SeqServerUri"/>
    <add key="SeqApiKey"/>
  </appSettings>
  <connectionStrings>
    <clear/>
  </connectionStrings>
</configuration>
``` 

### Parámetros de configuración:

| Parámetro | Descripción | Valor predeterminado |
| --------- | ----------- | -------------------- |
| LogFolderPath | Ruta de acceso de la carpeta raíz donde se crearán los archivos de logs. | Cuando no se específica un valor, se usará la raíz del ensamblado: `Processa.Utilities.Logging.dll` |
| LogFolderName | Nombre de la carpeta donde se crearán los archivos de logs. | `Logs` |
| PrefixFileName | Prefijo que se utilizará para el nombre de los archivos de logs. | Cuando no se específica un valor, se usará el nombre del emsamblado que hace uso de la funcionalidad. |
| DebugTemplate | Plantilla que se utilizará para escribir y dar formato a la información en el archivo de **Debug**. | {Username} - {Timestamp:yyyy-MM-dd HH:mm:ss.fff} - {Message}{NewLine} |
| DebugRetainedFile | Número de archivos que serán persistidos por defecto para la información de **Debug**. | `2` |
| DebugFileName | Nombre para el archivo de logs donde se escribirá la información de **Debug**. | `Debug.txt` |
| ErrorTemplate | Plantilla que se utilizará para escribir y dar formato a la información en el archivo de **Error**. | Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}Message: {Message}{NewLine}Exception: {Exception}Exception Data: {ExceptionData}{NewLine}{Separator}{NewLine} |
| ErrorRetainedFile | Número de archivos que serán persistidos por defecto para la información de **Error**. | `32` |
| ErrorFileName | Nombre para el archivo de logs donde se escribirá la información de **Error**. | `Error.txt` |
| InfoTemplate | Plantilla que se utilizará para escribir y dar formato a la información en el archivo de **Info**. | Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}{Message}{NewLine}{Separator}{NewLine} |
| InfoRetainedFile | Número de archivos que serán persistidos por defecto para la información de **Info**. | `2` |
| InfoFileName | Nombre para el archivo de logs donde se escribirá la información de **Info**. | `Info.txt` |
| EventLogName | Nombre del registro de eventos del sistema operativo donde se escribirán las entradas de seguimiento para la aplicación. | `Processa` |
| EventLogSourceName | Nombre que se utilizará como identificador de origen en el log de eventos del sistema operativo. | Cuando no se específica un valor, se usará el nombre del emsamblado que hace uso de la funcionalidad o en su defecto: `Anonymous`. |
| EventLogTemplate | Plantilla que se utilizará para escribir y dar formato a la información en el log de eventos del sistema operativo. | Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}Message: {Message}{NewLine}Username: {Username}{NewLine}Application Name: {ApplicationName}{NewLine}Exception: {Exception}Exception Data: {ExceptionData} |
| RollingInterval | Especifica la frecuencia en la que se crearán nuevos archivos de registro para la escritura de los logs. **Valores esperados**: `Infinite`: Nunca se crearán nuevos archivos de registro y no se agregará información del período de tiempo al nombre del archivo. `Day` : Se creará un archivo todos los días y se agregará información del período de tiempo al nombre del archivo con el formato: `yyyyMMdd`. `Year`: Se creará un archivo cada año y el nombre del archivo tendrá un año de cuatro dígitos `aaaa`. `Month`: Se creará un archivo cada mes calendario y se agregará información del período de tiempo al nombre del archivo con el formato: `yyyyMM`. `Hour`: Se creará un archivo cada hora y se agregará información del período de tiempo al nombre del archivo con el formato: `yyyyMMddHH`. `Minute`: Se creará un archivo cada minuto y se agregará información del período de tiempo al nombre del archivo con el formato: `yyyyMMddHHmm` | `Day` |
| FlushToDiskInterval | Intervalo de tiempo (en segundos) en que se realizará una "descarga" completa de la información de logs a los respectivos archivos en disco. | `60` |
| SlackWebhookUri | URL Webhook del canal en Slack donde se enviarán los mensajes de notificación de errores. Si no se especifica una, los mensajes no se enviarán a Slack. | `null` |
| SeqServerUri | URL del servidor Seq donde se escribirán eventos de seguimiento. Ejemplo: `http://xxx.xxx.xxx.xxx:xxx/`. | `null` |
| SeqApiKey | Clave en el API de Seq que identifica a la aplicación que escribe los eventos. Cada clave puede asociarse con un conjunto de propiedades que se aplicarán automáticamente a los eventos escritos con ella. La clave no es obligatoria para la escritura de los eventos. Para ver las claves: [Seq Api Keys](http://localhost:port/#/settings/api-keys) | `null` |

## Configurar las notificaciones de Slack
Logging permite establcer un webhook personalizado para Slack para enviar a un canal de Slack información de una excepción. Una vez configurada la integración de Slack, Logging enviará todos los mensajes de error publicados a dicho canal.
Siga [estas instrucciones ](https://api.slack.com/incoming-webhooks) para obtener la URL del Webhook.

```AsciiDoc
Ejemplo de una URL de webhook en Slack
https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
```

```c#
var settings = new LoggerSettings();
settings.SlackWebhookUri = "http://slack.com/services/X/Y/Z";
var logManager = new LogManager(settings);
var anException = new ArgumentException("Lorem ipsum dolor sit amet, consectetuer adipiscing elit.");
logManager.Error(anException, "What happened here?");
```

![Preview](image_slack_alert.png)

## Configurar el registro de eventos
Cuando se escribe en logs la información de un `Error`, automáticamente se registra un evento en el sistema operativo con el origen `EventLogSourceName` en el registro de eventos `EventLogName`.

De forma predeterminada el nombre del registro de eventos es "_Processa_" o el valor de `EventLogName` y el nombre del origen será el valor de `EventLogSourceName`, de lo contrario se usará el nombre de aplicación o por defecto: "_Anonymous_".

:warning: Debe existir previamente el registro de eventos en el sistema operativo, de lo contrario los eventos **NO** serán registrados.

## Crear un registrador

Inicie un instancia de la clase `LogManager` y utilice los métodos `Debug`, `Info` y `Error` para registrar la información:

```c#
var logManager = new LogManager();
logManager.Debug("The time is {Now}", DateTime.Now);
logManager.Info(new { X = 1, Y = true });
var anException = new ArgumentException("Lorem ipsum dolor sit amet, consectetuer adipiscing elit.");
logManager.Error(anException, "What happened here?");
```

El formato de salida se puede modificar utilizando el objeto `LoggerSettings` de configuración, estableciendo valores personalizados en las propiedades `DebugTemplate`, `ErrorTemplate`, `InfoTemplate` y `EventLogTemplate` como se muestra a continuación:

```c#
LoggerSettings settings = new LoggerSettings();
settings.DebugTemplate = "MyCustomTemplate";
var logManager = new LogManager(settings);
```
Para más información vea: https://github.com/serilog/serilog/wiki/Formatting-Output

Propiedades integradas pueden aparecer en las plantillas de salida, como:

- **Message**: El mensaje del evento de registro, representado como texto sin formato
- **NewLine**: Una propiedad con el valor de System.Environment.NewLine
- **Timestamp**: Marca de tiempo del evento.

También puede agregar valores personalizados a través del método `PushVariable`, como se muestra a continuación:

```c#
LoggerSettings settings = new LoggerSettings();
settings.PushVariable("MyProperty1","MyValue1");
settings.PushVariable("MyProperty2","MyValue2");
settings.DebugTemplate = "{MyProperty1}{MyProperty2}";
var logManager = new LogManager(settings);
```

Por defecto el método `Error` filtra las excepciones de tipo `HttpListenerException`. Si desea agregar otras excepciones al filtrado (no reporte), pude utilizar el método `PopException`, como se muestra a continuación:

```c#
var settings = new LoggerSettings();
// Omitir el procesamiento de excepciones de tipo DivideByZero
settings.PopException(typeof(DivideByZeroException));
var logManager = new LogManager(settings);
```

## Leer errores del log de eventos del SO

Puede utilizar el comando que se muestra a continuación para leer los errores que se han guardado en el log de eventos del sistema operativo.

```powershell
Get-EventLog -LogName 'Processa'
```

## Ejemplo de uso básico

```c#
var logManager = new LogManager();
try
{
	logManager.Debug("The time is {Now}", DateTime.Now);
	logManager.Info(myObject);
}
catch(Exception exc){
	logManager.Error(exc, "What happened here?");
}
```

## Ejemplo de uso personalizado

```c#
LoggerSettings settings = new LoggerSettings();

// Guardar los archivos en la carpeta C:\Temp\MyLogs
settings.LogFolderPath = @"C:\Temp";
settings.LogFolderName = "MyLogs";
	
// Prefijo de los archivos.
settings.PrefixFileName = "MyApplication";

// Al guardar un error, informar a Slack.
settings.SlackWebhookUri = "http://slack.com/services/X/Y/Z";

// Conservar los archivos de Debug de las últimas 5 horas.
settings.RollingInterval = RollingInterval.Hour;
settings.DebugRetainedFile = 5;

// Escribir al log de Errores el path de la carpeta Mis documentos.
settings.PushVariable("MyDocumentsPath", Environment.SpecialFolder.MyDocuments.ToString());
settings.ErrorTemplate = "Timestamp: {Timestamp:yyyy-MM-dd HH:mm:ss.fff}{NewLine}Message: {Message}{NewLine}Exception: {Exception}Exception Data: {NewLine}{ExceptionData}{NewLine}MyDocumentsPath: '{MyDocumentsPath}'{NewLine}{Separator}{NewLine}";
	
// Omitir el procesamiento de excepciones de tipo DivideByZero
settings.PopException(typeof(DivideByZeroException));

LogManager logManager = new LogManager(settings);
	
// Escribir un log de debug.
logManager.Debug("The time is {Now}", DateTime.Now);
logManager.Debug(LogEventLevel.Information, "Slack WebHook: '{SlackWebhookUri}'", settings.SlackWebhookUri);
logManager.Debug(LogEventLevel.Error, "An uncontrolled exception message.", settings.SlackWebhookUri);
	
// Escribir un log de info.
logManager.Info(new { Value = 10, Guid = Guid.NewGuid().ToString(), Date = DateTime.Now });
	
// Escribir un log de error.
var anException = new ArgumentException("Lorem ipsum dolor sit amet, consectetuer adipiscing elit.");
anException.Data.Add("Value", "Value");
anException.Data.Add("Guid", Guid.NewGuid().ToString());
anException.Data.Add("Date", DateTime.Now);
logManager.Error(anException, "What happened here?");
	
// Las excepciones de tipo DivideByZero no se escribirán en logs.
try
{
	int number1 = 3000;
	int number2 = 0;
	Console.WriteLine(number1 / number2);
}
catch (DivideByZeroException exc)
{
	logManager.Error(exc, "This exception will be ignored.");
}
```
