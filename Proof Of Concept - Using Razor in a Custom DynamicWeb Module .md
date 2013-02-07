# Using Razor in a Custom DynamicWeb Module (PROOF OF CONCEPT)

*I am sorry for my bad english.. :-(*

I have started working with DynamicWeb (cms). With DynamicWeb you have the ability
to develop custom modules for you pages. These custom modules can contain custom C# code, that you can use to develop more custom stuff.

I want to tell you how you can use razor in you modules..
Normally you would write code as this:

    StringBuilder strBuiler = new StringBuilder();
    strBuiler.Append("<h1>" + model.Headline + "</h1>");
    strBuiler.Append("<hr />");
    strBuiler.Append("<ul>");
    foreach (var name in model.Names)
    {
        strBuiler.Append("<li>" + name + "</li>");
    }
    strBuiler.Append("</ul>");

That code is not pretty or easy to bugfix. If you have worked with ASP.net MVC and Razor, then the result will look familiar:

    <h1>@Model.Headline</h1>
    <hr />
    <ul>
        @foreach (var name in Model.Names)
        {
            <li>@name</li>
        }
    </ul>

## Setting up
To do this I need 2 dependencies
    
- System.Web.Razor
- RazorEngine (http://nuget.org/packages/RazorEngine/)

When I have added these 2 dependencies to my project I am ready to create some sort of loader.

### Visual Studio
Visual Studio is not very happy to have razor-files when you not develop a MVC project. You can get around this by adding this code to your web.config file:

      <system.web>
        <compilation>
          <buildProviders>
            <add extension=".cshtml" type="RazorEngine.Web.CSharp.CSharpRazorBuildProvider, RazorEngine.Web" />
            <add extension=".vbhtml" type="RazorEngine.Web.VisualBasic.VBRazorBuildProvider, RazorEngine.Web" />
          </buildProviders>
        </compilation>
      </system.web>

Visual Studio is still not happy with the razor-file, and can not find "@Model". That is because "@Model" is built-in as a extension in the MVC-assembly, but you can still use "@Model" and "@model" but I could not get intellisense support. :/

### Razor Loader
I have built a small class that can read a razor-file and return the generated HTML:

    public class RazorLoader
    {
        public string GenerateHtml<T>(string razorViewPath, T model)
        {
            string result;

            using (Stream stream = new FileStream(
                HttpContext.Current.Server.MapPath(razorViewPath),
                FileMode.Open,
                FileAccess.Read))
            {
                using (StreamReader reader = new StreamReader(stream))
                {
                    result = RazorEngine.Razor.Parse(reader.ReadToEnd(), model);
                }
            }

            return result;
        }
    }

### Using it!
Now I am ready to use the razor loader in a ContentModule:

    public class NameMViewodel
    {
        public string Headline { get; set; }
        public List<string> Names { get; set; }
    }

    [AddInName("RazorTester")]
    public class RazorTester : ContentModule
    {
        public override string GetContent()
        {
            var model = new NameMViewodel
            {
                Headline = Properties["Headline"].ToString(),
                Names = new List<String>() { "Oliver", "Kim", "Lars", "Micheal" }
            };

            return new RazorLoader().GenerateHtml("~/CustomModules/RazorTester/test.cshtml", model); ;
        }
    }

# Conclusion
Do it! It seems to work.. ;-)
