# cs333-lab6
Put your jokes API on your own server, so anyone on the internet can use it 🚀😂

In Lab 5 your jokes API ran on `localhost`, so only you could reach it. Now it goes on **your own server**: **pm2** keeps it running after you log out, and **nginx** passes requests from the internet to it.

Do each step below, and **answer the questions right here in this `README.md` file** as you go.

- **Part 1 — log in.** Your server, and what's on it.
- **Part 2 — run the API with pm2.** It keeps running after you log out.
- **Part 3 — nginx in front.** Requests to `/api/` go to your API.
- **Part 4 — use it from your laptop.** REST Client, pointed at your server.

You need your **finished Lab 5** (the `Jokes` folder, all 7 routes) pushed to GitHub.

**Stuck?** Look at the **Stuck?** table at the bottom first. Then post in the Slack lab thread: (1) the command you ran, (2) the **exact** error, copied and pasted, (3) what you already tried.

**Commands with `sudo`** run as the administrator. You have `sudo` on your server, so be careful: read each command before you press Enter.

---

# Part 1: Log in

Your server's address and username are in Slack. On your laptop:

```bash
ssh yourname@your-server-address
```

Everything until Part 4 is typed **on the server** (your prompt shows the server's name).

## 1. Look around

If you did Part 4 of Lab 5, skip to step 2. If not, type each of these and look at what it says:

| Type this | It shows |
| --- | --- |
| `whoami` | your username |
| `hostname` | the server's name |
| `node --version` | which Node |
| `df -h /` | how much disk space there is |
| `free -h` | how much memory there is |

**Question 1:** what version of Node is on your server? Is it the same as on your laptop?

## 2. Get your Lab 5 code

Your Lab 5 repo has to be **public** for this (GitHub: your repo, **Settings**, at the bottom: **Change visibility**).

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-LAB5-REPO.git
cd YOUR-LAB5-REPO/Jokes
npm install
```

## 3. Run it once, by hand

```bash
node index.js
```

It says `Listening on http://localhost:3000`. Leave it running and open a **second** terminal on your laptop, `ssh` in again, and type:

```bash
curl http://localhost:3000/random
```

`curl` sends a GET request from the terminal, like REST Client does. You should get a joke back.

Now go back to the first terminal and press **Ctrl+C**. Send the `curl` again.

**Question 2:** what does `curl` say now? Why? What would happen to your API if you just closed your laptop?

---

# Part 2: Keep it running with pm2

**pm2** is a process manager: it starts your Node app, keeps it running when you log out, and restarts it if it crashes.

## 4. Install pm2 and start your API

```bash
sudo npm install -g pm2
pm2 start index.js --name jokes
pm2 list
```

`pm2 list` shows a table. **Question 3:** what is the `status` of `jokes`? What does the `↺` column count?

```bash
curl http://localhost:3000/random
```

It works, and you didn't have to keep a terminal open.

## 5. Logs and restart

```bash
pm2 logs jokes --lines 20
```

**Question 4:** what's in the log? Which line of `index.js` printed it? (Press **Ctrl+C** to stop watching the log. That only stops the log, not your API.)

Other pm2 commands you'll use:

| Type this | It does |
| --- | --- |
| `pm2 list` | shows what pm2 is running |
| `pm2 logs jokes` | shows what your app prints (`console.log` and errors) |
| `pm2 restart jokes` | restarts it (do this after you change the code) |
| `pm2 stop jokes` | stops it |

## 6. Survive a reboot

```bash
pm2 save
pm2 startup
```

`pm2 startup` prints **one** `sudo env PATH=...` command. Copy that whole line, paste it and run it. Now pm2 starts your API again whenever the server restarts.

---

# Part 3: nginx in front

Your API listens on port **3000**. Web browsers use port **80** for `http://`. **nginx** (say "engine X") is a web server that listens on port 80. We'll tell it: anything that starts with `/api/`, pass on to port 3000. That is called a **reverse proxy**.

```
browser  --->  nginx (port 80)  --- /api/... --->  your jokes API (port 3000)
```

## 7. Install nginx

```bash
sudo apt update
sudo apt install nginx
```

(If it says nginx is already the newest version, that's fine.) Open `http://your-server-address` in your laptop's browser.

**Question 5:** what page do you see?

**TODO:** also try `http://your-server-address:3000/random` in your browser. **Question 6:** what happens? (Either answer is fine; just say what you see.)

## 8. Send /api/ to your API

Open nginx's settings for your site:

```bash
sudo nano /etc/nginx/sites-available/default
```

Find the `location / {` block (a few lines, ending with `}`). Right **after** its `}`, type this:

```nginx
	location /api/ {
		proxy_pass http://localhost:3000/;
	}
```

Save: **Ctrl+O**, **Enter**, then **Ctrl+X** to leave nano.

Check what you typed, then load it:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

`nginx -t` tests the file. If it says anything other than `syntax is ok` and `test is successful`, read the line number it gives you, open the file again and fix it.

Open `http://your-server-address/api/random` in your browser. 🎉

**Question 7:** the browser asked for `/api/random`, but your Express route is `/random`. Which part of the nginx setting takes `/api` off? (Hint: try it without the last `/` in `proxy_pass` and see what happens. Put it back after.)

---

# Part 4: Use it from your laptop

Type `exit` to log out of the server. Your API keeps running.

## 9. REST Client, pointed at your server

Open `server.http` in this repo. On the second line, put your server's address in place of `your-server-address`. `{{base}}` in each request is replaced with that line, so you only type the address once.

Send requests **1**, **2** and **3**. They are the same requests as in Lab 5, now going over the internet.

## 10. Where did my joke go?

Send request **4** (POST), then request **5** with the id your joke got. Then `ssh` in and type:

```bash
pm2 restart jokes
exit
```

Send request **5** again.

**Question 8:** what happened to your joke? You saw this in Lab 5 too (Question 8). What is the fix? (That's where we go next: MongoDB.)

**Question 9:** in your own words, what does pm2 do, and what does nginx do? Why do we need both?

---

# Hand it in

11. Push this repo (`server.http` with your address, and this README with your answers) to **your own GitHub repo**.
12. Submit in Moodle: your GitHub repo link **and** your API link, `http://your-server-address/api/random`. Leave your API running so I can test it.

---

## Stuck?

| You see | What's going on | Fix |
| --- | --- | --- |
| `Permission denied (publickey)` | Your server doesn't have your SSH key, or you used the wrong username | Check the username in Slack, then ask in the Slack thread |
| `git clone` asks for a username and password | Your Lab 5 repo is private | Make it public (step 2), then clone again |
| `Cannot find package 'express'` | You didn't `npm install` in the `Jokes` folder | `cd` into `Jokes`, then `npm install` |
| `EADDRINUSE: address already in use :::3000` | Your API is already running (by hand or in pm2) | Ctrl+C the one you started by hand; `pm2 list` to see pm2's |
| `pm2: command not found` | pm2 isn't installed | `sudo npm install -g pm2` |
| `sudo: npm: command not found` | Node on your server was installed just for your user (with nvm) | Leave out `sudo`: `npm install -g pm2` |
| `pm2 list` says `errored`, or `↺` keeps going up | Your app crashes when it starts | `pm2 logs jokes` shows the error |
| `curl: (7) Failed to connect to localhost port 3000` | Your API isn't running | `pm2 list`, then `pm2 restart jokes` |
| `nginx -t` says `unexpected "}"` or `unknown directive` | A typo, or a missing `;` or `}` | Check the line number it gives you; every setting ends with `;` |
| `502 Bad Gateway` | nginx is working, but your API isn't running | `pm2 list` and `pm2 logs jokes` |
| `404 Not Found` from nginx | The `location /api/` block isn't there, or nginx wasn't reloaded | Check the file, `sudo nginx -t`, `sudo systemctl reload nginx` |
| `Cannot GET /api/random` (from Express) | The last `/` is missing in `proxy_pass` | `proxy_pass http://localhost:3000/;` |
| The browser can't connect at all | Wrong address, or the server isn't on | Check the address in Slack; `ping your-server-address` |
