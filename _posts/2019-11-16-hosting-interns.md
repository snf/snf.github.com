---
layout: post
title: "My experience hosting interns at Microsoft"
image: "/public/data/sharedptr/kcachegrind.png"
description: ""
category: ""
tags: [interns, Rust, programming languages, open source, Microsoft]
---

### Introduction

I've been always focused on being the best possible at technical problem solving but as I grew up, I learned to appreciate other skills.

For over a year, I had been asking for interns to join the safer programming languages virtual team that I co-created with other people in Microsoft.
By the time that my organization (MSRC) has agreed to assign two interns, the v-team had grown to many more people than expected and the project was starting to show signs of success.

Last summer I hosted two undergrad interns in the Safe Systems Programming Languages (SSPL) v-team that I co-funded that you can read more about it [here]. It was my first experience managing interns and here I'll describe my experience.

I'm skipping the recruitment as it was managed by the HR department with the affiliated universities.

### What do you offer?

I dropped out of university and started working professionally at the age of 18 so never had the chance to take an internship. However, I remember when I joined my first company, I was super excited because I was going to work with the most experienced exploit writers in the field. I was already doing that in my free time just for the fun of doing it but that step meant that someone else valued my skills and that I would be able to develop them faster.

For the interns, I wanted to give them something that would make them as excited as I was when I first joined that company. It's very hard to guess what a person wants and what excites them but one of the big things is *working with new and exciting technology*.
In the SSPL v-team, we had an interesting set of challenges that we could give to the interns so I wasn't very worried about fullfilling that expectation.

However, it shouldn't be all about the exciting stuff but being able to learn to function in an organization, promoting their work, documenting and succeeding within a big team.

To avoid falling in the trap of **cheap labour**, I set a criteria for also what the interns were getting out of this summer position and getting:
- Used in production: enforcement of standards
- Open source: recognition and community contribution
- Usage of new technologies: learning
- Challenging project without a single solution: learning
- Involved with other people or teams: team work, mentors

The idea was that the assigned projects would have as many points as possible from this list.

### Interview

At MSRC, we usually do a round of 4 technical interviews and this one wasn't very different.

However, these were the first interns that we were hosting in the office and the recruitment process was led by HR. We've got some indications from HR on how to make the interviews. In addition to the technical questions, each interviewer had to ask 2 "soft skills" questions. Mines were related to adaptability.

I enjoyed this part because most of the time, the "soft skills" interview is at the mercy of one or more interviewers who are better at this so having a fixed set of questions to ask was an improvement.

<!-- These questions were in the line of:
- Favourite products that they use and how they would improve them
- The time they had an issue and how they managed to solve it
- Etc -->

<!-- My only problem here is that soft skills interviews are standard these days and people treat them like an extension of the technical interview. It's fairly easy to study for these questions and how to answer them "correctly". -->

For the technical interview, each engineer chose their questions. I usually like starting with a problem and dive deep into it. At that time, I was working on immutable data structures in Rust and the flow of the interview I decided to use was something like:
- Can you describe what an immutable data structure is?
- Where would you use one and why?
- Can you implement an immutable tree in the programming language of your preference?
- What if I want to ...
- Can you optimize it for ...?

This type of questions usually help me evaluate how a person reasons about problems. If the person is familiar with the topic, we can usually go deeper into it until I either run out of questions and change to something else or the interviewee is challenged.

After the interview, I met with the other interviewers and compared our results. In our case, it was a tough decision as we had two very motivated candidates who were unfamiliar with the technologies we wanted to use but excited to learn them. Luckily, we ended up getting both.

### The preparation

My checklist for preparing for each new intern was:

- Check their devices were working. I opted for leaving the Surface Books in the box so they could open and setup the computers. Then learned that unboxing enterprise devices is not as fun as unboxing gaming consoles
- Got into contact with them and asked if they needed any help moving into town or anything else as they were coming from London
- Ordered learning material for them
- Wrote the executive summary of their projects
- Prepared the rest of the team for their arrival

### First day

We had a one on one meeting where I introduced myself again and went through these topics:
- Questions on what they wanted to achieve during the internship and which skills they wanted to develop
- Project to be delivered
- Methodoly. Where and how to execute the projects
- Mentors. These are the people they can easily approach for guidance and feedback. [Andrew] and [Ryan] were a huge help there.
- Ask if they have questions that I could try answering
- Welcomed them to come to my desk if they needed anything

After that, I introduced them to each member of the team where each person explained their project. I welcomed the interns to get more information about the specific projects if they had interest in them. As online tasks go, I added them to the respective chat groups, mailing lists and team meetings.

### First week

I don't think this was any different than any other person starting a job. But with the addition that I was trying to make myself very available in case they were worried of asking questions.

- Setting up the machines, accesses and repositories. These tasks let them feel the enterprise experience at its finest :)
- Continued support and answering questions about their projects
- Approached them every morning and asked how things were going and if they needed any help.
- They studied the project and the technologies involved. In this case Rust plus the other things they were using it for

### Work

By the second week, the interns were working by themselves without requiring much support which was a great thing to see.

For the rest of the internship the two-way feedback consisted in tracking their projects, helping them set milestones and a weekly one on one meeting where we discussed the projects and personal ambitions.

The list I'd go through each meeting is:
- How are they doing in the company and in the town?
- How is the project going and how do they feel about it?
- Skills they want to develop
- Anything else that they would like to learn more about
- Colleagues' projects they wanted to learn more about

I continued inviting them to the team meetings where people discussed their progress so they could get familiar with other things than their own ones.

### Closing

When both interns finished at the end of summer, both had acheived their projects. I invited them to present them internally and publicly. Internally, through a presentation, and, publicly, through blogposts.

As I mentioned in the [What do we offer? section], the idea was that they would not only get the experience of the projects, but also to have something public to show to teachers and, possibly, to recruiters when they are in the hunt again.

You can read more about Alex's experience using Rust for a top secret project and Hadrian's COM library in the blogpost.

### Conclusion

It was a great experience but I attribute it to both interns being very energetic and 

Thanks for reading.

* Follow me on Twitter: [@snfernandez](https://twitter.com/snfernandez)
* Contact me at Gmail: sebanfernandez
* Secure is better: [GPG Key]({{ site.url }}/public/data/sebanfernandez_0xEB1C845F_pub.asc)
