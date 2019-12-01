---
layout: lab
num: proj03
package: earthquakes
ready: true
desc: "Individual Lab Track Project, part 3"
assigned: 2019-11-30 17:00
due: 2019-12-05 23:59
github_org: "ucsb-cs56-f19"
org: "ucsb-cs56-f19"
gauchospace_url: "https://gauchospace.ucsb.edu/courses/mod/assign/view.php?id=TBD"
---

<div style="display:none" >
Look here for formatted version: http://ucsb-cs56.github.io/f19/lab/proj03
Look here for formatted version: http://ucsb-cs56.github.io/f19/labWIP/proj03
</div>


# This project is only for the individual lab track

This project is only for those on the "individual lab" track.  It counts towards the project portion of your grade (20% of your final grade, rather than the lab component of your grade.)

You may cooperate with one or more pair partners from your team to help in debugging and understanding the project, but each person should complete the work separately for themselves.

Late submissions will be accepted only up until grading is complete; submissions received after grading is complete will NOT be accepted for regrades.   To ensure that your submission is accepted, be sure that you complete it and submit on Gradescope before the published deadline.


# This is a Continuation of Proj01 and Proj02

In the final step, we'll add a database table for locations, and the ability to store favorate locations in that table

# Step by step instructions

Continue with your:
* Github repo: <tt>{{page.num}}-githubid</tt>
* Heroku App:  <tt>cs56-f19-{{page.num}}-<i>github</i></tt>


## Step 9: Add Locations table to your application

In this step, you'll add a second database table to your application, one that stores favoriate locations.  We'll add a button to each of the search results that allows us to store that search result into the database, along with the id of the user that stored it.   We'll then be able to list both all locations, plus the "favorite" locations of the currently logged in user.   For each of these, we'll be able to go right to an Earthquake search for that location.

Create a feature branch with an appropriate name, prefixed with your initials, and do your work on the feature branch (not on `master`)

The database table will store locations, and it will have these fields
* `id`, which every SQL database table generally has
* `placeId`, the id that is returned by the location search API (e.g. `198134367` for the city of Santa Barbara) 
  when returned by the search at <https://cs56-f19-proj02-pconrad.herokuapp.com/locations/results?location=Santa+Barbara>
* `name`, the name of a location (e.g. `UCSB`, `Isla Vista`, `LAX Airport`, `Golden Gate Bridge`)
* `latitude`, the latitude of that location
* `longitude`, the longitude of that location

To add a new database table to your application, take the following steps:

1. Add an `@Entity` class.  Call this file `Location.java`, and add it under the `hello/entities` directory, 
   just as you did for `AppUser` in the lab07 series.    
   * Note that the `App` prefix on `AppUser` was just a hack
     because `user` is a reserved word in Postgres SQL.  Normally we would have just called that class `User.java`
   * So your class is just `Location.java` not `AppLocation.java`.
2. Add a `@Repository` class.  Call this file `LocationRepository.java`, and add it under the `hello/repositories` directory.
   * Model it after the `UserRepository` class, but instead of a collection of `AppUser` objects, it's a colleciton of `Location` entity objects.
   * You won't have a method called `findByUid`, since `uid` is not a member of `Location`. 
   * Instead, you can either:
     * define a method called `findByPlaceId` with appropriate return type, parameter type and parameter name.
     * Leave this out altogether for now, since you won't even need it unless you are planning to do the extra credit part of the project.
   (For background: in general, you can define `findByAbcd` methods for any field `abcd` that is that is part of the `@Entity` that the repo represents as part of the `interface` for the repo, and the Spring framework will automatically generate code for the class that implements the interface.    The generated code will do the correct database query.)
   
   It's probably a good idea to do a `mvn clean compile` to make sure everything still compiles before going on to the next item in the list, as well as a `mvn test`.     
3. Add a `@Controller` class called `LocationsController.java` under `hello/controllers`.  
   * Initially, just start with an endpoint to list the locations, similar 
     to the `UsersController.java`.  
   * This endpoint should be similar to the one that lists all of the users in `UsersController`.
4. Add a view in a directory `src/main/resources/templates/locations` in a file `index.html`.  Put in an HTML table that
   has appropriate headers, and that will list each location in the locations table.
5. Add an item to the navigation menu that directs the user to the endpoint that lists locations.   The link in the menu
   should say "Favorites", but it should link to `locations/index`.   Note that at present,
   since there are no locations in your database, you'll have an empty list. But at least you should get that, 
   and not an error.  We'll add code to put stuff in this table in a later step.

Make sure that the code you've added so far works, and that the test you added passes

Then merge that branch into master.  Deploy the master branch on Heroku, and make sure the code works on Heroku as well.


## Step 10: Add an "Add to Favorites" button on locations results page
 
In this step, we'll add an "add to favorites" button on each location result that comes up on the location list.

This button will be in a form that uses a "post" request.   We should use a "post" request because this request will
change the state on the server by adding a new record to the database.

Create a feature branch off of master for this step with an appropriate name.

1. We'll start by creating the endpoint in the `LocationsController` class.
   * The endpoint will look like this:

     ```java
         @PostMapping("/locations/add")
         public String add(Location location, Model model) {
           locationRepository.save(location);
           model.addAttribute("locations", locationsRepository.findAll());
           return "locations/index";
         }
     ```
   * Note that this requires a variable `locationRepository`.  You'll put this into the 
     `LocationsController` in a way similar to the `userRepository` is declared inside the 
     `UsersController`.    It is declared as a private variable, and then injected into the 
     constructor.   You should code this in a similar way.

   * The code here takes a parameter `location` of type `Location` as a parameter.  That will get passed in from the `th:object` 
     attribute passed in in the form we create in the next item below.  

     The code `locationRepository.save(location)` adds that location to the database.
     
     Then `model.addAttribute("locations", locationsRepository.findAll());` stores an attribute `locations` in the `model`, which
     we can then iterate over.   We then direct the user to the list of locations, using the same view we already created in Step&nbsp;9.
     
2. Next, we'll add the button that invokes the `/locations/add` endpoint.  That requires us to build a form that contains
   "hidden" fields that contain the values we want to add to the database.
   
   We'll want an instance of this form inside each row of the table of location results that gets returned by the `/locations/results` 
   view.
   
   So, inside `src/main/resources/template/locations/results.html` find the part that creates each row in the table of results.
   
   The row will start with `<tr th:each="p : ${places}">` and end with the closing tag `</tr>`.
   
   You'll need to add an additional `<td></td>` element into this row at the end.  The `<form>` element will go inside this.  But for now, just add this as `<td>Add</td>`.  We'll replace that word `Add` with the form at a later step. 
   
   To give this column in the table a heading, 
   you'll want to add `<th>Add to favorites</th>` element into the matching header row at the top. 

   Do a location search, and see whether you the table of locations has an `Add to favorites` column with the word `Add` in every
   item in that column.   

   If so, do a commmit, and we'll add the `<form>` in the next step.
   
3. Now replace the word `Add` with a form element that looks like this.  You may want to first separate the `<td>` and `</td>` tags
   and put them on separate lines so that you can indent the `<form>` element inside:

   ```html
   <form action="#" th:action="@{/locations/add}" th:object="${location}" method="post">
       <input type="submit" class="btn btn-primary" value="Add" />
   </form>
   ```
 
   This creates a form, but it is missing something important: the data that goes into the new object.  In this instance, that data isn't going to be typed in by the user, but instead it comes from the row of data.   To get it into the form we used what are known as "hidden fields" in HTML.  Each of those will look like this one.  

   ```html
     <input type="hidden" th:field="*{place_id}" th:value="${p.place_id}" />
   ```
   
   You'll need a line like this for each of the fields `place_id`, `name`, `latitude`, `longitude`.    
   Place each of these between the `<form ... >` open tag, and the existing `<input type=submit... >` element.   
   Take note that the `th:field` values need to match the field names of the `Location` class (the `@Entity` class), 
   while the field names in the `th:value` expressions need to match those coming from the `p` variable, which is 
   an instance of the `LocSearch` class.   Be mindful of the difference between `lat` and `latitude`, for example.

   When you have made these changes, you should see a button with the word `Add` on it instead of simply the word `Add` 
   each time you do a locations search.  Clicking any of these buttons should lead you to the index pages for `Locations`,
   where you'll see that the location has been added to the database.
   
Test these changes.   If/when they work, do a commit. and then a pull request.  Merge that pull request into master.
 
## Step 11: Add a link from the locations index page to be able to list earthquakes.
 
You'll now see that while the `locations/results` page has a link `Get Earthquakes` in every row, the `locations/index` page does not.
 
Your task in this step is simple: add that feature to every row of the `locations/index` page as well.  
 
I won't provide detailed instructions; by now, you should have enough information to know how to proceed.
 
Make a feature branch and implement the feature. You may divide this among multiple commits, or do it in a single commit, as you see fit.   Test, and then do a pull request, and merge that pull request into master.

## Step 12: Add a "delete" button for each row of `locations/index`

We now have a way to add things to our favorites (the "Add" button on `locations/results`) but we don't have a way to delete
them.  In this step, we'll add that.

Create a feature branch for the delete button.

1.  First make a controller endpoint for deleting locations.  It will look like this:

    ```
    @DeleteMapping("/locations/delete/{id}")
    public String delete(@PathVariable("id") long id, Model model) {
        Location location = locationRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Invalid courseoffering Id:" + id));
        locationRepository.delete(location);
        model.addAttribute("locations", locationsRepository.findAll());
        return "locations/index";
    }
    ```
   
    This will allow us to create a form that will allow us to delete a particular location by specifying its id right in the URL.
    You'll need an import for the `@DeleteMapping` annoation (exactly what that is is left as an exercise to figure out.)

    Add in this code, and makes sure that your app still compiles and runs.  If so, that's worthy of a commit.

2.  The next step is to add a column to the table on the `locations/results` page with the column heading "Delete".
3.  Now, add an extra `<td></td>` element with  the placeholder word `Delete` inside.  We'll replace this with a form element later.
    
    If you can see the placeholder when loading the `locations/results` page, that's worthy of another commit.

3.  Now add a form element that looks like this.  
    ```html
    <form action="#" th:action="locations/delete/${p.id}" method="delete">   
       <input type="submit" class="btn btn-danger" value="Delete" />
    </form>
    ```
    
    Note that we used `btn-danger` as one of the CSS `class` values we added to the `input` element.  In the Bootstrap framework,
    this makes the button appear in a different color, one associated with delete actions.   (For more info, see [this reference](https://getbootstrap.com/docs/4.0/components/buttons/).)
    
    Each of these buttons is associated with deleting a different element from the database by virtue of the `${p.id}` part of the
    `th:action` attribute, which specifies the URL to which the button will direct the application.
    
    Try adding and deleting some locations to see if this code works properly.   Note that at present, it is possible to add
    duplicate locations (e.g. adding the city of Santa Barbara multiple times), however these duplicates will be distinguished by
    having different `id` values, even if they have the same `place_id` value.
    
    An additional feature that could be added would be the ability to automatically check for duplicates before adding, and 
    not adding a place that already exists in the database.  However, we'll not require that as part of this project. Once your
    add and delete buttons work properly, we are done!
    
# Final Steps

## Final Step 0: A few things to check

1. Look over the staff's working version here:

   * <https://cs56-f19-proj03-pconrad.herokuapp.com/>
   
   Compare it to your working version on Heroku.  If you see differences, try to determine which of these is true:
   * Is is a minor difference that doesn't matter?
   * Did the staff miss part of the instructions?
   * Did you miss part of the instructions?
   
   If you aren't sure, ask questions on Slack.

## Final Step 1: Check that your code is all on master and Heroku

* Have you pushed all changes to your last feature branch?
* Have you done a final pull request?
* Have you accepted that pull request?
* Have you deployed your master branch to Heroku?
* Do all the parts of your application work? Can you login/logout, and access all pages?

## Final Step 2: Update your javadoc and jacoco report

To update your javadoc and jacoco report, do this:

```
mvn clean
mvn javadoc:javadoc
mvn javadoc:test-javadoc
mvn test
mvn jacoco:report
mvn site
mvn site:deploy
git add docs
git commit -m "xx - update javadoc and jacoco report"
git push origin master
```

## Final Step 3: Check your README.md

Check that your README.md has links to
* your GitHub pages webpage, and that the webpage is published.
* your app running on Heroku
* your repos Travis-CI status

## Final Step 4: Submit on Gauchospace

Then, finally visit <{{page.gauchospace_url}}> and make a submission.

In the text area, enter something like this, substituting your repo name and your Heroku app name:

<div style="font-family:monospace;">
repo name: https://github.com/chrislee123/spring-boot-minimal-webapp<br>
on heroku: https://cs56-{{site.qxx}}-{{page.num}}-chrislee123.herokuapp.com<br>
</div>

Then, **and this is super important**, please make both of those URLs **clickable urls**.

The instructions for doing so are here: <https://ucsb-cs56.github.io/topics/gauchospace_clickable_urls/>


# Grading Rubric:

TBA






    