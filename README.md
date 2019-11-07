# Live Project

## **Introduction**

For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC/Entity Framework Code First in C#. Working with the .NET Framework was a great learning oppertunity for fixing bugs, cleaning up code, and adding requested features. There were some big changes that could have been a large time sink, but working in a scrum envirnoment with Azure DevOps Server, we used what we had to deliver what was needed on time. I saw how a good developer works with what they have to make a quality product. I worked on several back end stories. Because much of the site had already been built, there were also a good deal of front end stories and UX improvements that needed to be completed, all of varying degrees of difficulty. Everyone on the team had a chance to work on front end and back end stories. Over the two week sprint I also had the opportunity to work on some other project management and team programming skills that I'm confident I will use again and again on future projects.
<br />
Below are descriptions of some of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.
<br />

## **Back End Stories**
* [Project Anchor Overload](https://github.com/therubini/Live-Project/blob/master/README.md#project-anchor-overload)
* [Project Contact Successful](https://github.com/therubini/Live-Project/blob/master/README.md#project-contact-successful)
* [Project Debut the Create Profile](https://github.com/therubini/Live-Project/blob/master/README.md#project-debug-the-create-profile)
<br />

##  **Project Anchor Overload**

I needed to add an overload method to the Anchor Button that takes a Controller name (ie Jobs, JobActions, Schedules, etc) and renders buttons for that specific set of pages. The project did not want to have to rewrite the code for existing buttons, they just wanted flexibility in creating new button
<br />
```C# 

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
```C#

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
#### The View Page
```C#
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
```C#


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
## **Project SideNav Bar**
On a mobile view, or when the screen shrinks below 990 px, it collapses the navbar. I created a 3-bars Humburger menu which became visible on the left top corner of the page. The nav-content will be displayed on click to the 3-bars Humburgen menu with a 100% width. 
#### Javascript to render the effects of the CSS into HTML
```Javascript
$("#toggle").click(function () {
    $(this).toggleClass("open");
    $("#menu").toggleClass("opened");
});
```
#### Code for when the screen shrinks: non-mobile
```CSS
/*styles for max width 990px. No nav bar was rendering between 772px to 990px. The adustments allowed a functional nav bar to display in all px*/
@media screen and (max-width: 990px) {

* {
    margin: 0;
    padding: 0;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}
#toggle {
    position: fixed;
    z-index: 3;
    width: 3em;
    height: 3em;
    top: 0;
    left: 0;
    margin: 15px 0 0 15px;
}
#toggle span {
    display: block;
    position: absolute;
    width: 100%;
    height: 0.2em;
    margin: 1.25em 0 0 0;
    background: var(--dark-blue);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#toggle span:before, #toggle span:after {
    content: " ";
    position: absolute;
    width: 100%;
    height: 0.2em;
    background: var(--dark-blue);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#toggle span:before {
    margin: -1em auto;
}
#toggle span:after {
    margin: 1em auto;
}
#toggle.open span:before, #toggle.open span:after {
    margin: 0;
    left: 3.2em;
    top: -1em;
    background: var(--dark-blue);
}
#toggle.open span:before {                     /*toggle effects when clicked*/
    -webkit-transform: rotate(135deg);
    -moz-transform: rotate(135deg);
    transform: rotate(135deg);
}
#toggle.open span:after {
    -webkit-transform: rotate(-135deg);
    -moz-transform: rotate(-135deg);
    transform: rotate(-135deg);
}
#menu {
    visibility: hidden;
    opacity: 0;
    position: fixed;
    z-index: 50;
    text-align: center;
    background: var(--dark-blue);
    -webkit-transform: scale(1.5);
    -moz-transform: scale(1.5);
    transform: scale(1.5);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#menu.opened {                /*How the menu views when opened*/
    visibility: visible;
    opacity: 1;
    -webkit-transform: scale(1);
    -moz-transform: scale(1);
    transform: scale(1);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#menu ul li a i {
    position: center;           /*aligned menu to align without overlap*/
    padding: 0 1.25em 0 0;
    font-size: 2em;
}

} /*end styles for max-width 990*/
```
#### Code display: Mobile
```CSS

/***** Navbar Styling *****/

#toggle {
    position: fixed;
    z-index: 50;
    width: 2.8571428571em;
    height: 2.8571428571em;
    top: auto;
    left: 0px;
    margin: 15px 0 0 15px;
    cursor: pointer;
}
#toggle span {
    display: block;
    position: absolute;
    width: 100%;
    height: 0.2em;
    margin: 1.25em 0 0 0;
    background: var(--dark-blue);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#toggle span:before, #toggle span:after {
    content: " ";
    position: absolute;
    width: 100%;
    height: 0.2em;
    background: var(--dark-blue);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#toggle span:before {
    margin: -1em 0 0 0;
}
#toggle span:after {
    margin: 1em 0 0 0;
}
#toggle.open span {
    background-color: transparent;
}
#toggle.open span:before, #toggle.open span:after {
    margin: 0;
    background: #286efa;
}
#toggle.open span:before {
    -webkit-transform: rotate(135deg);
    -moz-transform: rotate(135deg);
    transform: rotate(135deg);
}
#toggle.open span:after {
    -webkit-transform: rotate(-135deg);
    -moz-transform: rotate(-135deg);
    transform: rotate(-135deg);
}
/************BEGIN LEFT SIDE NAVBAR STYLING**********/

#menu {
    visibility: hidden;
    opacity: 0;
    position: fixed;
    z-index: 10;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    text-align: center;
    font-size: 1.25em;
    background-color: white;
    -webkit-transform: scale(1.5);
    -moz-transform: scale(1.5);
    transform: scale(1.5);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
#menu.opened {
    visibility: visible;
    position:fixed;
    z-index:9;
    width:100%;              /*Adjusted to allow menus/sub menus to fit entirly in nav bar*/
    top: -50px;
    height:auto;
    opacity: 1;
    -webkit-transform: scale(1);
    -moz-transform: scale(1);
    transform: scale(1);
    -webkit-transition: 350ms ease all;
    -moz-transition: 350ms ease all;
    transition: 350ms ease all;
}
.dropdownx {                   /*Used dropdowns to implement the sub menus*/
    position: relative;
    display: block;
}
.dropdownx-content {
    display: none;
    position: center;
    background-color: var(--blue-gray);
    min-width: 160px;
    box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
    padding: 12px 16px;
    z-index: 5;
}
.dropdownx:hover .dropdownx-content {
    display: block;
}
#menu ul li a {     
    position:relative;
    color: white;
    z-index: 4;
    display: block;
    width: 90%;
    height: 65px;
    margin: 1em auto 0.5em auto;
}
#menu ul li a i {
    position: center;
    padding: 0 1.25em 0 0;
    font-size: 2em;
}
/*************END LEFT NAVBAR STYLING*****************/
```
## **Project Product Details Styling**
I was tasked to style the Product/details to view like all other details view pages in the site. I was to implement button partials which paths led to the save and back to list. I was also tasked to create an Add to Cart button that only displayed an icon which styled similar to the partial buttons.
```C#

@{
    ViewBag.Title = "Details";
}

<h2>Product Details</h2>               <!--Title updated-->

<div class="container rounded w-50">
    <!--Container created with width of 50% dimension -->
    <div class="card" style="opacity: .8; border-radius: 20px; padding: 50px">
        <div class="text-center">
            <dl class="dl-horizontal">
                <dd>
                    <img src="~/Content/images/Product/@(Model.ImagePath)" class="card-img-top mx-auto center" alt="Image"                                      style="width:210px;height:200px;" />
                </dd>

                <dd>
                    @Html.DisplayFor(model => model.ProductName)
                </dd>

                <dt>
                    Description
                </dt>

                <dd>
                    @Html.DisplayFor(model => model.ProductDescription)
                </dd>

                <dt>
                    Price
                </dt>

                <dd>
                    $@Html.DisplayFor(model => model.UnitPrice)
                </dd>
            </dl>
        </div>
        <p class="text-center">
            @Html.AnchorButton(AnchorType.BackToList, Url.Action("Index"))   

            <a href="@Url.Action("AddToCart", "CartItem", new { id = Model.ProductId }, null)"title="Add to Cart"><i class="btn btn-sm                  btn-primary fa fa-shopping-cart" style="height:30px"></i></a>
            <!--Shopping Cart icon used for the "Add to Cart" button styled to match the anchorbutton-->
        
        </p>
    </div>
</div>
```
## **Other Skills Learned**
* Working with a group of developers to identify front and back end bugs to the improve usability of an application
* Improving project flow by communicating about who needs to check out which files for their current story
* Learning new efficiencies from other developers by observing their workflow and asking questions
    * I implemented new properties to Models to render new partial buttons to Controllers in order to create efficient solutions for the        team
* Practice with team programming/pair programming when one developer runs into a bug they cannot solve 
    * One of the developers on the team was having trouble with the back end ActionButton partials to render a path and display as      needed by the project manager. I was able to help them identify the issue with the logic in order to produce the wanted results. I completed the project, and I showed them how. 
