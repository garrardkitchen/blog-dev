---
title: "The Grass Is Rarely Greener"
date: 2022-01-30T18:27:54Z
tags: [azure, aws, empty promises, building relationship, willing, attrition, managed service]
---

I recently had a conversation that stirred up some _surpressed_ memories.  The conversation was related to moving to a different cloud vendor.  That journey didn't work out so well for me and the company I was working for at the time.  Sadly, that company stopped trading and I don't think it's that much of a leap to link that journey to the demise of that company.  Hopefully, now the title of this post is starting to make more sense?

Several years back, the company that I was working for at the time shifted to AWS, away from Azure.  It was heavily suggested - I was present during that conversation - that it would be in our interest if we "jumped ship" to _their_ platform.  "Their" being "AWS".  Long story short, sadly it was not.  In fact, it turned out to be a huge waste of time.  IMO, they should never have suggested this.  There were other suggestions made that _they_ really should not have made; sadly we followed these too.  In retrospect, it was totally unjustifiable not to mention unfair for them to suggest any of this for a company of that size [small], especially knowing how long all of this would take to do.  I even gained 2 AWS certifications to help best advise the company on how to implement/integrate with the AWS Platform and backing services.

If you are thinking, "You are grown-ups with years of experience under your belt yeah, so you didn't have to go with what they said did you?".  Indeed, but it was AWS and these people we were talking to were big hitters and as such, the "carrot" was big. Who wouldn't right?!

I will return to this in a later post but for now I will call out, IMO, several important and possibly overlooked considerations when contemplating a move to a different Cloud Provider.

# Considerations when moving to another Cloud Provider

Here are a few nuggets that will hopefully resonate:

1 - I don't know of any business that will allow their "technical people" to have their way and purely focus on tech debt or similar, without developing features (or improvements) over a prolonged amount of time.  If part funded with VC money, I'm sure they won't be too happy either - further delay on their return, and for what?  With all the best intentions in the world, migrations are time-consuming (not to mention a scheduling nightmare), and this time is exponential when dealing with new or unfamiliar tech.  Tech will also present problems, regardless of where it is.

2 - Cloud vendors offer essentially the same backing services - orchestration, serverless compute, VM compute, db, caching, temporal decoupling thru queues, and the list goes on... .  They may make it easier or more attractive for particular language techs and ecosystems, but essentially, it's the same offerings plus patterns (EIP but in the Cloud) but elsewhere. Not to mention they all have their own version of the 5 pillars of the "Well-Architecture" framework.

3 - As an extension of (2) above, there's training to consider.  Also, new problems to solve, and migration paths to figure out.  Not to mention RBAC and ownership.  And don't forget DevOps, possibly made slightly easier if you've opted to a non-native IaC approach like TF (Terraform).  And don't forget automation, runbooks, alerting, 3 pillars of observation (incl. distributed tracing). And don't forget... .

4 - If there's something that your current cloud vendor doesn't offer and the need is  absolutely justifiable, you can also look for a SaaS alternative.  When you boil it down, all you are _really_ interested in are specific use-cases/features, integration options (SDKs) and it being a "managed service" as to avoid requiring additional Engineers to manage it and perform associated activities to ensure it's HA, patched and secure.  Even if it's a service only offered by a different cloud vendor, there are light-touch integration options - for example, AWS Lambda and AWS Kinesis Data Streaming.  This is an approach we've had to take.

5 - It's always going to take longer than you think.  Fact.  We tend to think "happy path".

6 - Now, this next one is a biggy ... if you are moving to new tech, whether it be a new cloud vendor but more appropriately shifting to a new server-side language (eg from c# to go), you are going to lose between 20-40% of those Engineers who's core language is c#.  I've not observed the same levels of attrition on a shift to a different client-side tech.  More often, you'll have several client-side techs in play. Maybe some core concepts of newly opted JS framework have changed, but essentially the language tech hasn't (JS/TS).  Well, that's if you're not shifting to Blazor (WebAssembly).  One major deficit of when your Engineers leave is that so does their domain knowledge 😱.  Again, speaking from a position of experience, it's sad to see people go.  Morale takes a hit.  And when Engineers leave, their roles need replacing too. Recruitment ensues, which requires planning plus patience, often the cause of disruptions and then it takes other Engineers' time for onboarding and orientation.

7 - If cost is a consideration or motivation for the move then what are you doing re: the Cost Optimisation tenet of your provider's flavour of a "Well-Architected" Framework?  More important than this but less obvious, what is the Cloud Service Provider (CSP) or organisation that is the conduit between you and the Cloud Vendor doing to help?  In my experience, they only get involved if you need help with an issue, or if there's a zero-day exploit you need to be made away of or other type of issue involving one of your backing services.  Anything difficult, then they tend to hand off to the Cloud Vendor themselves, but hey, at least they're on the same support call with you! 👀.  They generally don't get in your face either if you're spending a futune!  You may get the occasional email from them letting you know that you are very important to them.  IMO, they need to be continually proving their worth to you [your organisation] and if they're not, engage a different CSP  or employ certified professional(s) who can evaluate your architecture, and your DevOps processes as well as costs.  Essentially, you need certified/experienced Engineers to help to make real tangible contributions and improvements.

8 - If technical issues are plaguing you, then likely they'll manifest themselves also on a different cloud provider.  So, if you've invested heavily with a particular tech, or approach, and this is too costly, too risky to the business due to instability, first understand why first before jumping ship.  You never know, you might be using it inefficiently or incorrectly.  Take AKS for instance.  It's hugely involved.  If you get any part of that wrong, it's a risk to your business.

There will be more I'm sure, but for the sake of getting closure on this topic and to put these emotions back into _that_ box, I will leave it there.  

# Recommendations

In summary, my advice is this; evaluate your current platform to identify (in no particular order):

- Cost savings (eg there could be better bin packing or scaling options or up to 70% reduction on "compute" by using Reserved Instances)
- Configuration changes to improve stability, minimize failure on upgrading your dependencies, and self-healing
- Improvements on patterns and practices used by your Engineers (incl. low code options, delegate failure edge/corner cases to your architecture, consistency of solution - don't solve the same problem multiple ways!, only measure what you can action)
- Automation (eg runbook when metric exceeded, cron jobs)
- What is your prominent tech?  For example, if your ecosystem is predominately .NET then Azure is a justifiable Cloud Platform.  You simply won't get the same depth/scope of features/integrations if you go with a different Cloud Provider.  Think about it...would you offer up all your crown jewels to a competing provider? I rest my case!

I do hope this post has been informative.  If I help one person avoid this _more often than not wasted effort_, then this post will have been worth it.  Changes of this magnitude are laden with risk and one thing companies are adverse to is risk.
