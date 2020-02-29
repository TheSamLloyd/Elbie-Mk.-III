# Elbie (Landon Bot)

> **_NOTE_**: This guide is being rewritten as we move into 4.0, which I am really excited about. Please feel free to submit bugs and issues as they arise. We're now significantly more flexible in terms of adding systems, so if you need a new system implemented, please let me know and I will work on it! Elbie has been rewritten not quite from the ground up but more from the top down in Typescript, allowing us more predictable behaviors and better debugging, as well as way more flexibility.

An extensible [Discord](https://discordapp.com/) bot designed to facilitate running rules-lite [TTRPG](https://en.wikipedia.org/wiki/Tabletop_role-playing_game) systems, including [Dungeon World](http://dungeon-world.com/), [Fiasco](http://bullypulpitgames.com/games/fiasco/), and others, with rudimentary support for Dungeons & Dragons 3.5e.
What makes Elbie distinct from many of the conventionally available Discord bots is her tight RPG integrations, capable of handling skills, inventory, dice rolls, and creating character summaries on the fly. She also has rudimentary (and in-progress) audio capabiities, with built-in support for character theme songs.

Elbie's purpose is to provide a wide array of tools and accessories and then get out of the way, letting the players and DM be able to act freely without having to constantly be referring to a character sheet. This enables better role-playing and more mentally present players.

## Installation

### Adding to a server

Elbie lives in Heroku at present to make her available 24/7. To add Elbie to a Discord server, visit [this link](https://discordapp.com/oauth2/authorize?client_id=255390394219626496&scope=bot&permissions=1) and select a server in which you have the Manage Server permission.

### Installing Locally

To run Elbie locally, fork this repository and clone it to your local machine. You'll need to register a Discord app to get a login token, and if you want her to report version information, you'll need a Github token as well. You'll need to add her to a server as well, which you can do while registering her as a Discord app.

## Usage

All of Elbie's commands start with a **prefix**, letting her know that you're talking to her. By default, this prefix is set to `+`, such that asking her to flip a coin is simply `+flip`.

Elbie is broken up into **modules**, each of which encapsulates several related commands. The "master" version of Elbie has all modules I've written installed, but writing one's own is fairly simple and lets Elbie be easily extensible. As it stands, the modules available in the master version are:

* bot
  * `+ping`: Returns "pong!" as proof-of-life.
  * `+echo (String)`: Echoes back `(String)` as proof-of-life
  * `+flip (String? heads) (String? tails)`: Returns either "heads" or "tails", or if provided, returns one of the pair of options given.
  * `+choose (String options)`: Returns one of the (space-delimited) choices at random, e.g. `+choose a b c d e f horse` will return either "a", "b", "c", "d", "e", "f" or "horse". At least 2 choices must be provided.
* rpg
  * `+roll (Integer? modifier | String? skill | String? stat) (extra dice as '+roll')`: Makes a single default roll as defined by whatever TTRPG system the channel is currently bound to (2d6 for PBtA games, 1d20 for most D&D/Pathfinder games, etc.) If `modifier` is provided, it will be added as a modifier. Aliased as `+roll`.
  * `+bind (String shortname) (String name)`: (If you're interested in binding Elbie to a channel for a campaign, just email me and we can talk!)
  * `+who (String? name)`: Generates a character summary of either the current player if no name is provided, or if `name` is a player name (like "Sam"), a character name (like "Lurreka Al-Petarra"), or a nickname (like "Lurreka"), it will generate a summary of that character or the character assigned to that player. (For reference, if for some reason you have a player named "Sam" and a different player has a character named "Sam", `+who Sam` will refer to the player. If you are doing this, why?)
  * `+list`: Lists all the characters in the current campaign with their player's name (e.g. "Amateotl Maikali, played by Sam")
  * `+hp (Integer? amount)`: Adjusts the current character's HP by the given `amount`. Restricts to values between 0 and the character's maximum health. Not providing `amount` returns the character's current HP.
  * `+mark (Integer? amount)`: Increases the current character's XP by `amount` if provided. Defaults to 1.
  * `+levelup`: Increases the character's level by 1, if the character has enough XP. If not, fails. (Implementation of this command successfully depends on the system.)
  * `+theme`: Immediately plays the current character's theme (assuming audio module is installed and working.)
  * `+adv`: Accepts one dice string (`xdy+z`) and rolls it twice, returning both results and taking the higher one.
  * `+disadv`: Accepts one dice string (`xdy+z`) and rolls it twice, returning both results and taking the lower one. Aliased with `+dadv`.
* audio (Not implemented in 4.0.0a)
  * This module is in active development and features are likely to change without significant warning at any time. If you are messing with audio and you crash Elbie, it is your fault. I will reboot her but I *strongly* encourage you not to hurt my daughter.
  * `+play (String URL)`: Plays the audio from the linked YouTube video. If you want, you may omit everything except the video ID. If you do include the full video URL, it will likely embed in the channel, depending on permissions. If a song is currently playing, asking Elbie to play a new song will have her stop the old one and begin playing the new one as soon as it is cached.
  * `+stop`: Elbie will stop playing any audio she is currently playing.
  * There, at present, is not a way to adjust Elbie's volume on a channel-wide basis, but every player can adjust her to their liking by right clicking her icon in the voice channel and adjusting "Player Volume."
  * There is also current *no queueing available*, so at present you'll have to be courteous not to impose on someone else's song.
  * Other improvements we plan to make include her ability to read YouTube playlists so that a DM could prepare a soundtrack, and to loop songs.

## Extending Elbie

Elbie is written to be fully extensible. All you need to do to write your own module is create a folder inside of `bot_modules` titled for your new module, and inside that create `index.ts`. You may want to import `common` from `../common`, which provides several common functions required by modules. Then you'll need to add a new class in `index.ts`:

```typescript
import {Module, Command} from '../module'
class newModule extends Module{
  name = 'NewModule'
  desc = 'My cool new module.'
  foo(command:Command):void{
    //function body
  }
  commands = {
    alias: {key: this.foo, desc: 'Does nothing, but looks nice in my new module.'}
  }
}

export default new newModule()
  ```

To break this pattern down, `name` and `desc` are self-explanatory. In the `commands` object, `alias` is what you type to get Elbie to trigger the command ("+alias"), and she will execute `this.foo()` with the trigger command. All of your functions that take a command as an argument should ONLY take a command and have the return type `void` unless you have a really, really good reason to break from that pattern. Encapsulation means that modules only should know the minimum they need to function.

After you've made your module, add an entry in `bot_modules/index.ts` pointing to your new module.

If you've done everything right, when you start up Elbie, she should log the modules she has installed in the console and you should see your module's name and description, as well as its commands, which you should be able to use.

## Future Enhancements

### Audio

Audio is her next big step -- having her able to smoothly play music from youtube and possibly other sources would help her be a bot that can be used in a variety of circumstances.

### Web Interface

There is a lot you can do with a text interface, but for our purposes, setting up a web interface for instantiating a campaign, adjusting character attributes, or making administrative changes to your TTRPG sessions is going to be easier for everyone.

### Grid-based Movement

Many TTRPGs use a game-board like grid for character positioning and line-of-sight determination. At present, there are no plans to include such a feature in Elbie. Get back to role-playing!! ;-)

## Thanks & Tech

* [discord-js](https://discord.js.org)
* node
* express
* yarn
* Mongoose
* MongoDB
* Heroku
* ytdl

_____

* [Friends at the Table](http://friendsatthetable.net/)
* [The Adventure Zone](http://www.maximumfun.org/shows/adventure-zone)
* [Dungeon World](http://dungeon-world.com/)
* The Pilgrims of Her Blue Flame

## One Last Thing

If you're a traditional D&D or Pathfinder player who has grown a little tired of the standard formula of "go into dungeon, kill goblin, walk out with treasure, repeat until dead", I urge you to check out any of the "Powered By the Apocalypse" games. Dungeon World is their D&D analogue, and since our group switched, we've seen better stories, more realistic role-playing, less focus on stats and skills and more focus on interpersonal connection.

If you've never played a TTRPG before, Dungeon World is a great place to start. There are few rules to memorize, lots of things you can do, and lots of ways to be an extraordinary person in a fantasy world. I'm blessed to work with the world's greatest Game Master, but there are lots of great published adventures that your group can run that don't involve writing a novel's worth of backstory.
