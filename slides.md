---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /assets/intro.jpg
# some information about your slides (markdown enabled)
title: Software Development | Foundations
info: |
  ## Software Development | Foundations
# apply unocss classes to the current slide
class: text-left
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Authentication via Passport
Back-End Development - part 12/12
- [ ] Continuing with JWT and EJS
- [ ] Create login and register page
- [ ] Passport

<div class="abs-br m-6 text-xl">
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
-->

---
transition: slide-left
---

# Recap

- [My Github Repo](https://github.com/avcoder/unit03-backend) of backend up to this point
- if Zoom crashes goto: https://meet.google.com/xzx-rria-pvs
- Review: we installed EJS via `npm i ejs`
- then in `server.js`:
  ```js
  import path from "path";
  import { fileURLToPath } from "url";

  app.set("view engine", "ejs"); // place after app = express() statement
  const __filename = fileURLToPath(import.meta.url);
  const __dirname = path.dirname(__filename);
  app.set("views", path.join(__dirname, "views")); 

  app.get("/", (req, res) => {
    // res.send("🚚 Welcome to the Food Truck!");
    res.render("login", { title: "hello world", user: "Al" }); // passing object that login.ejs will use
  });
  ```
- then created `/src/views/login.ejs`
  ```html
  <!DOCTYPE html><html><head><title><%= title %></title></head><body><h1>Welcome, <%= user %>!</h1></body></html>
  ```


---
transition: slide-left
---

# Exercise: JWT (pg.12)
Before we use our auth.js, create a new /user route

1. Create all folders/files necessary to create a new user route
1. Ensure you have completed the following:
   - `router.post("/user", userController.createUser);`
   - `/src/controllers/userController.js`
   - `/src/handlers/user.js`
   - `/src/models/user.js` 
      - should have only 2 fields: username `{ String, required, unique}`
      - and password `{ String, required }`
   - Compass should now show new `users` collection within `foodtruck` database
   - Test via Postman: see if POSTing username/password creates a new user in mongoDB (may have to remove our earlier `protect` function to test this)
1. Next - incorporate the hashing functionality...Goto next slide


---
transition: slide-left
---

# Exercise continued: JWT (pg.13)

1. In `/src/handlers/user.js`:
  ```js
  import User from "../models/user.js"; // will handle DB interactions
  import { hashPassword, comparePasswords } from "../modules/auth.js";

  const createUser = async ({ username, password }) => {
    const user = await User.create({
      username,
      password: await hashPassword(password),
    });

    return user;
  };

  export default {
    createUser,
  };
  ```
1. Test via POSTMAN: did it create a new user entry in mongoDB with the password being hashed?
1. Did the server return `201` and also a token? (click "Body" tab to the left of `201` to check) 
1. Try registering again using existing username in database?  Gracefully [handle the error](https://unit03-lesson10.netlify.app/presenter/13).

---
transition: slide-left
---

# JWT: Login (pg.1)
Let's create sign-in functionality

1. When users login, thereby sending username/password, how will you authenticate? (what exactly will you be comparing?)
1. Let's create a `/login` route
1. In `server.js`
  ```js
  import userController from "./controllers/userController.js";

  app.post("/user", userController.createUser);
  app.post("/login", userController.loginUser);
  ```

---
transition: slide-left
---

# JWT: Login (pg.2)

1. in `/src/controlllers/userController.js`
  ```js
  const loginUser = async (req, res) => {
    const { username, password } = req.body;
    const user = await userHandler.loginUser({ username, password });

    if (!user) {
      res.status(401);
      res.json({ message: "invalid password" });
      return;
    }

    const token = createJWT(user);
    res.status(200).json({ token });
  };
  ```

---
transition: slide-left
---

# JWT: Login (pg.3)

1. in `/src/handlers/user.js`
  ```js
  const loginUser = async ({ username, password }) => {
    const user = await User.findOne({ username });
    const isValid = await comparePasswords(password, user.password);

    return isValid ? user : null;
  };

  export default {
    createUser,
    loginUser,
  };
  ```
- Test via Postman: see if POSTing username/password via `/login` signs in a user; IF successful, did you see a token?

---
transition: slide-left
---

# JWT: Login (pg.4)

1. Let's put back our `protect` guard function. In `server.js`
  ```js
  import { protect } from "./modules/auth.js";

  app.use("/api", protect, router);
  ```
1. Test it
  - create a new user
  - login using user credentials but remember to also pass in authorization token
  - Try GETting `/api/order` -- Does it work?
  - Try GETting `/api/order` without the token?  Does it fail?




---
layout: image-right
transition: slide-left
image: /assets/bos.png
backgroundSize: 500px 300px
class: text-left
---

# 10 minute break

🍦 Cool Tips, Trends and Resources:
- 🪗 [Array Grouping method](https://x.com/matanbobi/status/1693245513099776012)
- 🎮 [React powered video games](https://x.com/cpojer/status/1704562739904135393)
- 🏇 [Animation using GSAP](https://gsap.com/showcase/)
- 📓 [Soft skill books](https://addyosmani.com/blog/soft-skills-books/?ref=syntax)

<br>
<hr>
<br>

- 🧪 [Enter anonymous lab questions](https://docs.google.com/forms/d/e/1FAIpQLSevvGARdHQikso-uLqFCO481MABKE5HofuSrlzEPMNQ2ZLykw/viewform?usp=dialog)
- ℹ️ [Course feedback survey](https://circuitstream.typeform.com/to/ZoyYk7px#course_id=SoftwareAN&instructor=9514)

<!-- 
- take attendance
-->

---
transition: slide-left
---

# Refactor / Tidy

- Move our get `/` route into our routes file as well
- Change `app.use("/api", router)` to `app.use("/", router);`
- Cut/paste our `app.get("/"...)` into our `router.js` file as `router.get("/"...)`
- Refactor all our `/api/order` routes to now be prepended with `/api/whatever` 
- Insert 2 new routes related:
  ```js
  router.get("/user", userController.createUser);
  router.get("/login", userController.loginUser);
  ```
- `import userController from "./controllers/userController.js"`

---
transition: slide-left
--- 

# Passport (pg.1)

- https://www.passportjs.org/packages/
- `npm i passport passport-local-mongoose express-session connect-mongo`
  ```js
  import passport from 'passport'; // in server.js
  import session from 'express-session';
  import MongoStore from 'connect-mongo';
  import User from "./models/user.js";

  // passport-local-mongoose provides .createStrategy() which uses passport-local under the hood
  passport.use(User.createStrategy()); 
  passport.serializeUser(User.serializeUser()); // upon successful login writes user ID to session
  passport.deserializeUser(User.deserializeUser()); // called on every request that has a session which then reads user ID from session, and populates req.user
  
  app.use(session({  
    secret: process.env.PASSPORT_SECRET, // remember to input this in .env
    key: process.env.PASSPORT_COOKIE_KEY,  // remember to input this in .env
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({ mongoUrl: process.env.DB_CONN}) // store sessions in mongoDB (not in memory)
  }));

  app.use(passport.initialize()); 
  app.use(passport.session());
  ```

---
transition: slide-left
---

# Passport (pg.2)

- Create new route to:
  ```js
  router.get("/register", userController.registerForm);
  router.get("/login", userController.loginForm);
  ```
- Create new controller functions in `/src/controllers/userController.js`
  ```js
  const registerForm = async (req, res) => {
    res.render("register", { title: "Register" });
  };

  const loginForm = async (req, res) => {
    res.render("login", { title: "Login" });
  };
  ```
- Create new login.ejs and register.ejs pages

---
transition: slide-left
---

# Passport (pg.3)

- in `/src/models/user.js`:
  ```js
  import mongoose from "mongoose";
  import plm from "passport-local-mongoose"; 

  const userSchema = mongoose.Schema({
    username: {
      type: String,
      required: true,
      unique: true,
    },
    password: {
      type: String,
      required: true, // hashed password
    },
  });

  // below adds username, hash and salt fields,
  // and will store username, hashed password and salt value
  userSchema.plugin(plm); 

  export default mongoose.model("user", userSchema);
  ```
- Test it


---
transition: slide-left
---

# Exercise

1. Try incorporating the signup, the login, and the GET orders screen using the front end of [foodtruck-app](https://github.com/avcoder/foodtruck-app)
2. Fill in all 6 blanks in order to make this work
3. If successful, you should be able to:
   - sign in (which creates new user in db)
   - be redirected to login page
   - upon successful login
   - should get a Token which is stored in localStorage (see Application tab > Local Storage > http://localhost)
   - be redirected to GET / page 
   - view all the existing orders


---
transition: slide-left
---

# Homework

- Make a To-Do List App with MongoDB [see instructions in LMS](https://courses.circuitstream.com/d2l/le/lessons/9514/topics/49825)