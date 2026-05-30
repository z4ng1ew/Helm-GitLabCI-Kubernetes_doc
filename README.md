# Final Project

💡 [Tap here](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624) **to leave your feedback on the project**. It's anonymous and will help our team make your educational experience better. We recommend completing the survey immediately after the project.

## Contents

1. [Chapter I](#chapter-i) 
2. [Chapter II](#chapter-ii) \
   2.1. [CI and CD setup](#part-1-ci-and-cd-setup)

## Instructions

How to learn at “School 21”:

- Here, you’ll find a unique learning experience with a lot of freedom. You’re given a task and left to find your own way to solve it, using whatever resources work best for you — whether that’s the Internet or AI tools like GigaChat. Just be mindful of information quality: verify, think critically, analyze, and compare.
- Peer-to-peer (P2P) learning is the exchange of knowledge and experience with peers, where everyone acts as both mentor and student. This approach allows you to gain a deeper understanding of the material by learning from one another.
- Feel free to ask for help: around you are peers who are also navigating this path for the first time. Share your own experience and ideas with others.  Join Rocket.Chat to stay updated with the latest community announcements. 
- Your learning is meaningless if you just copy someone else’s solutions. When receiving help from others, always make sure you fully understand the “why”, “how”, and “purpose” behind the solution. Don’t be afraid to make mistakes. 
- Does the task seem impossible? Take a break, get some fresh air and clear your mind — this has helped many people. Maybe after that, the solution will come to you naturally.
- The learning process is just as important as the result. It’s not just about completing the task — it’s about understanding HOW to solve it. 

How to work with the project:

- Before starting, clone the project from GitLab into a repository with the same name.
- All files should be created inside the _src/_ folder of the cloned repository.
- After cloning the project, create a _develop_ branch and do all your development there. Then, push the _develop_ branch to GitLab.
- Your directory should not contain any files other than those specified in the assignments.

## Chapter I

Now it's time to apply all the knowledge you've gained in this course to a final project dedicated to the full cycle of applying DevOps practices to an application. For this task, find a microservice application and walk it through all the stages, from containerization to pipeline setup on a working GitLab repository.

## Chapter II

The final product must be a report containing detailed descriptions of how each point was implemented, accompanied by screenshots. Prepare the report as a Markdown file in the `src` directory named `REPORT.MD`.

## Part 1. CI and CD Setup

### Task:

1. Find an open-source project on GitHub or GitLab consisting of at least three microservices, one service with a database, and one frontend.

2. Write Dockerfiles for the selected project, if necessary.

3. Write unit tests for microservices with application business logic, if necessary.

4. Develop a Helm chart for deploying the selected application.

5. Develop a pipeline for deploying the application consisting of four stages:
   - build,
   - test,
   - staging,
   - prod.

6. Set up CI/CD to deploy the application using the GitLab Runner in Kubernetes.