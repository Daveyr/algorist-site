---
title: Antifragile book notes Part 1
author: ''
date: '2021-10-11'
slug: antifragile-book-notes-part-1
categories:
  - Blog
tags:
  - Project Management
  - Organisational
  - Uncertainty
subtitle: ''
summary: ''
authors: []
lastmod: '2021-10-11T14:46:19+01:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

<img src="/img/stoic_wide.jpeg" alt="" width="100%"/>

I think my work colleagues and friends are now bored of me raving about the book _Antifragile_ by Nassim Taleb. My reading style is to read very slowly, mulling over each sentence again and again. When a book is this dense I also tend to make notes. Here they are below, from the point of view of a consultant and a numerical modeller. Some examples are taken straight out of the book whilst others are ones I have personally encountered that illustrate concepts the book raises.

## Consultants don't come out of this looking good
Back in the mists of time, everyone had skin in the game. In an agrarian economy, if you took a risk planting an unusual crop, or grew lax with the safety of your flock, it was your harvest at risk and only yourself to blame. In the modern world there has been a subtle and almost unnoticed risk transfer, where some jobs tend to be exposed to upside whilst transferring the downside risk to others. The archetype of this antifragility of the individual and fragility to society is a bank executive, where the position can enjoy bonuses right up to the moment their business (which they do not own) defaults and costs tax payers a large amount of money to bail out. It is an agency problem, where the agent (bank executive) has misaligned incentives to that of the principle (the bank).

Other more subtle examples include any advisory service where the payment is not linked to the outcome from implementing the advice; pundits and forecasters whose salaries are not linked to the accuracy of their pronouncements are also in this category. Consultants rarely have skin in the game and also commonly fall into this category. So what, if anything, can we do? We need to remove the agency problem by realigning incentives. Structuring the contract so that payment is conditional or proportional to benefit would encourage consultants to model realistically and build in the ability to measure performance. Of course this requires buy-in from clients themselves to implement the advice and track its performance. "Sticky" solutions like subscription based services also tend to align incentives: if the service it provides is rubbish then a customer can elect to leave the following month.

Do we need to go back to what they did in Roman times to keep engineers and builders honest? Back then, if a house that you built fell down and killed someone then you were executed as punishment. No wonder you see lots of Roman buildings and foundations that have survived to the present day.

![Would the colloseum have survived if the builders were not sufficiently motivated?](/img/colloseum.jpeg)

There are some roles with more than just skin in the game. Taleb refers to these brave individuals as having _soul_ in the game. These days its easy to wonder, where have all the heroes gone? But examples abound: bearers of unwelcome news (prophets in old times), whistle blowers, investigative journalists, opposition leaders in countries under totalitarian rule, and, especially in times of pandemic, medical staff and care home workers. These are the true key workers.

## Optimisation fragilises
Optimisation makes things fragile. An excellent example in the book is the traffic flow problem. It displays convexity, as adding one more car to the road network produces a negligible addition to the average journey time until you reach a critical point where the marginal cost of journey time from one more car becomes exponential. Think about it, you have often taken twice as long on a car or plane journey but how often have you taken _half_ the time you estimated?

![Traffic congestion: a symptom of a fragile system](/img/flyover.jpeg)

Now let's think about this in terms of investment planning, route planning, or similar optimisation problems. You will generally optimise by reducing a cost function (travel time, risk cost, etc.) subject to constraints (sites you must visit, annual budget, etc.) but what is generally neglected is the sensitivity of these constraints, along with related assumptions, to the cost function. Say an infrastructure planner optimally plans a road network for an expected traffic volume. A plan is found that serves the projected growth in road traffic over the next ten years with the minimum cost. The model sees no benefit in oversizing this project, even though a small additional cost may result in a project that delivers much more capacity. Then six years pass and the traffic growth projection was underestimated and when road works coincide with home games at the sports stadium there is mass congestion. 

Planning in such a lean way may appear optimal but is actually fragile. However, fat in the system can mitigate Black Swans. 

### Options rather than determinism

There is something very deterministic about most kinds of modelling - strategic models even more so. Rarely are options presented; it is the one best way or suite of projects in the portfolio of assets to replace over a multi-year period and that is that.

The reason that optimisation fragilises is because the act of removing all fat in the system is the removal of optionality. In the book, debt is seen as the ultimate example of fragilisation because it removes all optionality from you.

This kind of deterministic approach is described by Taleb as _Aristotelian_ thinking. It is also rooted in the enlightenment, where scientific optimism led people to believe that everything could be eventually known and measured with ever increasing accuracy. Our post-modern world is more suited to _Thalesian_ thinking, where uncertainty and volatility is expected. Far from being a recent phenomenon, this view is actually rooted in stoicism, where the way to tackle chaos and non-linear effects was to keep ones options open. Core to stoicism is the realisation that much of what happens in life is outside of our control. Taleb extends this to say that much of what happens in life is outside our ability to anticipate and forecast: an aleatory uncertainty.

So much philosophy, but how does this relate to the real world? Let's take two unrelated examples: commodity risk for electricity generation and portfolio optimisation. A deterministic approach for commodity risk might be to attempt to accurately model the whole electricity market, river flows into hydroelectric power schemes and all your take-or-pay gas contracts, along with the most accurate weather/forward electricity price/spot gas/demand forecasts and attempt to model the next few weeks or months of supply and demand. A more intellectually honest approach might be to run Monte Carlo simulations of distributions of reasonable values for all these inputs and measure the spread of outcomes. A Thalesian approach would be this plus stress test modelling for specific extreme worst case scenarios (black swans) and measure how long the business survives (_premeditatio malorum_ to the stoics). Even if this specific scenario never happens, by having Treasury maintain the amount of cash on hand to weather it you can have reasonable confidence in being able to weather similar  magnitude events. We actually had this hybrid probabilistic and extreme stress test modelling approach when I worked at Contact Energy.

<img src="/img/clyde_powerstation.png" alt="" width="100%"/>

The second example is portfolio optimisation for construction/infrastructure projects. A deterministic approach might be to gather all the input data for potential projects (cost, benefit, duration, prerequisite projects, exclusivities between projects) and then optimise the Gantt chart for maximum benefit under cost and time constraints. A better way would be to benchmark historic projects against those proposed to weed out the hubris caused by the planning fallacy. A Thalesian approach would be to model many times, perturbing the input values to test the sensitivity of the whole programme to changing inputs. From there you would be able to see projects that are much more resilient to price shocks and schedule blow out, and projects that enable optionality (e.g., those that are the prerequisite to others but themselves have no prerequisites) rather than those that constrain optionality.

Join me for part [two](/post/antifragile-book-notes-part-2/), where I will share my notes on _via negativa_, the curse of size, the average versus the dispersion, and the burden of proof of the novel.