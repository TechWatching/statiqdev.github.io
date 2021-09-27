Title: Porting From Wyam
Order: 7
---
Statiq Web is roughly compatible with the Wyam blog recipe, but Statiq and Wyam are two different projects and not everything has a direct equivalent, especially if you customized your Wyam site. That said, whatever you could do in Wyam you can almost certainly do in Statiq, even if it's a little different. If you're having trouble figuring out how to port a particular feature, please [head over to the discussions](https://github.com/statiqdev/Discussions/discussions) and let us know so we can help out.

Here are some notes if you're porting from Wyam to Statiq Web:

- You will need to [create a Statiq Web app](/web) at the root of your site (you can keep the `input` directory).
  - Run `dotnet new console` at the root of your site.
  - Run `dotnet add package Statiq.Web --version x.y.z` (using the [latest Statiq Web version](https://www.nuget.org/packages/Statiq.Web)).
  - Change the generated `Program` class in `Program.cs` to:

```csharp
using System;
using System.Threading.Tasks;
using Statiq.App;
using Statiq.Web;

namespace ...
{
  public class Program
  {
    public static async Task<int> Main(string[] args) =>
      await Bootstrapper
        .Factory
        .CreateWeb(args)
        .RunAsync();
  }
}
```

- Find a [theme](xref:web-themes) and follow the [installation instructions](xref:web-themes#installation) to install the theme into your site (optional).

- Create a `settings.yml` file at the root of your site and copy over settings from your `config.wyam` file
  - Since the new settings file is YAML you don't need to prefix strings or anything, for example `Settings[Keys.Host] = "daveaglick.com";` becomes `Host: daveaglick.com`.
  - If you defined a global "Title" setting in `config.wyam` the new theme should set "SiteTitle" instead (and if not, a "SiteTitle" should be defined).
  - If you defined an "Intro" setting, that should be placed in a new `_index.yml` file in your `input` directory with a key of "Description".

- If you created an `input/assets/css/override.css` file, move it to `input/scss/_overrides.scss` (and you can now use Sass inside the CSS overrides file).

- Replace any uses of `img-responsive` CSS class with `img-fluid` since this theme uses a newer version of Bootstrap and that CSS class changed.

- Rename and fix up any override theme files or partials according to the supported ones documented in the new theme.
  - For example, the old Wyam CleanBlog supported a `_PostFooter.cshtml` which should be renamed to `_post-footer.cshtml` in the new Statiq [CleanBlog](https://github.com/statiqdev/CleanBlog) theme.
  - The CSS may not match exactly, especially if you're changing themes, so you may need to take a look at the default partial implementations in the new theme and adjust your override files accordingly.

- You can likely remove any build scripting and bootstrapping code since you can now run `dotnet run preview` to preview the site.
  - You can also now setup [built-in deployment](xref:web-deployment).

- If you have Disqus code for comments on your blog that looks like something like this:

```javascript
var disqus_identifier = '@Model.FilePath(Keys.RelativeFilePath).FileNameWithoutExtension.FullPath';
var disqus_title = '@Model.String(Wyam.Blog.BlogKeys.Title)';
var disqus_url = '@Context.GetLink(Model)';
```

You can change it to this:

```javascript
var disqus_identifier = '@Document.Destination.FileNameWithoutExtension.FullPath';
var disqus_title = '@Document.GetString("Title")';
var disqus_url = '@Document.GetLink(true)';
```
