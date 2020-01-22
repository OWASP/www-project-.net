=Description=
Cross-site scripting attacks exploit vulnerabilities in web page validation by injecting client-side script code. The script code embeds itself in response data, which is sent back to an unsuspecting user. In addition to validating input, any data retrieved from untrusted or shared sources should be encoded on output. For example: data retrieved from a database that may have had malicious input persisted to it.

=Encoding Output Values in Code=
Use <code>Server.HtmlEncode</code> to encode untrusted data for use in HTML output:
<pre>var encodedHtml = Server.HtmlEncode(untrustedData);</pre>

Use <code>Server.UrlEncode</code> to encode untrusted data for use in the key/value pairs of a URL query string.
<pre>var url = "https://www.bing.com/search?q=" + HttpUtility.UrlEncode(untrustedData);</pre>

=Encoding Output Values in HTML markup=
Starting with ASP.NET 4.0 you can HTML encode values in markup with the <code><%: %></code> syntax, as shown below.
<pre><span><%: untrustedData %></span></pre>

Or, in Razor syntax, you can HTML encode with <code>@</code>, as shown below.
<pre><span>@untrustedData</span></pre>

Starting with ASP.NET 4.5 you can also HTML encode the result of data-binding expressions. Just add a colon (:) to the end of the <%# prefix that marks the data-binding expression:
<pre>
<asp:TemplateField HeaderText="Name">
    <ItemTemplate><%#: Item.Products.Name %></ItemTemplate>
</asp:TemplateField>
</pre>

=IHtmlString=
If you have model properties that are used to display raw HTML you should consider using the <code>HtmlString</code> and <code>MvcHtmlString</code> classes starting with .NET 4.0. These both implement the <code>IHtmlString</code> interface and will instruct ASP.NET to skip output encoding when using <code><%: model.Property %></code> or <code>@model.Property</code> in HTML markup. Converting a property on your view model from <code>String</code> to <code>MvcHtmlString</code> will instruct ASP.NET that HTML encoding has already been accounted for.

<pre>public class User 
{
    public int Id { get; set; }
    public string Name { get; set; }
    public MvcHtmlString Description { get; set; } // Output encoding is handled manually
}</pre>

=AntiXssEncoder=
By default the ASP.NET encoding methods use a black-listing technique that evaluates the string for a set of character combinations that may indicate presence of malicious script. A superior approach is to use a [[Input Validation Cheat Sheet#White_List_Input_Validation|white-listing technique]] for validation, which can be achieved using the Anti-Cross Site Scripting Library from Microsoft. Starting with ASP.NET 4.5 you can specify that the <code>AntiXssEncoder</code> from this library be used as the default encoder for you entire application using the <code>encoderType</code> setting in web.config as shown below.
<pre><httpRuntime encoderType="System.Web.Security.AntiXss.AntiXssEncoder" /></pre>

If you are using a version of .NET earlier than 4.5, you will need to download and include the library as a reference to your project, and then use the earlier library name for the encodeType setting as shown below.
<pre><httpRuntime encoderType="Microsoft.Security.Application.AntiXssEncoder, AntiXssLibrary" /></pre>

In addition to the common <code>HtmlEncode</code> and <code>UrlEncode</code> methods, the Anti-Cross Site Scripting Library provides the following <code>AntiXssEncoder</code> methods for more specialized output encoding needs:

==CssEncode==
Encodes the specified string for use in cascading style sheets (CSS). This method encodes all characters except those that are in the safe list, by using the CSS escape character (/) followed by up to six hexadecimal digits.
{| class="wikitable"
|<code>alert('XSS Attack!');</code>
|<code>alert\000028\000027XSS\000020Attack\000021\000027\000029\00003B</code>
|-
|<code>user@contoso.com</code>
|<code>user\000040contoso\00002Ecom</code>
|}

==HtmlFormUrlEncode==
Encodes the specified string for use in form submissions whose MIME type is "application/x-www-form-urlencoded". This method encodes all characters except those that are in the safe list. Characters are encoded by using %SINGLE_BYTE_HEX notation.
{| class="wikitable"
|<code>alert('XSS Attack!');</code>
|<code>alert%28%27XSS+Attack%21%27%29%3B</code>
|-
|<code>user@contoso.com</code>
|<code>user%40contoso.com</code>
|}

==XmlAttributeEncode==
Encodes the specified string for use in XML attributes, and is slightly more restrictive than XmlEncode below. This method encodes all characters except those that are in the safe list. Characters are encoded by using &#DECIMAL; notation.
{| class="wikitable"
|<code>alert('XSS Attack!');</code>
|<code>alert(&amp;apos;XSS&amp;#32;Attack!&amp;apos;);</code>
|-
|<code><script>alert('XSSあAttack!');</script></code>
|<code>&amp;lt;script&amp;gt;alert(&amp;apos;XSS&amp;#12354;Attack!&amp;apos;);&amp;lt;/script&amp;gt;</code>
|}

==XmlEncode==
Encodes the specified string for use in XML. This method encodes all characters except those that are in the safe list. Characters are encoded by using &#DECIMAL; notation.
{| class="wikitable"
|<code>alert('XSS Attack!');</code>
|<code>alert(&amp;#39;XSS&amp;#32;Attack!&amp;#39;);;</code>
|-
|<code><script>alert('XSSあAttack!');</script></code>
|<code>&amp;lt;script&amp;gt;alert(&amp;apos;XSS&amp;#12354;Attack!&amp;apos;);&amp;lt;/script&amp;gt;</code>
|}

=References=
*[http://blogs.msdn.com/b/cisg/archive/2008/08/28/output-encoding.aspx Output Encoding]
*[http://msdn.microsoft.com/en-us/library/System.Web.HttpUtility_methods(v=vs.110).aspx HttpUtility Methods]
*[http://blogs.msdn.com/b/sfaust/archive/2008/09/02/which-asp-net-controls-automatically-encodes.aspx Which ASP.NET Controls Automatically Encode?]
*[http://weblogs.asp.net/scottgu/new-lt-gt-syntax-for-html-encoding-output-in-asp-net-4-and-asp-net-mvc-2 New <%: %> Syntax for HTML Encoding Output in ASP.NET 4 (and ASP.NET MVC 2)]
*[http://www.asp.net/aspnet/overview/aspnet-and-visual-studio-2012/whats-new#_Toc318097391 HTML Encoded Data-Binding Expressions]
*[http://msdn.microsoft.com/en-us/library/system.web.ihtmlstring(v=vs.110).aspx IHtmlString Interface]
*[http://stackoverflow.com/questions/2293357/what-is-an-mvchtmlstring-and-when-should-i-use-it What is an MvcHtmlString and when should I use it?]
*[http://www.microsoft.com/en-sg/download/details.aspx?id=43126 Microsoft Anti-Cross Site Scripting Library V4.3]
*[http://msdn.microsoft.com/en-us/library/system.web.security.antixss.antixssencoder_methods(v=vs.110).aspx AntiXssEncoder Methods]
*[[ASP.NET Request Validation]]

[[Category:OWASP .NET Project]]
