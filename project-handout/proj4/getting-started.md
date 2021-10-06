# Getting Started

## Logistics

This project is worth 8% of your overall grade in the class.

* Part 1 is due **Monday, 3/29/2021 at 11:59PM PDT (GMT-7)** and will be worth 20% of your score. Your score will be determined by public tests only.
* Part 2 is due **Friday, 4/9/2021 at 11:59PM PDT (GMT-7)** and will be worth the remaining 80% of your score. We'll be running the public tests for Part 2 and all hidden tests for both Part 1 and Part 2 on this submission.

The workload for the project is designed to be completed solo, but this semester we're allowing students to work on this project with a partner if you want to. Your partner does not have to be the same as the one you had for previous assignments. Feel free to search for a partner on [this Piazza thread](https://piazza.com/class/kjoxqrf1eq04mr?cid=5)!

## Prerequisites

You should watch both Transactions & Concurrency lectures before beginning this project.

## Additional Resources

Debugging walkthrough video for project 4 can be found [here](https://drive.google.com/drive/folders/1UnpcSU-rG9VAHfsD5WXO8CfFuzEDbwH_?usp=sharing).

## Fetching the released code

The GitHub Classroom link for this project is in the Project 4 release post on [Piazza](https://piazza.com/class/kjoxqrf1eq04mr). Once your private repo is set up clone the Project 4 skeleton code onto your local machine.

### Setting up your local development environment

If you're using IntelliJ you can follow the instructions [in Project 0](../proj0/getting-started.md#setting-up-your-local-development-environment) in to set up your local environment again. Once you have your environment set up you can head to the next section [Part 0](skeleton-code.md) and begin working on the assignment.

## Adding a partner

Once you've found a partner make sure you fill out the details in the form on the piazza release post. If you want to share code over GitHub you can follow the instructions [here](../../common/adding-a-partner-on-github.md).

## Debugging Issues with GitHub Classroom

Feel free to skip this section if you don't have any issues with GitHub Classroom. If you are having issues \(i.e. the page froze or some error message appeared\), first check if you have access to your repo at `https://github.com/berkeley-cs186-student/sp21-proj4-username`, replacing `username` with your GitHub username. If you have access to your repo and the starter code is there, then you can proceed as usual. If you have access to your repo but the starter code is not there, run the following commands in a terminal \(again replacing `username` with your GitHub username\):

```text
git clone https://github.com/berkeley-cs186/sp21-rookiedb sp21-proj4
cd sp21-proj4/
git remote remove origin
git remote add origin https://github.com/berkeley-cs186-student/sp21-proj4-username.git
git push -u origin master
```

Then, you can proceed as usual.

#### 404 Not Found

If you're getting a 404 not found page when trying to access your repo, make sure you've set up your repo using the GitHub Classroom link in the Project 4 release post on [Piazza](https://piazza.com/class/kjoxqrf1eq04mr).

If you don't have access to your repo at all after following these steps, feel free to contact the course staff on Piazza.

