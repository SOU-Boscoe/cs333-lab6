# cs333-lab6
MongoDB on your laptop, and your jokes API live on your own server 🚀😂

This lab has two halves:

- **Parts 1–2, on your laptop: MongoDB.** Create, read, update and delete data in the MongoDB shell, then save data from a Node program with **Mongoose**.
- **Parts 3–6, on your server: your Lab 5 jokes API, live on the internet.** You copy it up, **pm2** keeps it running, and **nginx** passes requests from the internet to it.

Do each step below, and **answer the questions right here in this `README.md` file** as you go.

**You need:**
- your **finished Lab 5** `Jokes` folder (all 7 routes) on your laptop
- **MongoDB and mongosh on your laptop** (Lab 5 Part 3). If you skipped that part, do it first.
- the server setup from class (week 6 slides): you can `ssh yourfirstname@your-server-address` with your key, pm2 is installed, and nginx shows your web page

**Laptop or server?** Every step says **💻 LAPTOP** or **🖥️ SERVER**. Your prompt tells you where you are: your laptop's name, or `yourfirstname@` your server. Most mistakes in this lab come from typing a command in the wrong terminal.

**Stuck?** Look at the **Stuck?** table at the bottom first. Then post in the Slack lab thread: (1) the command you ran, (2) the **exact** error, copied and pasted, (3) what you already tried.

---

# Part 1: MongoDB shell, CRUD 💻 LAPTOP

**CRUD** is the four things you do with data: **C**reate, **R**ead, **U**pdate, **D**elete. You'll do all four in `mongosh`, the same shell as Lab 5 Part 3.

## 1. Open the shell and pick a database

```bash
mongosh
```

At the `test>` prompt:

```js
use shopDB
```

`shopDB` doesn't exist yet. MongoDB makes it the moment you put data in it.

## 2. Create

A **document** is one record, like a row in a SQL table. It looks like a JavaScript object. A **collection** is a group of documents, like a table.

```js
db.products.insertOne({ _id: 1, name: "Pen", price: 1.20 })
db.products.insertOne({ _id: 2, name: "Pencil", price: 0.80, stock: 12 })
show collections
```

**Question 1:** what is `_id` for? What would MongoDB do if you left it out? (Try it: insert a third product with no `_id`, then look at it in step 3.)

## 3. Read

```js
db.products.find()
db.products.find({ name: "Pencil" })
db.products.find({ price: { $gt: 1 } })
db.products.find({}, { name: 1, _id: 0 })
```

**Question 2:** what does `$gt` mean? What does the second `{ name: 1, _id: 0 }` in the last command do? (It's called a **projection**.)

## 4. Update

The Pen has no `stock`. Give it one:

```js
db.products.updateOne({ _id: 1 }, { $set: { stock: 35 } })
db.products.find({ _id: 1 })
```

**Question 3:** `stock` didn't exist on the Pen. What did `$set` do?

## 5. Delete

```js
db.products.deleteOne({ _id: 2 })
db.products.find()
```

**TODO:** put the Pencil back (step 2 has the command).

## 6. One-to-many: documents inside a document

A product can have many reviews. In MongoDB you can keep them **inside** the product, as an array of documents:

```js
db.products.insertOne({
  _id: 3,
  name: "Eraser",
  price: 1.30,
  stock: 43,
  reviews: [
    { authorName: "Jim", rating: 4, review: "Cool Eraser" }
  ]
})
```

**TODO:** add a second review to the Eraser with `$push`. Fill in your own review:

```js
db.products.updateOne({ _id: 3 }, { $push: { reviews: { authorName: "YOUR NAME", rating: 5, review: "YOUR REVIEW" } } })
db.products.find({ _id: 3 })
```

**Question 4:** in a SQL database, where would the reviews go? Why is keeping them inside the product handy here?

Copy everything from your terminal (from `mongosh` to the last command) and paste it here, in a code block:

```
paste your terminal here
```

Type `exit` to leave mongosh.

Commit: `Part 1: MongoDB CRUD`

---

# Part 2: Mongoose 💻 LAPTOP

In Part 1 you typed every command yourself. A real app saves data from its code. **Mongoose** is a library that lets your Node program talk to MongoDB.

**You need two terminals for this part:** one to run your program, and one with `mongosh` open to look at the data it saved. Both are on your laptop.

## 7. Make the project

In this repo's folder:

```bash
mkdir fruitsProject
cd fruitsProject
npm init -y
npm pkg set type=module
npm install mongoose
```

`npm pkg set type=module` lets you use `import`, the same as your Lab 5 code.

## 8. Save a fruit

Make `fruitsProject/app.js` and type this in:

```js
import mongoose from "mongoose";

await mongoose.connect("mongodb://127.0.0.1:27017/fruitsDB");
console.log("Connected to MongoDB");

// A schema is the shape every fruit must have
const fruitSchema = new mongoose.Schema({
  name: String,
  rating: Number,
  review: String,
});

// A model uses the schema. "Fruit" becomes the "fruits" collection
const Fruit = mongoose.model("Fruit", fruitSchema);

const fruit = new Fruit({ name: "Apple", rating: 7, review: "Pretty solid as a fruit." });
await fruit.save();
console.log("Saved:", fruit.name);

const fruits = await Fruit.find();
for (const f of fruits) {
  console.log(f.name, f.rating);
}

await mongoose.connection.close();
```

`await` means "wait here until the database answers". Talking to a database takes time, and the next line needs the answer. (More about this next week.)

Run it:

```bash
node app.js
```

Now run it a **second** time.

**Question 5:** what changed the second time? Why?

## 9. Look at it in mongosh

In your **second** terminal:

```bash
mongosh
```

```js
use fruitsDB
show collections
db.fruits.find()
```

**Question 6:** your model is called `Fruit`. What is the collection called? Are your fruits from Lab 5 Part 3 in there too? Why?

## 10. Many at once, and a wrong type

**TODO:** in `app.js`, put `//` in front of the two lines that make and save the Apple. Under them, add:

```js
await Fruit.insertMany([
  { name: "Banana", rating: 6, review: "Good for a snack" },
  { name: "Kiwi", rating: 9, review: "Underrated" },
  { name: "YOUR FRUIT", rating: 8, review: "YOUR REVIEW" },
]);
```

Run `node app.js` again and check with `db.fruits.find()` in mongosh.

**TODO:** now try to save a fruit whose `rating` is the word `"ten"` instead of a number:

```js
const mango = new Fruit({ name: "Mango", rating: "ten", review: "So good" });
await mango.save();
```

**Question 7:** what error do you get? How does the schema protect your data? (Then delete the Mango lines.)

Commit: `Part 2: Mongoose`

---

# Part 3: Copy your jokes API to your server

Now your Lab 5 jokes API goes on your server.

**You need two terminals for this part: 💻 one on your laptop** (not logged in to the server) **and 🖥️ one `ssh`'d into your server.** Step 14 adds a third.

Your code goes in **your home folder** on the server, not in `/var/www/html`. Everything in `/var/www/html` is handed out by nginx to anyone who asks, so your code would be public.

## 11. Make room 🖥️ SERVER

```bash
ssh yourfirstname@your-server-address
```

A **port** is a numbered door on a server that a program listens on. **Only one program can use a port at a time.** In class you left a test app called `hello` running on port 3000 with pm2, and your jokes API needs port 3000 too. If `hello` is still there, your API won't start (`EADDRINUSE: address already in use :::3000`). `pm2 delete` stops it and removes it from pm2's list:

```bash
pm2 delete hello
pm2 list
```

`pm2 list` should now show an empty table. (`Process or Namespace hello not found` means it was already gone. That's fine.)

Make a folder for your API:

```bash
mkdir ~/jokes
```

## 12. Copy the files up 💻 LAPTOP

In your **laptop** terminal, go to your finished Lab 5 `Jokes` folder (your path may be different):

```bash
cd ~/cs333/cs333-lab5/Jokes
ls
```

You should see `index.js`, `package.json` and `package-lock.json`. Copy those three files up:

```bash
scp index.js package.json package-lock.json yourfirstname@your-server-address:~/jokes/
```

**Don't copy `node_modules`.** It's big and it was built for your laptop. In step 13 the server builds its own from `package.json`.

**WinSCP instead?** Log in like in class (slide 8), go to `/home/yourfirstname/jokes` on the right, your Lab 5 `Jokes` folder on the left, and drag over **only** those three files.

## 13. Install 🖥️ SERVER

```bash
cd ~/jokes
ls
npm install
```

`ls` should show your three files. `npm install` reads `package.json` and downloads Express onto the server.

## 14. Run it once, by hand 🖥️ SERVER

```bash
node index.js
```

It says `Listening on http://localhost:3000` and keeps running, so this terminal is busy. Open a **third** terminal on your laptop, `ssh` in again, and type:

```bash
curl http://localhost:3000/random
```

`curl` sends a GET request from the terminal, like REST Client does. You should get a joke back.

Now go back to the terminal running `node index.js` and press **Ctrl+C**. Send the `curl` again.

**Question 8:** what does `curl` say now? Why? What would happen to your API if you closed that terminal, or your laptop?

You can close the third terminal now.

---

# Part 4: Keep it running with pm2 🖥️ SERVER

**pm2** is a process manager: it starts your Node app, keeps it running when you log out, and restarts it if it crashes.

## 15. Start your API with pm2

You installed pm2 in class. Check that it's there:

```bash
pm2 -v
```

(If it says `pm2: command not found`, run `sudo npm install -g pm2`, then `pm2 -v` again.)

Make sure you're in `~/jokes`, then:

```bash
pm2 start index.js --name jokes
pm2 list
curl http://localhost:3000/random
```

**Question 9:** what is the `status` of `jokes`? What does the `↺` column count?

It works, and no terminal is busy running it.

## 16. Logs

```bash
pm2 logs jokes --lines 20
```

**Question 10:** what's in the log? Which line of `index.js` printed it? (Press **Ctrl+C** to stop watching the log. That only stops the log, not your API.)

Other pm2 commands you'll use:

| Type this | It does |
| --- | --- |
| `pm2 list` | shows what pm2 is running |
| `pm2 logs jokes` | shows what your app prints (`console.log` and errors) |
| `pm2 restart jokes` | restarts it (do this after you copy up new code) |
| `pm2 stop jokes` | stops it |
| `pm2 delete jokes` | stops it and removes it from the list |

## 17. Survive a reboot

```bash
pm2 save
pm2 startup
```

`pm2 startup` prints **one** line that starts with `sudo env PATH=...`. Copy that whole line, paste it and run it. Now pm2 starts your API again whenever the server restarts.

---

# Part 5: nginx in front 🖥️ SERVER

Your API listens on port **3000**. Web browsers use port **80** for `http://`. **nginx** listens on port 80. Right now it hands out the files in `/var/www/html`. We'll also tell it: anything that starts with `/api/`, pass on to port 3000. That's called a **reverse proxy**.

```
browser  --->  nginx (port 80)  --- /api/... --->  your jokes API (port 3000)
                     |
                     +--- anything else ---> files in /var/www/html
```

## 18. Check nginx

```bash
systemctl status nginx --no-pager | head -3
```

It should say `active (running)`. Open `http://your-server-address` in your **laptop's** browser.

**Question 11:** what page do you see? Where on the server does that page live?

**TODO:** also try `http://your-server-address:3000/random` in your browser. **Question 12:** what happens? (Either answer is fine; just say what you see.)

## 19. Send /api/ to your API

In class you added a `location /api/` block to nginx for your test app. It sends `/api/` to whatever runs on port 3000, and that's your jokes API now. Check it's there:

```bash
grep -A2 "location /api/" /etc/nginx/sites-available/default
```

If it prints the three lines below, **don't add them again** (two `location /api/` blocks is an error). Skip to the `nginx -t` test below. If it prints nothing, add the block:

Open nginx's settings for your site:

```bash
sudo nano /etc/nginx/sites-available/default
```

Use the arrow keys to find the `location / {` block (a few lines, ending with `}`). Right **after** its `}`, type this:

```nginx
	location /api/ {
		proxy_pass http://localhost:3000/;
	}
```

Save: **Ctrl+O**, **Enter**, then **Ctrl+X** to leave nano. (Using vim instead? `i` to type, then Esc and `:wq`.)

Check what you typed, then load it:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

`nginx -t` tests the file. If it says anything other than `syntax is ok` and `test is successful`, read the line number it gives you, open the file again and fix it. **Don't reload until the test passes.**

Open `http://your-server-address/api/random` in your laptop's browser. 🎉 Your API is on the internet.

**Question 13:** the browser asked for `/api/random`, but your Express route is `/random`. Which part of the nginx setting takes `/api` off? (Hint: try it without the last `/` in `proxy_pass`, test, reload, and see what happens. Put it back after.)

---

# Part 6: Use it from your laptop 💻 LAPTOP

Type `exit` in your server terminal. Your API keeps running.

## 20. REST Client, pointed at your server

Open `server.http` in this repo. On the second line, put your server's address in place of `your-server-address`. `{{base}}` in each request is replaced with that line, so you only type the address once.

Send requests **1**, **2** and **3**. They are the same requests as in Lab 5, now going over the internet.

## 21. Where did my joke go?

Send request **4** (POST), then request **5** with the id your joke got. Then:

```bash
ssh yourfirstname@your-server-address
pm2 restart jokes
exit
```

Send request **5** again.

**Question 14:** what happened to your joke? Why? Think about Part 2: your fruits were still there after `app.js` stopped. How would MongoDB fix your jokes? (That's Lab 7.)

**Question 15:** in your own words, what does pm2 do, and what does nginx do? Why do we need both?

## Changed your code?

To update the API on your server: copy the changed file up again from your **laptop** (`scp index.js yourfirstname@your-server-address:~/jokes/`, or drag it in WinSCP), then on the **server** run `pm2 restart jokes`. If you added a package, run `npm install` on the server before the restart.

---

# Hand it in

22. Push this repo to **your own GitHub repo**: this README with your answers, `fruitsProject` (`app.js` and `package.json`, not `node_modules`), and `server.http` with your address.
23. Submit in Moodle: your GitHub repo link **and** your API link, `http://your-server-address/api/random`. Leave your API running so I can test it.

---

## Stuck?

| You see | What's going on | Fix |
| --- | --- | --- |
| `mongosh: command not found` / `not recognized` | mongosh isn't installed, or isn't on your Path | Lab 5 Part 3: install mongosh, add it to Path, open a **new** terminal |
| `MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27017` | MongoDB isn't running on your laptop | Mac: `brew services start mongodb-community`. Windows: start the **MongoDB** service in Services |
| `MongooseServerSelectionError: connect ECONNREFUSED ::1:27017` | You wrote `localhost` in the connect address | Use `mongodb://127.0.0.1:27017/fruitsDB` |
| `Cannot use import statement outside a module` | `"type": "module"` is missing from `package.json` | In `fruitsProject`: `npm pkg set type=module` |
| `Cannot find package 'mongoose'` | You didn't `npm install mongoose` in `fruitsProject` | `cd fruitsProject`, then `npm install mongoose` |
| `scp: ... No such file or directory` | You're not in your `Jokes` folder on the laptop, or `~/jokes` doesn't exist on the server | `ls` on the laptop should show `index.js`; `mkdir ~/jokes` on the server |
| `Permission denied (publickey)` | Wrong username, or your key isn't on the server | It's `yourfirstname@...`. Then ask in the Slack thread |
| `Cannot find package 'express'` (on the server) | You didn't `npm install` in `~/jokes` | `cd ~/jokes`, then `npm install` |
| `EADDRINUSE: address already in use :::3000` | Something else is on port 3000: the `hello` test app from class, or your API is already running (by hand or in pm2) | `pm2 list`; `pm2 delete hello`; Ctrl+C the one you started by hand |
| `pm2: command not found` | pm2 isn't installed | `sudo npm install -g pm2` |
| `pm2 list` says `errored`, or `↺` keeps going up | Your app crashes when it starts | `pm2 logs jokes` shows the error |
| `curl: (7) Failed to connect to localhost port 3000` | Your API isn't running | `pm2 list`, then `pm2 restart jokes` |
| `nginx -t` says `unexpected "}"` or `unknown directive` | A typo, or a missing `;` or `}` | Check the line number it gives you; every setting ends with `;` |
| `502 Bad Gateway` | nginx is working, but your API isn't running | `pm2 list` and `pm2 logs jokes` |
| `nginx -t` says `duplicate location "/api/"` | You added the block again; it was already there from class | Open the file and delete one of the two `location /api/ { ... }` blocks |
| `404 Not Found` from nginx | The `location /api/` block isn't there, or nginx wasn't reloaded | Check the file, `sudo nginx -t`, `sudo systemctl reload nginx` |
| `Cannot GET /api/random` (from Express) | The last `/` is missing in `proxy_pass` | `proxy_pass http://localhost:3000/;` |
| The browser can't connect at all | Wrong address, or the server isn't on | Check your server's address; `ping your-server-address` |
