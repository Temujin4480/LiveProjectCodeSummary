# LiveProjectCodeSummary

My last two weeks of my time studying at the Tech Academy was spent working with a group of junior programmers like myself from all around the world, working to develop a MVC web application using C#, CSHTML, CSS and JavaScript.  We were connected via an Azure DevOps website that allowed us to communicate and use the web board to choose our stories and keep each other apprised of what tasks we were working on and what was completed.  The software we were developing was similar to a web board and was basically an app for employers at a company to create and distribute weekly schedules assigning employees to certain jobs.  We were working from a legacy codebase, so our job was to add features to the application and fix bugs.  

The project was a great experience for a number of reasons.  It let me see into the daily life of a software developer and what it takes to provide a quality product.  I got lots of practice merging, pushing and pulling code, and got some insight in to how careful a programmer has to be with version control when working on a group project.  I had a little bit of prior experience working with MVC apps, but those were smaller apps that contained basically one or two models, one or two views, and one controller.  This project was much more complex than what I was used to, having dozens of each of the MVC components, and studying the code to make sense of it really reinforced my understanding of the connection between models, controllers and views. Furthermore, through examining the code I also learned a lot of new front end techniques, the advantages to those, and ways to create web apps with built in HTML methods.  Finally, this project also provided me more reinforcement on how to connect apps with databases using the Entity Framework, and how the database tables are represented within the program.

I worked on two stories during my two weeks.  The first story was a back end story, while the second story was primarily front end.  Below are the descriptions of the stories I worked on including code snippets and screen shots.  

# **Back End Story**

My first story, Implement Job Other, had two parts.  The first part required me to get user notes about a job to save to our attached database and appear in the job menu.  Initially, when a user inputted a note about a job, nothing was returning to the menu page on the website.  

![Create without Notes](Images/Screenshot%20(12).png)

See, no saved notes.  After spending time getting myself acquainted with the code of the program, I found that the method in the Jobs controller to create a new job needed to bind the model JobOther, where the note property existed, with the model Jobs, which contained all the other properties of the job.  

        public ActionResult Create([Bind(Include = "JobIb,JobTitle,JobType,Active,Location,Manager")] Job job,
            [Bind(Include = "ShiftTimeId,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday,Default")] ShiftTime shiftTime,
            [Bind(Include = "JobOtherId,Note")]JobOther details)
        {
            shiftTime.Job = job;
            details.Job = job; 

            PopulateJobDropDowns(job);
            var LocationId = Request.Form["LocationSelector"].ToString();
            var ManagerId = Request.Form["ManagerSelector"].ToString();

            if (ModelState.IsValid)
            {
                job.Location = db.JobSites.Find(Int32.Parse(LocationId));
                job.Manager = db.Users.Find(ManagerId);
                db.Jobs.Add(job);

                db.JobOthers.Add(details);

                db.ShiftTime.Add(shiftTime);

                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(job);
        }

That did the trick, and notes were now appearing in newly created jobs.  

![Create With Notes](Images/Screenshot%20(18).png)

The second part of the story was to allow users to use an edit function to change the notes.  This required some changes to the edit method again in the Jobs controller.

        public ActionResult Edit([Bind(Include = "JobIb,JobTitle,JobType,Active,Location,Manager,Details")] Job modelJob)
        {
            Job job = db.Jobs.Find(modelJob.JobIb);
          
            JobOther jobOther = db.JobOthers.Find(modelJob.Details.JobOtherId);

            if (jobOther == null)
            {
                jobOther = new JobOther();
                jobOther.Job = job;
                jobOther.Note = modelJob.Details.Note;
                db.JobOthers.Add(jobOther); 
                db.SaveChanges();
                db.Entry(job.Details).State = EntityState.Modified;
            }
            else
            {
                job.Details.Note = modelJob.Details.Note;
                db.Entry(job.Details).State = EntityState.Modified;
            }

            if (job.Manager == null || job.Location == null)
            {
                PopulateJobDropDowns(job);
            }
            else
            {
                PopulateJobDropDowns(job, job.Location.JobSiteID, job.Manager.Id);
            }

            var LocationId = Request.Form["LocationSelector"].ToString();
            var ManagerId = Request.Form["ManagerSelector"].ToString();

            if (ModelState.IsValid)
            {
                job.Location = db.JobSites.Find(Int32.Parse(LocationId));
                job.Manager = db.Users.Find(ManagerId);
                db.Entry(job).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            
            return View(job);
        }
        
The challenging part of this task was that when a user edited a job, I was able to get the database to save the notes, however it was always returning a null value when it should have been returning a string.  This was a bit beyond my skills to figure out why, but with the help of the project director we were able to determine the issue causing this. We found the model state is determined to be valid at the time to form is passed to the controller, not at the time the property is checked. Therefore, we had to make changes to the Edit view to tell the controller the default value when it cannot find a value.  So, we added the follwoing line to the view: 

        @Html.HiddenFor(model => model.Details.JobOtherId, new { @Value = Model.JobIb.ToString() })

This made it so editing the notes was possible: 

![Edit With Notes](Images/Screenshot%20(19).png)

# **Front End Story**

In my second story I was tasked with fixing some action-link buttons for editing, deleting and showing details of a job schedule that were no longer working.  Since I was now much more familiar with the code, this was a much easier task than the first one.  I quickly noticed that the buttons weren't broken, they were actually missing in the code.  So, the first thing I did was add those buttons.  

         <table class="table">
        <tr>
            <th scope="col">
                @Html.ActionLink("Job Title", "Index", new { sortOrder = ViewBag.JobSortParm, currentFilter = ViewBag.CurrentFilter })
            </th>
            <th scope="col">
                @Html.ActionLink("Job Type", "Index", new { sortOrder = ViewBag.JobTypeSortParm, currentFilter = ViewBag.CurrentFilter })
            </th>
            <th scope="col">
                @Html.ActionLink("Person", "Index", new { sortOrder = ViewBag.PersonSortParm, currentFilter = ViewBag.CurrentFilter })
            </th>
            <th scope="col">
                @Html.ActionLink("Start Date", "Index", new { sortOrder = ViewBag.StartDateSortParm, currentFilter = ViewBag.CurrentFilter })
            </th>
            <th scope="col">
                @Html.ActionLink("End Date", "Index", new { sortOrder = ViewBag.EndSortParm, currentFilter = ViewBag.CurrentFilter })
            </th>
            <th scope="col"></th>
        </tr>
        @foreach (var Job in Model)
        {
            var number = Job.Value.Count() + 1;
            <tr>
                <td rowspan="@number">
                    @Html.DisplayFor(Model => Job.Key.JobTitle)
                </td>
                <td rowspan="@number">
                    @Html.DisplayFor(Model => Job.Key.JobType)
                </td>
            </tr>
            foreach (var schedule in Job.Value)
            {
                <tr>
                    <td>
                        @Html.DisplayFor(Model => schedule.Person.FullName)
                    </td>
                    <td>
                        @Html.DisplayFor(Model => schedule.StartDate)
                    </td>
                    <td>
                        @Html.DisplayFor(Model => schedule.EndDate)
                    </td>
                    <td>
                        @Html.AnchorButton(AnchorType.Edit, Url.Action("Edit", new { id = schedule.ScheduleId }))
                        @Html.AnchorButton(AnchorType.Details, Url.Action("Details", new { id = schedule.ScheduleId }))
                        @Html.AnchorButton(AnchorType.Delete, Url.Action("Delete", new { id = schedule.ScheduleId }))
                    </td>
                </tr>
            }
        }
        
        
I was confident this would work, but as the buttons were tied into the ScheduleId, I needed to create some schedule items to see if the buttons were visible once those schedule items were present.  However, when I tried to add some schedule items, they did not save.  

![Create Schedule](Images/Screenshot%20(17).png)

After playing around with the code for awhile, I found that in the schedule model there was a custom validation class that was included in the end date property of the schedule class.  By simply removing this it allowed schedule items to be created, and also edited and deleted.  It also did not affect the program to delete the end date property as I believe it's purpose was to create an error message if the job schedule began after the end date, but there was already some JavaScript code built in to the create view to give this error message.  

        [DisplayName("End Date")]
        [DisplayFormat(DataFormatString = @"{0:MM\/dd\/yyyy}", ApplyFormatInEditMode = true)]
        //Schedule Create function will not work with an end date typed with the following line of code:
        //[DateRange("Cannot enter an end date earlier than today or than the start date.")]
        public DateTime? EndDate { get; set; }
        
After commenting out the DateRange class I was able to create schedule items and see that the action buttons were working.  

![Create Schedule Verified](Images/Screenshot%20(20).png)
