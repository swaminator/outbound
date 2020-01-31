Prior to joining Amazon, I cofounded an EdTech startup called Classalyze (classroom analytics for teachers and administrators) that was incubated at the Pearson Learning Accelerator. I ran the startup as the CTO for three years, built the MVP before we were able to afford an engineering team, ran sales and operations once the product features were more mature, and grew the team to 10 full-time people until we eventually shut down three years later (which can be it’s own post!). Post-startup life, I joined AWS and now work on Amplify, which makes it easier for startups to ship products.

In case you’re not familiar with it, [AWS Amplify](https://aws.amazon.com/amplify/) offers fullstack (frontend+backend) developer tools - the command line interface (CLI) simplifies deploying backend resources such as authentication, APIs, databases, and file storage; the frontend libraries and UI components make integration with the backend seamless; and the Amplify Console provides a Git-based workflow for your team to deploy and host your web app. 

Given my background both as a Founder and at a big tech company, I wanted to share my experience and advice for turning your idea for an app into a working MVP. In Part 1, I’ll start by walking you through how to take an idea — in today’s case, an online poster business — from a concept to a real app. Part 2 (coming soon!), will be a hands-on tutorial for actually building the app. To build the app, we will be using AWS Amplify - not just because I work on it, but IMHO its truly the fastest way to build a mobile or web app on AWS and I wish it existed when I was a startup founder! It is currently used by startup teams at Hypertrack, Noom, Happybits and large enterprise teams at Volkswagen, BMW, Coca-Cola, and Airbnb. 

*If you’d like to follow along and start building your own MVP in real time, be sure to check out the [AWS Activate URL](https://console.aws.amazon.com/activate/home/#/apply/portfolio) and use the ORG ID: 0ap9q to get $3k in AWS credits. Make sure to use the same email you registered for Startup School with so you can get swiftly approved.*

Now that you have AWS credits, let’s walk through how to grow your idea to an MVP app.

**Step 1: Validate your idea without writing a single line of code**
It all starts with the idea. But while ideas are important, they are also the easiest part of any startup journey. The biggest mistake I made with my startup, was spending way too much time trying to get my initial idea to work. We love our first ideas but must be able to let our ideas grow and evolve as we progress. It is important to start with the problem your idea is solving. 

For my sample online poster business idea, my hypothesis problem is that *“it’s hard to to find a poster I want to pin up on my wall”*. Who is the target customer with the problem? For example, maybe it’s college students. If so, do college students actually have this problem? How does the solution you’re building improve their life? The best way to go about getting answers is by talking to students via email and in-person interviews.  If you do not find a problem/solution fit, iterate your idea further. In your 1:1 conversations, maybe there were other problems students alluded to (e.g. no time for cooking? not enough means to earn part-time income?).

If you do find problem/solution fit (Congratulations!), the next step is finding product/market fit — is the college student market large enough for your startup to be a financially viable undertaking. Spend as much time as you can in this stage **without building any tech**. Building technology is expensive and will take time, so figure out how you can *“fake it till you make it”*. For example, with an online poster business, the major hypothesis is that people are willing to buy posters online. You could validate the idea before building an online store by setting up an experiment using known services:

1. Create a Facebook group and upload poster images to it
2. Promote the Facebook group on campus with flyers or stalls
3. Accept payments via online services such as Venmo/PayPal 
4. Print the posters at a nearby printing shop 
5. Ship posters to the provided address. 

If you are able to get initial traction, your idea has product/market fit and it’s time to actually build the product. If you’re building a tech product (where the end deliverable is a web or mobile app), there are many considerations as a co-founder even before you get started building your product.

**Step 2: Unpack your idea**
It’s now time to unpack your idea into to a concrete set of user stories or requirements for your MVP.  A minimum viable product is by definition an imperfect product. As Michael Seibel said last week, do not attempt perfection. Ship something fast, get feedback, then iterate. The MVP exists to offer value to your early customers and gives you something to show potential investors. 

To define the scope of your MVP, you have to define the narrative for your product. If your problem statement is *“it’s hard to to find a poster I want to pin up on my wall”* then your MVP must address how this poster app makes finding the poster you want really easy (instead of simply referring to it as an online poster business).
Unpack this into the following user stories (note: you will ideally have 5 to 10 times the number of stories):

1. Customers can browse a catalog of posters
2. Customers can create an account to favorite posters and view order history
3. Administrators can create/update/delete posters
4. Customers can add/remove posters to their cart and order posters.
5. Customers can search for posters by keyword
6. Customers can upload their own images to print custom posters

Given most of the requirements are in place, the next step is to define a flow. Flows are a really fast way to ensure folks on your team are aligned on the expected user interaction. Ryan Singer, a Program Manager at Basecamp, has a [really good format for defining flows](https://signalvnoise.com/posts/1926-a-shorthand-for-designing-ui-flows), which I’ve copied below.

![img](https://s3.amazonaws.com/37assets/svn/323-flow-template.png)

I've created a sample one for the poster app idea. Once the flows are sketched out, you might choose to mock screens. Personally, I think this is the right time to start building your app.
![img](https://s3.amazonaws.com/37assets/svn/323-flow-template.png)

**Step 3: Pick a platform as a distribution channel (web or mobile or both)**
Your distribution channel is actually the most important decision. Are you building a web or mobile app or both? For example, a game might be better served on a mobile device, a productivity app might be better on a browser, while an e-commerce app would arguably be useful on both. This fundamental decision drives all your future technology decisions. For the poster app, I’d like for users to be able to make purchases on both the desktop and a mobile device. The fastest way to get to market would be to build a Progressive web app or a Single-page web app that can be rendered mobile-first on both phones and desktops. If you saw traction, then you could potentially invest further in native app experiences.

**Step 4: Define your tech stack**
 Once you’ve picked a platform it’s time to pick a technology stack. This is either a really complicated decision or a very simple one if you or your technical cofounder is already comfortable with a particular technology. If you are building a mobile app, do you choose Swift (iOS) or Java (Android), or something cross-platform like React Native? For a web app, the choices seem endless - do you build a single page app with frameworks such as React, Vue, or Angular, or do you build server-rendered apps with more traditional (e.g. Rails, PHP) or modern (NextJS, Nuxt) frameworks? On the backend do you manage your own virtual servers (e.g. EC2)  or go serverless (Lambda, AppSync, Fargate)?

There is no right decision, it’s just what you’re comfortable with. As a tech co-founder who also did sales, one principle I followed was to outsource as much complexity as I could. Some things you give up as a result is control, but I was okay with that as I wanted to focus on selling my product. In 2013, I chose Ruby on Rails because 1) great community support - gems (or 3rd party libs) like Devise, ActiveAdmin were well adopted 2) defacto choice for most startups at the time 3) effortless deployment with Heroku 4) database - I chose Postgres mainly because Postgres and Heroku worked really well together (and I had only ever worked with relational DBs) 5) free to start. 

If I were building my startup in 2020, I would choose React on the frontend and AWS Amplify for the serverless backend. The primary reasons for using Amplify: 1) really easy to get started 2) free to start and very cheap compared to Heroku even once you start acquiring customers 3) managed services such as authentication and serverless functions (no need to manage scaling servers as your traffic increases). 
