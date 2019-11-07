# Live Project

## **Introduction**

For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC/Entity Framework Code First in C#. Working with the .NET Framework was a great learning oppertunity for fixing bugs, cleaning up code, and adding requested features. There were some big changes that could have been a large time sink, but working in a scrum envirnoment with Azure DevOps Server, we used what we had to deliver what was needed on time. I saw how a good developer works with what they have to make a quality product. I worked on several back end stories. Because much of the site had already been built, there were also a good deal of front end stories and UX improvements that needed to be completed, all of varying degrees of difficulty. Everyone on the team had a chance to work on front end and back end stories. Over the two week sprint I also had the opportunity to work on some other project management and team programming skills that I'm confident I will use again and again on future projects.
<br />
Below are descriptions of some of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.
<br />

## **Back End Stories**
<br />

##  **Project Anchor Overload**

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
## **Project Contact Successful**
I was tasked with creating a view page, once a submission form was validated. I also tested email errors in order to render proper messages. When a user successfully submits a contact form, the success flash message they receive is the Empty contact form itself. 
We want to upgrade this success message to improve our user experience.We would like to see a separate view page.
```

public class ContactUsController : Controller
    {
        [HttpGet]
        public ActionResult Index()
        {
            return View();
        }
        [HttpPost]
        public ActionResult Index(ContactUs userInput)
        {
            if (ModelState.IsValid)
            {
                var toAddress = "liveprojectdummyacct@gmail.com";
                var fromAddress = userInput.Email.ToString();
                var subject = userInput.Subject;
                var message = "<b>Name: </b>" + userInput.Name + "<br>" + "<b>Email: </b>" + userInput.Email + "<br>" + "<b>Telephone: </b>" + userInput.Phone + "<br><br>" + "<b>Message: </b>" + userInput.Message;

                SendEmail(toAddress, fromAddress, subject, message);
                return View("Success");
            }
            return View();
        }

        public void SendEmail(string toAddress, string fromAddress, string subject, string message)
        {
            using (MailMessage newMessage = new MailMessage())
            {
                newMessage.From = new MailAddress(fromAddress);
                newMessage.To.Add(new MailAddress(toAddress));
                newMessage.Subject = subject;
                newMessage.Body = message.ToString();
                newMessage.IsBodyHtml = true;
                try
                {
                    SmtpClient smtp = new SmtpClient();
                    smtp.Host = "smtp.gmail.com";
                    smtp.Port = 587;
                    smtp.Credentials = new NetworkCredential
                    ("liveprojectdummyacct@gmail.com", "ASDqwe123!");
                    smtp.EnableSsl = true;
                    smtp.Send(newMessage);
                    ModelState.Clear();
                    ViewBag.Message = "Thank you for contacting us! Someone will get back to you shortly. ";
                }
                catch (SmtpFailedRecipientsException ex)
                {
                    foreach (SmtpFailedRecipientException t in ex.InnerExceptions)
                    {
                        var status = t.StatusCode;
                        if (status == SmtpStatusCode.MailboxBusy ||
                            status == SmtpStatusCode.MailboxUnavailable)
                        {
                            Response.Write("Delivery failed - retrying in 5 seconds.");
                            Thread.Sleep(5000);
                            //resend
                            SmtpClient smtp = new SmtpClient();
                            smtp.Host = "smtp.gmail.com";
                            smtp.Port = 587;
                            smtp.Credentials = new NetworkCredential
                            ("liveprojectdummyacct@gmail.com", "ASDqwe123!");
                            smtp.EnableSsl = true;
                            smtp.Send(newMessage);
                        }
                        else
                        {
                            ViewBag.Message = $"Failed to deliver message to " + t.FailedRecipient;
                        }
                    }
                }
                catch (Exception ex)
                {
                    ViewBag.Message = $"A secure connection or the client was not authenticated.";
                }
                finally
                {
                    newMessage.Dispose();
                }
            }
        }
    }
```
## The View Page
```
@{
    ViewBag.Title = "Success";
}

<br />
<br />
<br />
<div class="jumbotron form-horizontal formContainer" style="opacity: .8; border-radius: 20px; padding: 50px">
    <svg viewBox="0 10 250 20"><text x="0" y="25">Boom. Your form was submitted.</text></svg>
    <br />
    <br />
    <p class="lead font-weight-bold">@Html.Raw(ViewBag.Message)</p>
    <br />
    <p class="text-center">
        @Html.AnchorButton(AnchorType.Home, Url.Action("Index", "Home")) @*User friendly anchor button to redirect them to home*@
    </p>

</div>
```
## **Project Debug the Create profile**
There was a class for a Personal Profile that has been buggy since implementation. The application would crash when an existing user tried to create a new personal profile from the Manage Profile secion. My task was to track down this error and fix whatever is causing it.
```


public class ProfileController : Controller
    {
        // GET: PersonalProfiles/Create
        public ActionResult Create()      //Deleted parameter of "String Id" 
        {
            ViewBag.ProfileID = User.Identity.GetUserId();
            return View();
        }

        // POST: PersonalProfiles/Create
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "ProfileId,AboutMe,TagLine")] PersonalProfile PersonalProfiles)
        { //Added a try/catch to prevent application from crashing when user tries to create another profile on top of existing profile
            try
            {
                if (ModelState.IsValid)
                {
                    db.Profile.Find(PersonalProfiles.ProfileID);
                    db.Profile.Add(PersonalProfiles);
                    db.SaveChanges();
                    return RedirectToAction("Index");
                }
            }
            catch (Exception)
            {
                ModelState.AddModelError("", "User profile already exists. Make any changes to the EDIT button. Thanks!");
            }
            ViewBag.ProfileID = new SelectList(db.Users, "Id", "DisplayName", PersonalProfiles.ProfileID);
            return View(PersonalProfiles);
        }
    }
```
