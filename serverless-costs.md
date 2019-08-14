Maybe you're a product manager or program manager or delivery manager or otherwise accountable for software development and support budgets. Maybe your developers are talking about wanting to go "serverless" on some new work. There might be lots of chat about AWS and Azure and functions and lambdas and it just sounds like the latest shiny thing, bored programmers wanting to play with new toys.  Your first instinct might be that they just get on with their work, using the tried and true and trusted servers you already have in place, with their well-understood costs and support needs.

I'm here to suggest that you listen to the developers.

Recently we had to perform some upkeep on a few of our app-server VMs, after working mainly on serverless code for the last year or so[^cloud]. In doing so, I got to thinking about the costs — direct and indirect — of apps-deployed-to-a-server vs. functions-running-someplace.

In particular, I have in front of me two stages of the same data flow. One runs on an Linux server (an Azure VM, specifically), alongside one other major app hosted on the same server. The other (newer) stage is comprised of two different Azure Function apps.

The older app is written in Python (though it could be written in any language) and triggered by a cron job. Logs are archived to disk, the disk is backed up, it all works just fine, like this sort of app always has. It reaches out to a web API to pull down a lot of employee and scheduling data, normalizes it, archives it, stores it to a database, etc.

The newer app is written in JavaScript (again, it could be written in any language) and triggered by a timer. Logs live in Application Insights, everything is archived nicely, and it just works. It reads that database, populates and updates matching Active Directory records, and passes the information on to another function app that populates and updates another system through *its* web API.

The workloads are similar enough that we're not comparing apples to oranges. Maybe oranges to nectarines.

### Upkeep costs

Most of the time, there's very little maintenance or attention needed for either app. But when reorganizing and upgrading our Azure hosting infrastructure, we have to be aware of:

- making sure the Linux server is up-to-date, running a compatible version of Python, running the proper integration daemons, etc.
- keeping the machine up and running through the upgrade processes
- maintaining a continuous backup regimen against the correct disk

So any changes generally require a couple hours of work, often spent figuring out just what the current state of *this* particular server might be.

If we want to upgrade or change the infrastructure around the function apps, we just... do it. Worst case, we redeploy the function app to a new resource group, timers and app insights and all. It takes a few minutes. Or CI server actually does it all.

### Operational costs

The two main apps on that Linux server don't require *too* much heavy iron, but they do have some serious disk and memory needs when they're active. We have a fairly modest VM "size" applied, at a promo deal no less, and the cost is about \$430 per month (including the backups, everything being tied in and managed through our Azure subscription, etc.). Let's say half of that is going to this app: $215 per month.

The function apps — also backed up, tied in to Azure, etc. — scale out as needed, throwing more CPUs on the fire if the load warrants. Otherwise they're dormant. Monthly cost: $6.45 (the decimal point is right where it belongs).

Now, to be fair, the newer system is only processing a portion of the full dataset. When it takes on the full dataset next year, we expect the load to increase sixfold. So that might be something like $40. Again, the decimal is correctly placed.

### Development and Deployment

In both cases, we're writing and testing locally, using TDD and BDD tools along the way. No significant difference there.

But the deployment story is different: on the older system, there are firewalls to configure; in some cases, URLs to expose (along with the associated DNS updates), and so on.

We deployed the newer functions to Azure, and immediately had a known, consistent, and accessible URL for our functions, as well as matching development versions of the same. Minutes vs. hours, at the least.

**_I wonder if there is something to mention here about Azure providing a localized framework for testing the function app??_**

### Nothing to see here

None of this is particularly shocking — it's not news that on-demand billing can make a lot of sense for batchy, bursty applications. I'd just forgotten how striking the difference could be.

I'd done precious little serverless code a few years ago; the "old" way was second nature, well-understood, and seemed to me by *far* the most expedient way to get going. Habits can be changed, skills can be learned (and dropped), and what's obvious now isn't a permanent state. For all sorts of reasons, cost-related and otherwise, it would now take some work to convince me to waste time with a server for any new SaaS project. And it would probably take even more work to convince the folks who are actually responsible for the budget.


[^cloud]: Yes, I know that "serverless", like "the cloud", is another name for "someone else's computer". But it's the accepted name, so on we go.
