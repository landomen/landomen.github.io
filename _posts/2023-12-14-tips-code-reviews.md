---
title: 10 Tips for Better Code Reviews
description: In this post, we will cover 10 tips on how to improve your code review process and increase developer velocity.
date: 2023-12-14 11:00:00 +0100
categories: [Programming, Software Engineering, Code Review]
tags: [programming, software engineering, code reviews, feedback]
image: /assets/img/posts/tips-code-reviews/cover.jpg
---


As software developers, we spend the majority of our time reading and reviewing code. Indirectly, when we want to make changes to existing code that someone else wrote, and directly when we perform code reviews.

Code  reviews are a part of every healthy engineering team. They are a great way for authors to get peer feedback and for reviewers to gain more knowledge of the system and the new changes.

It’s important to have a good code review culture to make them as impactful and as smooth as possible. In this post, we will  cover  some tips on how to improve your code review process and increase developer velocity.

> Note: these are just suggestions and do not cover every good practice you should follow. The process you use should depend on your team structure and company guidelines.



## 1. Keep pull requests small and focused

Pull requests should be small in size and should try to solve one specific thing. It’s easy to get carried away with refactoring while working on a new feature or a bug fix, but including all those changes in your pull request can make it bloated.

It’s easier to perform a quality code review on a smaller number of code changes than on a large pull request. Small  and focused pull requests require less time investment from the reviewer and also usually result in better quality reviews.

Large pull requests demand a bigger time investment and can result in reviewers skipping over changes to finish them faster, which can result in bugs and lower-quality code.

If you encounter a large pull request as a reviewer, it’s better to suggest to the author to split the pull request into smaller chunks than to do a low-quality review.

## 2. Involve the whole team

Code reviews should be done by every team member, regardless of their seniority. They are a great way to get familiar with the codebase and to learn and grow.

Try to avoid relying on one specific person, like a senior engineer or team lead, to review and approve code. Usually, the thinking is that they are most familiar with the codebase and will perform the best quality review. While that is partially true, it can result in longer review times, as they have other priorities and might have trouble finding the time to thoroughly review pull  requests. In turn, this can result in lesser-quality reviews.

## 3. Set expectations for review times

Pull request authors would like to get feedback as early as possible, but performing code reviews can be a big context switch for the reviewer. The team should set expectations on what is an acceptable amount of time to wait for feedback on pull requests. Try to agree on a reasonable time frame so that the time to merge is as low as possible.

One suggestion that often comes up is to start and end the work day by performing code reviews. That way, no pull request is blocked for more than half a day. But everyone is different, so figure out a schedule that works best for you and your team.

## 4. Don’t be afraid to request code changes

Some engineers, especially the ones who recently joined the company, are hesitant to ask questions during code review and to request changes on the basis that they are not familiar enough with the codebase to doubt the author. But asking questions is a great way to learn about the codebase. Don’t just hit approve, but think about the code and ask clarifying questions, use this opportunity to get yourself familiar with the part of the system you’re reviewing. It’s better to request code changes and to approve after getting answers than to approve low-quality code.

## 5. Responsibility should be on the author

The author of the pull request should be the one who is responsible for ensuring that the code they want to merge meets the quality requirements of the team. This means that they tested their changes, covered edge cases, wrote tests, thought about security concerns, and used appropriate technology and solutions as the rest of the codebase. The author should make the code review process as easy as possible for the reviewer, providing additional context and resources about the solution.

There should be a level of trust between the reviewer and author, that the author has verified their changes. It should not be required of the reviewer to run the code by themselves to verify that everything works. They can raise questions and suggest to the author to check some cases. However, having to double-check the author’s work usually means that there are some trust issues. Of course, this depends on the seniority of the author and reviewer, but even then the goal should be to make the engineers as independent as possible.

## 6. Provide as much context to the reviewer as possible

It should be clear to the reviewer what the pull request is about without having to see the code. A good description can provide the necessary context that the reviewer might be missing when reviewing pull requests outside of their domain (another team for example).

The description should include what is being changed and why, why it’s being done this way if multiple solutions were considered, if any known issues need to be addressed, or if some functionality will be addressed in later PRs to avoid unnecessary comments.

**Include links**  <br>
Consider including links to Jira tickets, technical documents, Slack discussions, or anything that offers additional context to the reviewer.

**Include images and videos** <br>
If working on a user-facing feature, it’s a good idea to also include images and/or videos of the changes and show the before and after state. That way it makes it easier for reviewers to visualize the changes and spot potential design or behavior issues that might not be apparent from only the code.

It can also serve as a good documentation of visual changes over time.

## 7. Use pull/merge request templates

If you use [GitHub](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository) or [GitLab](https://docs.gitlab.com/ee/user/project/description_templates.html) for your pull/merge requests, then you can easily set up so-called review templates. This is just a `.md` file that you add to your `.github/.gitlab` folder in your project.  When you create a new pull request, it will pre-fill the description with this template.

The template can include sections like descriptions, and links to Jira tickets, as well as a checklist for common checks that the author should review and confirm before opening the pull request.

Here is a simple example of a GitHub template:


<script src="https://gist.github.com/landomen/d9085f18f8cc3865cb0783ef562cb0d9.js"></script>




Check the official guides for more information:

-   GitHub:  [Creating a pull request template for your repository](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)
-   GitLab:  [Description templates](https://docs.gitlab.com/ee/user/project/description_templates.html)

## 8. Set up static code analysis

The code should be formatted to match the project coding style before the review is requested. That way a lot of time can be saved by avoiding nitpick comments about empty lines, formatting, and so on. The easiest way to achieve this is to set up a static code analysis and auto-formatting as part of your git or CI workflows.

## 9. Provide clear and actionable feedback

If while performing a code review, you encounter a different solution that you’re not quite sure of, don’t be judgemental, but politely ask the author why they decided to go with this solution and if they considered a different solution. Maybe there is something that they or you didn’t think of. If needed, schedule a call with them to discuss face-to-face.

Having a thread with 50 comments on a pull request arguing why something wasn’t done the way the reviewer wants it is of no help to anyone. Feedback should be clear, actionable, and directed towards the code and not the author. Include specific suggestions in your feedback and explain why you are requesting changes. The goal of code review should be to learn and maintain high code quality and not to waste time with personal preferences and opinions.

While it’s easier to find issues with the code you’re reviewing, try to also find the good things about the solution and provide positive feedback. It can be of great encouragement to less experienced or new team members.

## 10. Follow-through

After the initial review, the author should address the comments, make the changes to their code, and request another review, following all the best practices we described above.

It can be easy for the reviewer to simply approve the pull request at this point, but this can result in potential bugs after merging if there are issues with the new changes. It’s therefore important to verify the changes and request new ones if feedback is not addressed.

Once approved, the author should merge the pull request and update any tickets or documentation.

# Conclusion

Code reviews are a crucial part of any great engineering team as they serve both to increase the quality of the code and as a way to share knowledge among team members.

The above tips can serve as a good starting point for improving your code review process. They offer some general advice but don’t cover every good practice, so you should check additional resources and discuss what the needs are inside your team.

Let me know in the comments if you found this helpful and what best practices you follow in your team.

## Resources:

-   [https://github.com/kamranahmedse/code-reviews](https://github.com/kamranahmedse/code-reviews)  — great and exhaustive checklist covering different stages from the author and reviewer perspective