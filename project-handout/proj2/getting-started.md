# Getting Started

## Logistics

This project is due **Thursday, 2/18/2021 at 11:59PM PST (GMT-8)**. It is worth 6% of your overall grade in the class. The workload for the project is designed to be completed solo, but this semester we're allowing students to work on this project with a partner if you want to. Feel free to search for a partner on [this Piazza thread](https://piazza.com/class/kjoxqrf1eq04mr?cid=5)!

## Prerequisites

You should watch the B+ Trees lectures before working on this project.

## Academic Integrity Policy

“_As a member of the UC Berkeley community, I act with honesty, integrity, and respect for others._” — UC Berkeley Honor Code

**Read through the academic integrity guidelines** [**here**](https://piazza.com/class/kjoxqrf1eq04mr?cid=8)**.** We will be running plagiarism detection software on every submission against our own database of this semester's submissions, past submissions, and publicly hosted implementations on platforms such as GitHub and GitLab, followed by a thorough manual review process. Plagiarism on any assignment will result in a [non-reportable warning](https://sa.berkeley.edu/student-code-of-conduct-section6) and a grade penalty based on the severity of the infraction.

As long as you follow the guidelines there isn't anything to worry about here. While we do rely on software to find possible cases of academic dishonesty every case is reviewed by multiple TAs who can filter out false positives.

## Fetching the released code

The GitHub Classroom link for this project is in the Project 2 release post on [Piazza](https://piazza.com/class/kjoxqrf1eq04mr). Once your private repo is set up clone the Project 2 skeleton code onto your local machine. You'll be working off of a fresh copy of the RookieDB skeleton instead of reusing the one from Project 0.

### Setting up your local development environment

If you're using IntelliJ you can follow the instructions [in Project 0](../proj0/getting-started.md#setting-up-your-local-development-environment) in to set up your local environment again. Once you have your environment set up you can head to the next section [Your Tasks](your-tasks.md) and begin working on the assignment.

## Adding a partner

Once you've found a partner fill out [**this form**](https://forms.gle/sJsPSCZaaeKgTJya9) so we know who you're working with. If you want to share code over GitHub you can follow the instructions [here](../../common/adding-a-partner-on-github.md).

## Debugging Issues with GitHub Classroom

Feel free to skip this section if you don't have any issues with GitHub Classroom. If you are having issues \(i.e. the page froze or some error message appeared\), first check if you have access to your repo at `https://github.com/berkeley-cs186-student/sp21-proj2-username`, replacing `username` with your GitHub username. If you have access to your repo and the starter code is there, then you can proceed as usual. If you have access to your repo but the starter code is not there, run the following commands in a terminal \(again replacing `username` with your GitHub username\):

```text
git clone https://github.com/berkeley-cs186/sp21-rookiedb sp21-proj2
cd sp21-proj2/
git remote remove origin
git remote add origin https://github.com/berkeley-cs186-student/sp21-proj2-username.git
git push -u origin master
```

Then, you can proceed as usual.

#### 404 Not Found

If you're getting a 404 not found page when trying to access your repo, make sure you've set up your repo using the GitHub Classroom link in the Project 2 release post on [Piazza](https://piazza.com/class/kjoxqrf1eq04mr).

If you don't have access to your repo at all after following these steps, feel free to contact the course staff on Piazza.

