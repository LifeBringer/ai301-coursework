I fixed a bug in firstmate, an open source tool that runs a team of AI coding agents, and the project merged my fix on November 12. The tool keeps a summary of every task, and on large projects its refresh ran past a 60-second limit, so the summary stopped updating.

I hit the bug on my own computer first: 118 failed refreshes in two weeks. The tool built the summary one task at a time, with slow extra work for every line of history. My change builds the tasks side by side and removes that extra work. The summary itself stays the same. In my test, the refresh finished in 6 seconds instead of 37. On a busy computer, the old code missed the limit, and mine finished in 20 seconds.

My pull request stayed open for four weeks. In that time I learned that two other people had opened fixes before me. I had not checked, so I said so on the issue and named their work. Before I asked the maintainer for more review time, I ran my change through a checklist I wrote for this course. It stopped me twice, once because my description promised more than my change did. I fixed both problems, and the pull request merged that same day.

What I learned: check your work, and what you say about it, before you ask someone else to review it.

https://github.com/kunchenguid/firstmate/pull/7376

#OpenSource #SoftwareEngineering #AI
