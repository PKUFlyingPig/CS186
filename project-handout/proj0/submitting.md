# Submitting the Assignment

This project is due on **Monday, 1/25/2021 at 11:59PM PST (GMT-8)**.

## Pushing changes to GitHub Classroom

To submit a project, navigate to the cloned repo in a terminal and stage the files for your submission using `git add`. For example, in this project you would run:

`git add src/main/java/edu/berkeley/cs186/database/databox/StringDataBox.java`

to stage your change to `StringDataBox.java`. Once your changes are staged commit them with `git commit -m "Put your own informative commit message here"`. Finally use`git push` to push all of your changes to the remote GitHub repository created by GitHub Classroom.

## Submitting to Gradescope

Once your changes are on GitHub go to the CS186 Gradescope and click on the project for which you want to submit your code. Select GitHub for the submission method \(if it hasn't been selected already\), and select the repository and branch with the code you want to upload and submit. If you have not done this before, then you will have to link your GitHub account to Gradescope using the "Connect to GitHub" button. If you are unable to find the appropriate repository, then you might need to go to [https://github.com/settings/applications](https://github.com/settings/applications), click Gradescope, and grant access to the `berkeley-cs186-student` organization.

Note that you are only allowed to modify certain files for each assignment, and changes to other files you are not allowed to modify will be discarded when we run tests.

You should make sure that all code you modify belongs to files with `TODO(proj0)` comments in them. A full list of files that you may modify for this project are as follows:

* `databox/StringDataBox.java`

Once you've submitted you should see a score of 1.0/1.0. If so, congratulations! You've finished your first assignment for CS 186.

### Submitting via upload <a id="submitting-via-upload"></a>

If your GitHub account has access to many repos, the Gradescope UI might time out while trying to load which repos you have available. If this is the case for you, you can submit your code directly using via upload. You can zip up your source code with `zip -r submission.zip src/` and submit that directly to the autograder.

## Grading

* 100% of your grade will be made up of one test released to you \(the test that we

  provided in `database.databox.TestWelcome`\)

* This project will be worth 0% of your overall grade, but failing to complete it may result in you being **administratively dropped from the class**

