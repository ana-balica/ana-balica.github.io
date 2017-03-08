---
layout: post
title: "Code review checklist"
date: 2017-02-21
tags:
  - code
---

Writing code is hard, reviewing it, is equally hard. In my team every ticket is scanned by a second pair of eyes.

Why? Half-assed work can make a company look bad, lose money, lose clients, generate stress and extra work, thus a good code review is a must before going live.

Over the last couple of months, I've developed my own internal code review checklist. I use it both for reviewing for own finished code and my teammates code complete tickets. It's split up into 3 sections: code, automated testing and manual testing.

### Code

1. **Does it compile?** Look for obvious errors, most IDEs will spot them for you. Baffling that these kind of issues slip into code completes, but we are all humans and mistakes happen. Most common reasons: last minute untested change that surely won't break anything or forgot to push the fix.

1. **Can you spot any runtime errors** just by looking at code? Try to reproduce the exception.

1. **Is the feature/bugfix complete?** Does it tick off all the requirements? Is there anything missing or misbehaving?

1. **Is this a good solution?** Is this the best solution out of all those that have been considered? This is mostly related to system design and general approach to problem solving. The definition of good and best is context dependent.

1. **Does it introduce any security holes?** Definitely not something to be disregarded.

1. **Is the code efficient** in whatever metrics matter for your case?

1. **Is it logically correct?** Does it solve the problem? This is usually central to the code review, hence obvious, but I don't want to skip obvious steps.

1. **How does it affect the rest of the system?** We review code diffs with no or minimum context regarding the rest of the codebase. With diffs you can only see that much, thus how about start looking in other places? Think about all the other components that might depend on this change. Where any of the APIs violated?

1. **Is it within the scope?** Does the ticket touch any unrelated parts of the codebase or does side fixes? The more it gets cluttered, the more complicated it becomes to review, test and release. Stay on topic.

1. **Are there any code style issues?** Conventions _are_ important for the sake of codebase sanity. I know we all have strong opinions about all the things, but for your team and code well-being, please disagree and commit.

1. **Is the code readable?** Can someone else understand it? How long did it take you to figure it out? Do all namings make sense?

1. **Does it have good and enough documentation?** Not to be ignored or considered of least importance.


### Automated testing

1. **Does it need tests?** Sometimes a favicon update will do just fine without tests.

1. **Does it have tests?** Seriously, write tests.

1. **Are tests logically correct?** Tests can have bugs too. Review tests thoroughly.

1. **Are tests simple, easy to read and debug?** Forget about DRY, don't build a whole infrastructure to test your code (unless you are writing an isolated testing framework with a clear API that everyone else on the team is more or less familiar with), KISS (keep it simple, stupid). If something fails, it should be apparent what broke.

1. **Are tests testing the right thing?** Tests are not a checkbox to be ticked off, they are your safety net. Just the existence of tests or higher test coverage doesn't guarantee a bug-free life. Make sure to get a good insurance with a solid testbase.

1. **Are tests testing the edge cases?** Best case scenarios suck!


### Manual testing

1. **Can you run the code?** Don't skip that. Automated tests look good on paper (ehhhm, computer screen?), but running the project is still a good idea. Automated tests reduce manual testing to the minimum, but don't exclude it completely.

2. **Can you follow through several test scenarios?** Pick obvious and non-obvious ones: do they succeed?

3. **Is the rest of the system looking/working fine?** Links to _How does it affect the rest of the system?_ from Code section. Similar, but more straight-forward. Have a click-around and make sure things are fine.

4. **Can you break it?** Try to think of ways it might fail: feed it large sets of data, consider third-party systems downtime and how it can affect you, use an unusual email address ([did you know email address can have comments in-between tokens?](https://tools.ietf.org/html/rfc2822#page-47)). Don't worry, your users will still find ways to break it for you.

------

Feel free to suggest more checkpoints :)
