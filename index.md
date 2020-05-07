---
paginate: false
theme: gaia
---
<!-- _class: lead invert -->
![bg left:40% 70%](https://raw.githubusercontent.com/Bestowinc/darkroom/master/darkroomlogo_mini.svg)

# **Darkroom**

A contract testing tool built in Rust.

---

# Issues with some current test flows/pitfalls
- At some point a system needs to be tested as it's going to be used
* test setups are optimized for convenience
* a lot of work goes into making tests more convenient
* at a certain threshold, one is no longer testing how the real system works,
rather a mock of the system

---
# What are the pain points with our current e2e tooling
- Engineers focus more on scaffolding for tests rather than on composition
- anemic
---

# Why Darkroom
* #### Solves **highest**  order of complexity of a system (the real thing) with simple solutioning
* #### excludes front-end **𝓡𝓘𝓟 ⚰️**
* #### Works best testing a fully working API.
* #### Attempts to make make as few assumptions as possible about how a system will react.
---

# filmReel
![bg left:40% 70%](https://raw.githubusercontent.com/Bestowinc/filmReel/master/images/filmreel.svg)

- A format for encoding API state flow expectations.

- The filmReel specification aims to move the source of truth for how an API should behave from the service itself and into a series of linear state diagrams. This allows one to set loosely coupled and human readable expectations for an API.


---
<style scoped>
section { columns: 2; }
ul { font-size: 0.70em; }
</style>

```json
{
  "protocol": "HTTP",
  "cut": {
    "from": [
      "FIRST_NAME"
    ],
    "to": ".response.body.last_name"
  },
  "request": {
    "body": {
      "first_name": "${FIRST_NAME}"
    },
    "uri": "GET /last_name/by_first_name"
  },
  "response": {
    "body": {
      "last_name": "${LAST_NAME}"
    },
    "status": 200
  }
}

```


# Frame
  * A JSON file ending with the suffix `.fr.json`

    - *Cut Instruction Set* - A JSON object holding Read and Write instructions that push and pull variables `"from"` and `"to"` the *Cut Register* through [*Cut operations*](cut.md#cut-operation).
    - **Request** - A [JSON object](https://en.wikipedia.org/wiki/JSON#Data_types_and_syntax) that fully defines how the *Frame* payload is built and sent.
    - **Response** - A JSON object that defines the expectations for the contents of a response message.

---
<!-- _class: invert -->
<style scoped>
section { columns: 2; }
</style>

# Reel 
* the sum of all *Frame*s in a directory that share the same *Reel name*.
```sh
user_reel
├── usr.01e.createuser.fr.json
├── usr.01s.createuser.fr.json
├── usr.01se.createuser.fr.json
├── usr.02s.changeaddress.fr.json
├── usr.03s.changebirthdate.fr.json
├── usr.04s.changebirthlocation.fr.json
├── usr.06e.changeemail.fr.json
├── usr.06s.changeemail.fr.json
├── usr.07s.confrimemail.fr.json
├── usr.08s.changename.fr.json
└── usr.09s.getuser.fr.json
```

---

<!-- _class: invert -->
<style scoped>
section { columns: 2; }
ul { font-size: 0.70em; }
</style>

# Reel membership

```
┌─────────── Reel name              // usr
│   ┌─────── Sequence number        // 01
│   │ ┌───── Frame type             // se
│   │ │  ┌── Command name           // createuser
▼   ▼ ▼  ▼
usr.01se.createuser.fr.json
                    ▲
                    └─ Frame suffix // .fr.json
```

- ##### **Reel name** - describes the functiality of the entire flow (subjective)
- ##### **Sequence number** - a number representing a particular step in an object's state transition:
- ##### **Frame type** - defined by the return body of a *Frame*:
- ##### **Command name** - describes the functionality of the *Frame* JSON file

---
<!-- _class: invert -->
<style scoped>
ul { font-size: 0.80em; }
</style>

#### **Frame type** - defined by the return body of a Frame

* #### **Error Frame** - Frame with a return body holding an error status code.
  - represented by the letter `e`.
  - precedes a **Success Frame** sharing the same whole sequence number.
* #### **Success Frame** - Frame with a return body holding a success status code:
  - represented by the letter `s`.
  - Typically indicates a state transition.
  - A sequence number should aim to be referenced by only one success Frame.
* #### **Post Success Error Frame** aka **P.S. Error Frame** -  an Error Frame that must be preceded by a Success Frame sharing the same whole sequence number:
  - represented by the letters `se`.
---

  ```
  user_reel
  ├── usr.01e_1.createuser.fr.json // no first name
  ├── usr.01e_2.createuser.fr.json // no email
  └── usr.01s.createuser.fr.json   // user created successfully
  ```
  **Ex**: A *sequence number* with more than one **Error Frame**.

* **Whole sequence number** - the float representation of a sequence number indicating an object state.
* ex: `01_e1` => 1.1,  `01_e2` => 1.2,

---

<!-- _class: invert -->
<style scoped>
ul { font-size: 0.70em; }
</style>

# Cut 
- a JSON file starting with a **Reel name** and ending with the suffix `.cut.json`.

```json
// usr.cut.json
{
  "FIRST_NAME": "Luke",
  "RESPONSE": "Luke is your first name"
}
```
### A **Cut** allows data to be stored and propagated to Frames in a **Reel** sequence using instructions held in a Frame's **Cut Instruction Set**
---
<style scoped>
ul { font-size: 0.70em; }
section { columns: 2; }
</style>

```json
// usr.cut.json
{
  "USER_ID": "PeterJackson_h8tr_2",
  "STATUS": 200
}
```
  - An example *Frame* that logs out a user using a *Cut Variable* to build the request URI.

```json
{
  "protocol": "HTTP",
  "cut": {
    "from": [
      "USER_ID",
      "STATUS"
    ]
  },
  "request": {
    "entrypoint": "https://www.lotrfanfic.com",
    "body": {},
    "uri": "POST /logout/${USER_ID}"
  },
  "response": {
    "body": "User:${USER_ID} logged out",
    "status": "${STATUS}"
  }
}
```

---
```
{
  "protocol": "HTTP",
  "cut": {
    "from": [
      "USER_ID",
      "STATUS"
    ]
  },
  "request": {
    "entrypoint": "https://www.lotrfanfic.io",
    "body": {},
    "uri": "POST /logout/${USER_ID}"
  },
  "response": {
    "body": "User:${USER_ID} logged out",

    "status": "${STATUS}"
  }
}
```
---
```
{
  "protocol": "HTTP",
  "cut": {
    "from": [
      "USER_ID",
      "STATUS"
    ]
  },
  "request": {
    "entrypoint": "https://www.lotrfanfic.io",
    "body": {},
    "uri": "POST /logout/PeterJackson_h8tr_2"
  },
  "response": {
    "body": "User:PeterJackson_h8tr_2 logged out",

    "status": 200
  }
}
```
---
```json
{
  "protocol": "HTTP",
  "cut": {
    "from": [
      "USER_ID",
      "STATUS"
    ]
  },
  "request": {
    "entrypoint": "https://www.lotrfanfic.io",
    "body": {},
    "uri": "POST /logout/PeterJackson_h8tr_2"
  },
  "response": {
<   "body": "User:PeterJackson_cr1t1c logged out",
>   "body": "User:PeterJackson_h8tr_2 logged out",
    "status": 200
  }
}
```
---
#### httpbin.org
- POST/IP demo
---
#### Stripe
- introduce sops for our stripe key in a bash script
- Create a stripe token
- Create a stripe customer

---
#### CRS
- Create customer flow in QA
---
#### PAS
- Bind Policy flow
