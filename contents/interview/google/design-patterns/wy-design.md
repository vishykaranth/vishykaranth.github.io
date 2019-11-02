## Why Design is Critical to Software Development

### Introduction
- I recently had a discussion about the merits of design within the field of software development. My protagonist was of the opinion that design was not something that should be taught in the early stages of becoming a software developer, and that design was largely a matter of common sense anyway. Their opinion of design was that it was too time consuming to be of use in commercial applications. I will try to address this point throughout the course of this article.

- As a professional software engineer with over fifteen years worth of commercial experience, and having worked in both good and bad development environments, I know the benefits that come with good design.

- I don't intend to describe each of the various design methodologies as there are numerous books and articles already available on such subjects. I intend instead to describe why design is a crucial part of the software development lifecycle.

### Software is an Engineering Discipline
- I always like to use the analogy of a software engineer and civil engineer. Both should employ engineering concepts to achieve their objectives. What this means is that when designing a software application, you should borrow engineering elements from the field of civil engineering.

- Here is a definition of software engineering taken from Wikipedia:

- "Software engineering is the application of a systematic, disciplined, quantifiable approach to the development, operation, and maintenance of software, and the study of these approaches; that is, the application of engineering to software".
- When designing a suspension bridge to span a mile wide river, would you really want to just proceed without giving any thought to how you intended to build it. Just turn up one Monday morning, and start bolting steel girders together. The obvious answer is of course not, but this same level of diligence and engineering does not always seem to apply to software development, even it would seem from some of those working within the industry. It would be classed as negligent to build a suspension bridge without having a design in place first.

- The earlier you are taught design, the better. Imagine turning up to a course on car building, where the lecturer said "This week, I'd like you all to begin building your cars please, and next week I'll show you how to design them". I'm sure you would think there was something wrong with the way the course was being taught.

- Before you build something, anything, you should design it first.

### Software Design is Not a Trivial Process
- Whilst some elements of software design may be common sense, that does not necessarily imply that it is all common sense. In fact, it is fair to say that the design process can be difficult and demanding. It can take years to develop the experience, knowledge and skills to become expert at it.

- Design constraints including extensibility, reusability, tolerance, coupling, cohesion and reliability to name a few, must all be carefully considered. There will be many trade offs and compromises that must be considered as part of the design. To say that all of this is simply common sense trivialises the process.

- The design process also manages the complexity that is inherent when building a fully fledged software application. With so many considerations to take into account, the human brain can quickly become overwhelmed with information. This needs to be broken down into manageable chunks that a designer can understand and work with.

### Design is the Key to Communicating Your Intent
- If you don't have a design, how do you and your team know how you are going to build the solution? Design takes time to get right, and can be fraught with problems. I spent several years as an Analyst Programmer, where part of my time would be spent visiting customers to document their requirements. I would then work this into a document explaining as clearly as possible how I had interpreted their requirements. This would include (amongst other things) use-case diagrams using the formal notation of Unified Modeling Language (UML).

- I never got this right the first time. I would send this to the customer for their input, and they would invariably point out the omissions and inaccuracies, for me to then revise the requirements document. After several iterations, I would arrive at a set of requirements that the customer would be happy with (for now, but that's a whole other story).

- A similar process can and should be employed in the design process, taking the requirements document, and iteratively shaping this into a working design. The software design should form the key communication tool amongst the software team.

### What Do You Mean You Don't Have Time?
- I have heard many reasons as to why the design process is often skipped, the most common one being not having time. My reply to this is "you don't have time NOT to." Seriously, spending time getting the design right, just like spending time getting the requirements right, will always save you much more time over the longer term than it will cost you in the short term.

- To rectify a design fault can be costly, but to rectify a source code fault is costlier still. The later in the software development lifecycle you find a defect, the costlier it will be to fix. So one way of looking at the need for design is to reduce the costs of defects that will be found in the implementation and testing phases (finding a defect in the testing phase is costlier still).

- By spending time designing, you are actively trying to reduce the number of defects that will be found in the later stages of the software development lifecycle. You are also mitigating any risks by actively seeking out a working solution, rather than proceeding regardless, only to find later on that something is not possible, or must be compromised.

### Design Can Reduce the Risks and Costs to Your Software Project
- There are many benefits to incorporating design into your software development lifecycle. Here are a few of them:

- The proposed solution is visible to the team and all project stake holders, and so everyone has a clear idea of what is required and therefore accelerates communication.
You can find problems with the solution before you start the implementation stage, where re-development will be far more costly.
Maintaining the application will be far easier when you have a series of design documents to refer to. This makes handing the maintenance of the application over to another team or third party much easier.
I will leave you with one of my favourite quotes on the subject of software design by C.A.R Hoare.

- "There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult."
### Conclusion
- The design process is crucial to the success of any software development project. It forms the central communication document for the development team. It reduces the risks and costs to the project. It helps to manage the complexity that is inherent to the development lifecycle.