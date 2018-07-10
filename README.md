# Processa.Utilities.Logging

Facilita el registro de información de diagnóstico (*logging*) para una aplicación. Expone métodos para generar tres tipos de información:

![Preview](image_preview.png)


- **Debug**: Información de depuración. Básicamente se trata de mensajes de texto adicional a la fecha y hora en que se escribió el mensaje. Los archivos se rotan cada dos días.


- **Error**: Información de excepciones. Guarda el registro de excepciones (mensaje, tipo, pila, etc) generadas por la aplicación. Los archivos se rotan cada 31 días. Ademas de guardar la información en un archivo, deja una copia de la misma en el log de eventos del sistema operativo y si se establece un valor para la propiedad `SlackUrl`, también envía un mensaje al Webhook configurado.

- **Info**: Información de seguimiento. Guarda un objeto en formato `JSON`. Si desea, puede excluir algunas propiedades del objeto antes de guardar. Los archivos se rotan cada dos días.


## Conceptos básicos de configuración

`LogManager` utiliza un API de configuración simple a través de la clase `LoggerSettings`. El constructor predeterminado de `LoggerSettings` implementa una configuración básica que en muchos casos no tendrá que cambiar.

La configuración predeterminada establece una subcarpeta Logs en la misma carpeta del archivo binario (generalmente `bin`), reteniendo los últimos dos archivos `Debug` e `Info` y los últimos 31 archivos `Error`.

Si desea personalizar la configuración, basta con cambiar el valor de algunas de sus propiedades. Por ejemplo el siguiente código establece la carpeta donde se generan los archivos de logs a `C:\Temp`.

```c#
LoggerSettings settings = new LoggerSettings();
settings.RootFolderPath = @"C:\Temp";
var logManager = new LogManager(settings);
```

## Configurar las notificaciones de Slack
Logging permite establcer un webhook personalizado para Slack para enviar a un canal de Slack información de una excepción. Una vez configurada la integración de Slack, Logging enviará todos los mensajes de error publicados a dicho canal.

Siga [estas instrucciones ](https://api.slack.com/incoming-webhooks) para obtener la URL del Webhook.

```AsciiDoc
Ejemplo de una URL de webhook en Slack
https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
```

```c#
var settings = new LoggerSettings();
// Vea: https://api.slack.com/incoming-webhooks
// Reemplace el valor "http://slack.com/services/X/Y/Z" por la URL de su webhook.
settings.SlackUrl = "http://slack.com/services/X/Y/Z"
var logManager = new LogManager(settings);
logManager.Error(AnException, "What happened here?");
```
## Crear un registrador

Inicie un instancia de la clase `LogManager` y utilice los métodos `Debug`, `Info` y `Error` para registrar la información:

```c#
var logManager = new LogManager();
logManager.Debug("The time is {Now}", DateTime.Now);
logManager.Info(new {X=1,Y=true});
logManager.Error(AnException, "What happened here?");
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
var settings = new LoggerSettings();

// Guardar los archivos en la carpeta C:\Temp\MyLogs
settings.RootFolderPath = @"C:\Temp";
settings.LogFolderPath = "MyLogs"

// Al guardar un error, informar a Slack.
settings.SlackUrl = "http://slack.com/services/X/Y/Z";

// Conservar los archivos de Debug de las últimas 5 horas.
settings.RollingInterval = RollingInterval.Hour;
settings.DebugRetainedFile = 5;

// Escribir al log el path de la carpeta Mis documentos
settings.PushVariable("MyDocumentsPath", Environment.SpecialFolder.MyDocuments.ToString());

// Omitir el procesamiento de excepciones de tipo DivideByZero
settings.PopException(typeof(DivideByZeroException));

var logManager = new LogManager(settings);
try
{
	logManager.Debug("The time is {Now}", DateTime.Now);
	logManager.Info(myObject);
}
catch(Exception exc){
	logManager.Error(exc, "What happened here?");
}
```
