# Live Project

## Introduction
For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale .NET MVC Web Application using Entity Framework in C#. The project was a website for a local Theater company that inculded an Admin section for non-technical theater employees to manage the site. I really enjoyed working on imrovements to an existing code base and seeing my changes in action. This was an excelent experience in working on a team using Agile/Scrum methodology through Azure DevOps.  I learned a lot from I working on several stories that involved changes to [back end](#back-end-stories) and [front end](#front-end-stories).  

Here is a brief description of the stories I completed:

## Back End Stories

* [Add Photo Modal](#Add-Photo-Modal)
* [Bug Fix](#bug-fix)


### Add Photo Modal

The Theater website Admin section had a list of models that were missing photos.  My task was to add a button next to each model on the list that would open a modal where to admin could attach a photo to that model.  I created a modal in the view for each model type and passed the information back to this method in the controller which creates a photo and attaches it to the apropriate model.  

```
        // Method called from dashboard modals to add a photo to models missing photos 
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult AddPhoto(HttpPostedFileBase file, string title, int id, string modelType, string description = "")
        {
            if (modelType == "castMember")
            {
                CastMember castmember = db.CastMembers.Find(id);
                castmember.PhotoId = PhotoController.CreatePhoto(file, title);
            }
            else if (modelType == "sponsor")
            {
                Sponsor sponsor = db.Sponsors.Find(id);
                sponsor.PhotoId = PhotoController.CreatePhoto(file, title);
            }
            else if (modelType == "prodPhoto")
            {
                ProductionPhotos productionPhoto = db.ProductionPhotos.Find(id);
                productionPhoto.PhotoId = PhotoController.CreatePhoto(file, title);
                productionPhoto.Description = description;
            }
            else if (modelType == "production")
            {
                //Create new production photo
                ProductionPhotos productionPhoto = new ProductionPhotos();
                productionPhoto.PhotoId = PhotoController.CreatePhoto(file, title);
                productionPhoto.Description = description;
                productionPhoto.Title = title;
                //Add photo to production
                Production production = db.Productions.Find(id);
                productionPhoto.Production = production;
                production.ProductionPhotos.Add(productionPhoto);
                //If production doesn't have default, set this to default
                int? defaultId = production.DefaultPhoto.PhotoId;
                if (production.DefaultPhoto == null || db.Photo.Find(defaultId).Title == "Photo Unavailable")
                {
                    production.DefaultPhoto = productionPhoto;
                }
            }
            db.SaveChanges();
            return RedirectToAction("Dashboard");
        }
```



### Bug Fix

When the default photo of a production was deleted from the site, it was meant to automatically select another photo connected to that production as the default.  This was not happening.  I researched to problem by finding the method and setting a break point and stepping through to watch what was happening.  I realized that the id of the photo was being changed before it was referenced for the resetting of the default photo.  I was able to fix the bug by simply rearranging the code.  



## Front End Stories

* [Add Button When List Empty](#Add-Button-When-List-Empty)
* [Display All User Donations](#Display-All-User-Donations)



### Add Button When List Empty

For this story the goal was to add a button to link to the User List page only when the Donor List was empty.  

```
<!-- Link to UserList Page, only displayed if donor list empty -->
  @if (!Model.Any())
  {
    <div class="donor-list-empty">
      <p>There are no recent Donors.  Would you like to go to the UserList page? </p>
      <a class="btn btn-main" href="/Admin/UserList"><i class="fa fa-hand-point-left fa-fw"></i>UserList Page</a>
    </div>
  }
  ```


### Display All User Donations

The purpose of this story was to show users of the site a list of their previous donations to the theater.  If they did not have any donations, instead there would be a link to the donations page.  

```
<!-- Donations section, displays list of past donations, if any -->
  @if (Model.Donations == null || Model.Donations.Count < 1)
  {
    <div class="row mt-5">
      <div class="col mx-auto text-center">
        <h4>Consider making a one-time donation to the Theatre.</h4>
        <a class="btn btn-main" href="@Url.Action("Create", "Donation")">Donate</a>
      </div>
    </div>
  }
  else
  {
    <div class="row mt-5">
      <div class="col mx-auto text-center">

        <h2>Your Past Donations:</h2>
      </div>
    </div>
    <div class="row justify-content-center" id="user-past-donations">
      <div class="col-auto">
        <table class="table table-responsive table-bordered">
          <thead>
            <tr>
              <th scope="col">Donation Time</th>
              <th scope="col">Amount</th>
            </tr>
          </thead>
          <tbody>
            @foreach (var donation in Model.Donations)
            {
              <tr>
                <td>
                  @donation.DonationTime
                </td>
                <td>
                  $@donation.Amount
                </td>
              </tr>
            }
          </tbody>
        </table>
      </div>
    </div>
  }
  <!-- End of section -->

```

This also required initializing a list of donations from the database in the view model.  

```
model.Donations = (from s in db.Donations
                    where s.Donor.Id == user.Id
                    select s).ToList();

```
[Page top](#live-project) | [Back End Stories](#back-end-stories) | [Front End Stories](#front-end-stories)
