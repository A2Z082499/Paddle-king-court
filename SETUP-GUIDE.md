# Setup guide: set up the booking site for a new client

Allow about 45 minutes the first time, 20 minutes once you know the steps.

## What you need from the client
- Business name, address, phone, email, a short tagline
- Court names (this version has exactly **two** courts), opening hours, prices
- Their **GCash number, account name and QR code image**
- Their logo (square JPG works best) and their Facebook/Messenger link
- A **Telegram** account on the owner's phone (for instant booking alerts; free)

## 1. Accounts (the client should own them)
1. Client creates a **GitHub** account and a **Supabase** account with their own email.
2. Either do the next steps on a screen-share, or have them add you as a collaborator/team member and remove yourself at the end.

Never host two clients in one Supabase project or one GitHub account of yours.

## 2. Create the Supabase project
1. supabase.com -> New project. Pick a name, a strong database password (save it), and the closest region.
2. Wait for it to finish creating.
3. Open **Project Settings -> API** (the label can differ slightly by version). Copy:
   - the **Project URL**
   - the **publishable / anon key** (the public one, never the "service role" or "secret" key)

## 3. Turn on booking alerts (optional but recommended)
1. In Telegram, search for **@BotFather**, send `/newbot`, follow the prompts, and copy the **token** it gives you.
2. Open your new bot in Telegram, press **Start**, and send it any message (for example "hi").
3. In a browser open `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` (put your token in place of `<YOUR_TOKEN>`).
4. Find `"chat":{"id":123456789` in the result. That number is the **chat id**. If the result is empty, send the bot another message and reload.

Alerts come to the owner's phone the moment a customer books or cancels, even when no page is open. They are free.

## 4. Run the database setup
1. Open `setup.sql` in a text editor. Edit **only** the lines marked `EDIT` in the SETTINGS block:
   - `s_admin_email`, `s_admin_password` (10+ characters, unique for every client, never `admin123`)
   - court names, prices, peak hour, first and last bookable hour
   - `s_site_url` (the final website address, for example `https://name.github.io/Game-time`)
   - `s_telegram_token` and `s_telegram_chat` from step 3
2. Supabase -> **SQL Editor -> New query**, paste the whole file, click **Run**.
3. You should see **Setup complete**. If you see a red error, read it: it usually says what to fix. You can run the file again any time (to change prices, the password or the Telegram details).
4. If alerts do not arrive later, open **Database -> Extensions** and make sure **pg_net** is enabled, then run `setup.sql` again.

The page's prices and hours (next step) must match what you typed here.

## 5. Edit the page settings
Open `index.html` and find the `CONFIG` block near the top of the `<script>`. Fill in:

| In CONFIG | Must match |
|---|---|
| `supabaseUrl`, `supabaseKey` | Step 2 |
| `courts` | `s_court_king`, `s_court_queen` in setup.sql |
| `openHour`, `lastHour`, `peakFrom`, `priceLow`, `priceHigh` | the same names in setup.sql (24-hour clock, 24 = 12 AM, 27 = 3 AM) |
| `hoursText`, `tagHours`, `priceText`, `dial` | the text customers read (change it to match the numbers above) |
| `brand`, `address`, `phone`, `email`, `tagline`, `headline`, `earlyNote` | the client's business details |
| `gcashNumber`, `gcashName`, `gcashQr`, `messengerLink` | the client's own payment details |

Open `admin.html` and edit its `CONFIG` block: the same `supabaseUrl`, `supabaseKey`, `brand` and `courts`.

Then add the client's `logo.jpg` and `gcash-qr.jpg` to the same folder (if the logo is missing, a gold circle shows instead).

## 6. Publish on GitHub Pages
1. In the client's GitHub account, create a repository and upload `index.html`, `admin.html`, `logo.jpg` and `gcash-qr.jpg`. (`setup.sql` is not needed online. Keep it private, because it contains the admin password.)
2. **Settings -> Pages** -> deploy from the `main` branch, root folder.
3. After 1 to 2 minutes the site is at `https://<name>.github.io/<repo>/` and the admin page at `.../admin.html`.
4. Optional custom domain: buy one, add the GitHub A records and a `www` CNAME, then enter it under Pages -> Custom domain and tick Enforce HTTPS.

## 7. Test everything before handing over
Do each of these yourself, on a phone and a computer:
- [ ] Booking page loads, logo and text are correct, prices and hours are right
- [ ] Book 2 hours on King Court. The payment popup shows **their** GCash number and QR
- [ ] A Telegram alert arrives with the booking details
- [ ] The slots stay open on the page (pending bookings do not block the time)
- [ ] Log in to the admin page, see the booking, **Confirm** it. The name appears on the slot of the booking page
- [ ] Book the same hour from another phone number: the admin page shows "#1 of 2" and the confirm button on the second one is blocked
- [ ] Book, then use **Change time** and **Cancel booking** in the popup. A cancel alert arrives
- [ ] Book the **last** slot (the 12 AM to 4 AM ones, if used) and a slot on Queen Court
- [ ] **Delete** a booking in the admin page: the time opens up again
- [ ] Wrong admin password is refused. A customer cannot open the admin list

## 8. Hand over to the client
- Give them: the website address, admin address, admin email and password. Ask them to bookmark the admin page on their phone.
- Tell them the daily routine: check the Telegram alert or the **To confirm** tab, confirm the GCash payment, press **Confirm booking**. If the payment never comes, press **Delete** to free the time.
- On the admin page press **Alerts: off** to switch it to **on** for a sound and pop-up while the page is open. Telegram alerts work without it.
- Explain that the free Supabase plan can **pause the project after a week of no activity** (a quiet site). Paid plans do not pause. Check Supabase's current pricing, and agree in writing who pays for hosting or support.
- Tell them to change the admin password if it was ever shared (re-run `setup.sql` with a new `s_admin_password`).

## Common problems
| Problem | Fix |
|---|---|
| Page shows 404 | The file is not in the repo folder, or the name has capital letters. Wait 1 to 2 minutes and hard-refresh (Ctrl + Shift + R) |
| "Admin setup not finished" on login | `setup.sql` did not run successfully. Run it again |
| "This account is not an admin" | Log in with the same email as `s_admin_email` |
| Slots do not load or booking fails | `supabaseUrl` / `supabaseKey` are wrong, or the project is paused (restore it in the Supabase dashboard) |
| Times show wrong prices | `CONFIG` and `setup.sql` prices or peak hour do not match |
| A late-night slot is missing or refused | `lastHour` in `CONFIG` and `s_last_hour` in `setup.sql` do not match |
| No Telegram alerts | Wrong token or chat id, you never sent the bot a message, or pg_net is off (see step 4) |

## Known limits of this version
- Exactly two courts (their names are configurable)
- Alerts go to Telegram (free). SMS or email would need a paid service
- The spam guard limits each phone number to 12 pending hours per day. There is no CAPTCHA
- Customers can cancel or change only their own **pending** booking, only while the payment popup is open
