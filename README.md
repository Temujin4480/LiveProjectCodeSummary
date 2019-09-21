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
        
The hard part of this task was that I was able to get the database to save an edit of the notes, however it was always showing a null value.  This was a bit beyond my skills to figure out why, but with the help of the project director we were able to determine the  issue was that the Model State is determined to be Valid at the time to form is passed to the controller, not at the time the property is checked. So, even though I was making changes to make the object valid, that didn't change the valid property. If there is no value, you need to pass it a default value, and since the JobIb is the same value as the JobOtherId, you tell it that is the default value. Therefore, the code in the edit cshtml page had to look as so:  
@Html.HiddenFor(model => model.Details.JobOtherId, new { @Value = Model.JobIb.ToString() })
