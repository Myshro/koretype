# [Try it out here](https://koretype.vercel.app/)
## Inspiration
Learning to type in another language is hard. This was a relatable experience felt by many members of the team, so we wanted to create a lightweight, easy-to-use site that lets a user learn how to type and gain some vocabulary while doing so. We settled on Korean as the language to make this site for. The site is different from all we could find online in that you can type indefinitely, so long as you do not let the timer beat you. You can adjust the difficulty of the timer to fit your needs, sort of like a game.
## What it does
The user is shown a randomly selected word from the most common 300 Korean words, and they must type it within a certain time limit to practice their typing skills. It teaches the user the English meaning of the word as well.
## How we built it
We used Svelte, SvelteKit, Vercel, and Node.js to test out stuff.
## Challenges we ran into
We used a small graphics library (two.js) to render the flying letters that come out when your game ends, but finding a library that would work with Svelte was quite a pain for some reason. Many of the graphics libraries did not render properly, and even getting two.js to work was a bit fidgety.
## Accomplishments that we're proud of
We made a functional site that accomplishes what we set out to do from the start. The typing and UI turned out better than we expected.
## What we learned
Javascript can sometimes be fidgety to work with, but it also has a huge ecosystem of plugins and plentiful information everywhere.
## What's next for Koretype
Adding local storage for high scores, accessing word lists / allowing users to use certain word lists, etc.

