# Live Project

## **Introduction**

For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC/Entity Framework Code First in C#. Working with the .NET Framework was a great learning oppertunity for fixing bugs, cleaning up code, and adding requested features. There were some big changes that could have been a large time sink, but working in a scrum envirnoment with Azure DevOps Server, we used what we had to deliver what was needed on time. I saw how a good developer works with what they have to make a quality product. I worked on several back end stories. Because much of the site had already been built, there were also a good deal of front end stories and UX improvements that needed to be completed, all of varying degrees of difficulty. Everyone on the team had a chance to work on front end and back end stories. Over the two week sprint I also had the opportunity to work on some other project management and team programming skills that I'm confident I will use again and again on future projects.
<br />
Below are descriptions of some of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.
<br />

## **Back End Stories**
<br />

##  **Anchor Overload**

I needed to add an overload method to the Anchor Button that takes a Controller name (ie Jobs, JobActions, Schedules, etc) and renders buttons for that specific set of pages. The project did not want to have to rewrite the code for existing buttons, they just wanted flexibility in creating new button
<br />
``` 

     public static MvcHtmlString AnchorButton(this HtmlHelper helper, AnchorType Type, string Url, string controller) 
     //Overload added to render a custom AnchorButton to a path outside of class
    {
        switch (Type)
        {
            case AnchorType.BackToList:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-arrow-left"));
            case AnchorType.Create:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-plus-square"));
            case AnchorType.Delete:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-trash"));
            case AnchorType.Details:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-info-circle"));
            case AnchorType.Edit:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-pencil"));
            // new save button
            case AnchorType.Save:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-floppy-o"));
            // new home/Menu button
            case AnchorType.Home:
                return new MvcHtmlString(ButtonString(Url, controller, "fa-arrow-left"));
            default:
                return new MvcHtmlString(ButtonString(Url, controller));
        }
    }
    private static string ButtonString(string url, string text, string icon = "")
    {
        var anchor = new TagBuilder("a");
        anchor.MergeAttribute("type", "button");
        anchor.AddCssClass("btn btn-sm btn-primary");
        anchor.MergeAttribute("href", url);

        var span = new TagBuilder("span");
        if (icon != "")
        {
            var i = new TagBuilder("i");
            i.AddCssClass($"fa {icon}");
            span.InnerHtml = $"{i.ToString(TagRenderMode.Normal)}";
        }
        span.InnerHtml += $" {text}";
        anchor.InnerHtml = $"{span.ToString(TagRenderMode.Normal)}";
        return anchor.ToString(TagRenderMode.Normal);
    }
}
```
