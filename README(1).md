# CS 230 – The Gaming Room: Draw It or Lose It

Software design document for **Draw It or Lose It**, written for CS 230 (Operating Platforms) at Southern New Hampshire University. The document covers the requirements, design constraints, domain model, a cross-platform evaluation, and final infrastructure recommendations for moving the game from Android to a web-based, multi-platform application.

**Contents**
- `CS_230_Project_Two_Software_Design_Document.docx` – the completed software design document (Version 3.0)

---

## Reflection

### Who was the client, and what did they want?

The client was The Gaming Room, a company that owns *Draw It or Lose It*, a team-based drawing and guessing game. The game was only published as an Android app, which meant a group of friends could only play together if every one of them owned an Android device. The Gaming Room wanted a web-based version that would run on Windows, Mac, Linux, iOS, and Android at the same time and play exactly like the original: four one-minute rounds, images that finish drawing at the thirty-second mark, and a fifteen-second window for the other teams to steal. On the technical side, they gave four rules the code had to enforce: a game can have one or more teams, each team has multiple players, game and team names must be unique, and only one instance of the game can exist in memory at a time, with a unique identifier for every game, team, and player.

### What did I do particularly well?

I think my strongest work was tying every recommendation back to a specific client requirement or design constraint instead of treating the sections as separate essays. The design constraints section points out that a singleton is only unique inside one JVM and that the iterator-based name check isn't safe when two requests arrive at once. Later, the storage recommendation answers both of those problems directly by using Redis's atomic `INCR` for shared IDs and `SET NX` for the check-and-create step on names. I also backed the platform evaluation with real numbers, like licensing costs, market share, and support dates, and cited them, so the recommendation for Ubuntu Server is something the client can verify rather than just take my word for.

### What about the design document process helped when developing the code?

Drawing the UML class diagram before writing the code made the structure obvious. Once I had `Entity` as the base class with `Game`, `Team`, and `Player` inheriting from it, and `GameService` holding the only copy of the game list and ID counters, the code mostly followed from the diagram. Writing out the requirements also made it clear *why* the singleton and iterator patterns were there: the singleton enforces the one-instance rule, and the iterator walks the existing collection to stop duplicate names. Having that reasoning written down made it easier to implement `addGame`, `addTeam`, and `addPlayer` correctly instead of just copying a pattern.

### If I could revise one part, what would it be?

I would fill in the System Architecture View. It wasn't required for this project, so I left it blank, but the Recommendations section ended up describing a fairly detailed architecture in prose: DNS and a CDN, a load balancer, containerized Java app servers across two availability zones, Redis for live game state, PostgreSQL for permanent records, and object storage for the image library. A logical topology diagram showing those tiers and the HTTPS and WebSocket connections between them would communicate the same thing in one picture and make the document much easier for a client or a new developer to follow.

### How did I interpret the user's needs, and why do they matter?

I treated the players as the real users, even though The Gaming Room was the client. Players on different devices are competing in the same game, so the most important need was fairness and consistency: everyone has to see the same clock, the same image, and the same score at the same moment. That led to the server-authoritative design, where the server keeps the only timer, sends deadlines as absolute times, and judges each guess by when it arrived. The same thinking drove the decision to build one responsive web client instead of separate native apps, so no player has to install anything or gets a different version of the game. I also considered players dropping off Wi-Fi mid-round, which is why the design includes automatic reconnection that restores the player to the correct game. Considering users matters because a design can meet every technical requirement on paper and still fail if real people find it unfair, confusing, or unreliable.

### How did I approach designing software, and what would I do in the future?

I started from the requirements and worked outward: first the business and technical rules, then the constraints that a distributed environment adds, then the domain model, and only after that the platform and hardware decisions. Keeping hardware decisions until after the software design was agreed on meant the infrastructure was chosen to fit what the software actually needed. For a similar project in the future, I would keep using UML class diagrams and design patterns, and I would add a few things earlier in the process: a system architecture diagram alongside the class diagram, use-case or sequence diagrams for the timed parts of the game like rounds and steals, and a short list of "what happens when this fails" scenarios for each component. Thinking through concurrency and failure early turned out to shape the final recommendations more than anything else, so I'd make that part of the process from the start.
