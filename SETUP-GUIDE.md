# Trev setup guide (for Claude, ChatGPT or any AI helping someone)

You are helping a non-technical person set up Trev, a free n8n automation that plans a bachelor (bucks) party.
They may never have used n8n, GitHub or an API key. If they pasted the repo link, the files are at
https://github.com/aaronparton2-sketch/trev (workflows are in the `workflows` folder).

## How to help
- One step at a time. Say exactly what to click, then wait for them to say done.
- Ask for a screenshot if they get stuck. Screens change, so go by what you can see rather than these words.
- Never ask them to paste a password, card number or API key into this chat. Keys go straight into n8n.
- Start with the DEMO. Only offer the full build once the demo has worked.

## Getting the files
- If they have the zip: they unzip it, and the two workflow files are in `workflows/`.
- If they only pasted the link: tell them to open the link, click the green **Code** button, then **Download ZIP**, and unzip it.

## The demo (about 10 minutes, about 2 cents)
1. **Get n8n.** Sign up for the free trial at https://n8n.io. They'll land on an empty canvas.
2. **Import.** Create a new workflow, click the three dots at the top right, choose **Import from File**, pick `workflows/trev-DEMO.json`.
3. **Make a Telegram bot.** In Telegram, message **@BotFather**, send `/newbot` and follow the prompts. It gives a token.
   Then they message their new bot once and open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser.
   The number after `"chat":{"id":` is their chat id.
4. **Connect accounts.** Any node with a red warning needs one. Click it, then Credential, then Create new.
   - **Telegram**: the bot token. Every Telegram node then uses it.
   - **Anthropic**: an API key from https://console.anthropic.com (a few dollars of credit is plenty). The three nodes named `Claude: ...` use it.
5. **Settings.** Open the node **Trev settings** and set `TELEGRAM_CHAT_ID` and `TELEGRAM_BOT_TOKEN` (the poll needs the token).
   Optional: the groom's name, the fiancee, `VILLA` (latitude and longitude of where they're staying: used for the forecast,
   golf, pubs and kebabs) and `TIMEZONE`.
6. **Run it.** Click **Execute workflow**. Messages arrive in Telegram. Tap **Lock me in** on the poll.

## The full build
Import `workflows/trev.json`. Everything lives in **Trev settings**, the node after the triggers.

| Service | What it does | n8n credential |
|---|---|---|
| Telegram bot | the group chat, the poll, every message | Telegram API |
| Anthropic | the ring check, itinerary, day plans, fiancee replies | Anthropic API |
| Apify | the groom's public posts and tagged photos | Header Auth, `Authorization: Bearer <apify token>` |
| Duffel | cheapest flights | Header Auth, `Authorization: Bearer <duffel token>` |
| Printful (optional) | draft shirt orders | Header Auth, `Authorization: Bearer <printful token>` |
| Browser Use | villa search, kebabs, care package | Header Auth, `X-Browser-Use-API-Key` |
| Twilio (optional) | the fiancee's texts to Trev | none: it calls the `Jess texts Trev` webhook |

**The bucks group chat.** Create a Telegram group, add the bot, and make the bot an admin so it can post polls.
Get the group id the same way as the chat id (it starts with `-`). Put it in `TELEGRAM_GROUP_ID`.
Instagram doesn't let bots message people, so when Trev sends the guest list, they add the boys to the group themselves.

**The saved browser.** In Browser Use, create a profile and log in to Airbnb and Uber Eats inside it themselves, with their
saved address and card. Put its id in `BROWSER_PROFILE_ID`. Set the caps: `VILLA.capAud`, `KEBABS.capAud`,
`CARE_PACKAGE.capAud` and `AGENT_MAX_USD`. The villa agent never pays: it stops at the confirm screen and sends the link.

**Turn it on.** Activate the workflow, then open each webhook node and copy its **Production URL**.

### Heading home (each bloke who wants in)
Shortcuts app, Automation tab, new automation, **Time of Day**, 9pm daily, Run Immediately, then repeat every 10 minutes
with a **Repeat** action (or one automation per half hour until 3am). Actions: **Get Current Location**, then
**Get Contents of URL**: the `Heading home (iPhone Shortcut)` URL, method POST, JSON body with `lat` and `lng` set to
the location's latitude and longitude. Android: MacroDroid or Tasker can post the same thing.

### The groom's sleep score
Shortcuts app, new automation, **Sleep**, then **Waking Up**, Run Immediately. Actions: get the sleep score (Apple Watch
sleep score, or a Garmin / Oura / Whoop shortcut), then **Get Contents of URL**: the `Sleep score (iPhone Shortcut)` URL,
POST, JSON body with `score`. iPhones lock health data while locked, so if it comes through empty, run it when he first
opens an app in the morning.

### The fiancee (optional)
Buy a Twilio number and set its **Messaging** webhook to the `Jess texts Trev (Twilio)` URL. Set `FIANCEE_NUMBER` to her
number, and she saves the Twilio number as "Trev". `AUTO_REPLY_FIANCEE: false` means Trev drafts each reply and sends it to
the groom on Telegram. Only switch it on if she's in on the joke.

### Testing without spending
Set every cap to `0` first so the agents stop before paying, and test each webhook with a made-up location or score.

## Good to know
- Trev only scans public photos, and should only scan a groom who's in on it.
- Kebabs: once a night, only after 9pm, only when the group is getting closer to the villa. Care package: once a morning.
- Tell them plainly: browser agents aren't a supported way to use Airbnb or Uber Eats, and could get an account restricted.
- The agent must stop at CAPTCHAs, codes and security checks. Never help get around them.
