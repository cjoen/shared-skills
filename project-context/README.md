Here are some skills that help setup strong project context - dense and efficient context docs mean less tokens wasted and your agents have a better guidance for how to operate and what to avoid

## Overview

the update-context skill is made to create high level documentation and mapping for your AI agents to use when navigating the codebase, giving them an efficient roadmap that minimizes token waste.  Sometimes this context is enough for smaller tasks.  In a brownfield project with more complex goals its important to build a clear picture of the target component or areas that you're working with.  

codebase-research solves that problem.  it maps out key functions, conventions and folder structure doing a deep dive into a specific corner of the project so its clear 1. where edits need to be made or 2. conventions for how new components should align with current standards

This article inspired these skills to make the process easier. I highly recommend reading it first. [Research-Plan-Implement](https://boristane.com/blog/how-i-use-claude-code/)

TL;DR - use these steps for targeted agentic coding - these skills make Step 1 really easy.
1. Research (/codebase-research) 
2. Plan (conversational level setting for the desired outcome) 
3. Review (Review agents plan and iterate until the plan is very details) 
4. Implement (The agent follows the plan)

Once everything is setup here is how I typically use it - for an example lets say I'm updating or adding a new component to the account -> user detail page
- pull new code and run `/update-context`
	- once your context has been setup this runs pretty quick
- run `/codebase-research user accoun detail page`
	- this runs some explore agents then compiles an output `account-detail-research.md`
	- once the `account-detail-research.md` doc is created start a new session
- In your first message reference the research.md file with the exact path and begin your planning session - this cuts down on unecessary cycles of redirecting the agents to the area you want and is a nice checkpoint.  
- You can also request a human-readable research doc that helps a ton with getting up to speed on new codebases or components

These steps give the agent precise context so you have the best input for your planning session.  
