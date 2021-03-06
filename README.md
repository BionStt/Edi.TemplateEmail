Edi.TemplateEmail
===============================

This library enable you to configure email in XML template and send the email in your .NET / .NET Core applications.

[![Build status](https://dev.azure.com/ediwang/EdiWang-GitHub-Builds/_apis/build/status/Edi.TemplateEmail-CI)](https://dev.azure.com/ediwang/EdiWang-GitHub-Builds/_build/latest?definitionId=-1)

## For .NET Core

Notice: .NET Core support is done via .NET Standard 2.0, it can support .NET Framework, but not recommended for this package yet.

```
PM > Install-Package Edi.TemplatedEmail.NetStd
```

### Step 1: Sample mailConfiguration.xml under your application root

```xml
<?xml version="1.0"?>
<MailConfiguration>
  <CommonConfiguration OverrideToAddress="false" ToAddress="overridetest@test.com" />
  <MailMessage MessageType="TestMail" IsHtml="true">
    <MessageSubject>Test Mail on {MachineName.Value}</MessageSubject>
    <MessageBody>
      <![CDATA[
Mail configuration on {MachineName.Value} is good: <br />
Smtp Server: {SmtpServer.Value}<br />
Smtp Port: {SmtpServerPort.Value}<br />
Smtp Username: {SmtpUserName.Value}<br />
Email Display Name: {EmailDisplayName.Value}<br />
Enable SSL: {EnableSsl.Value}<br />
      ]]>
    </MessageBody>
  </MailMessage>
</MailConfiguration>
```

Optional: You may need to set it to copy to output directory based on your usage.

### Step 2:

Define an EmailHelper object in your class.
```
public static EmailHelper EmailHelper { get; set; }
```

Get the configuration file path

```
var configSource = $"{Directory.GetCurrentDirectory()}\\mailConfiguration.config";
```

Initialize the EmailHelper by your mail server settings

```
var settings = new EmailSettings("smtp-mail.outlook.com", "Edi.Test@outlook.com", "YOURPASSWORD", 587)
{
    EnableSsl = true,
    EmailDisplayName = "Edi.TemplateEmail.NetStd",
    SenderName = "Test Sender"
};

EmailHelper = new EmailHelper(configSource, settings);
```

Optional: You can also define event handlers on this

```
EmailHelper.EmailSent += (sender, eventArgs) =>
{
    if (sender is MailMessage msg)
        Console.WriteLine($"Email {msg.Subject} is sent, Success: {eventArgs.IsSuccess}");
};
EmailHelper.EmailFailed += (sender, eventArgs) =>
{
    Console.WriteLine("Failed");
};
EmailHelper.EmailCompleted += (sender, e) =>
{
    Console.WriteLine("Completed.");
};
```

### Step 3: Map the Configuration Values and Send Email

```
public static async Task TestSendTestMail()
{
    bool isOk = true;

    var pipeline = new TemplatePipeline().Map("MachineName", Environment.MachineName)
        .Map("SmtpServer", EmailHelper.Settings.SmtpServer)
        .Map("SmtpServerPort", EmailHelper.Settings.SmtpServerPort)
        .Map("SmtpUserName", EmailHelper.Settings.SmtpUserName)
        .Map("EmailDisplayName", EmailHelper.Settings.EmailDisplayName)
        .Map("EnableSsl", EmailHelper.Settings.EnableSsl);

    EmailHelper.EmailFailed += (s, e) =>
    {
        isOk = false;
    };

    await EmailHelper.ApplyTemplate("TestMail", pipeline).SendMailAsync("Edi.Wang@outlook.com");
}
```
