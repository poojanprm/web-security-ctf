# 🚩 Web Security CTF — Write-up

> ⚠️ **SPOILER WARNING**
>
> This write-up contains the complete solution and final flag.
>
> Try solving the challenge yourself before reading further.

---

## 1. Initial Reconnaissance

The challenge starts with a simple **Internal Portal** containing an access-key input and a `Continue` button.

The first step is to inspect the page using the browser's Developer Tools.

Useful areas to investigate include:

- Page source
- JavaScript
- Browser Console
- URL parameters
- LocalStorage

---

## 2. Analyze the JavaScript

Inspecting the JavaScript reveals an object containing several route names:

```javascript
const routes={
    a:"home",
    b:"dashboard",
    c:"portal",
    d:"console",
    e:atob("YXJjaGl2ZQ==")
};

The value:

YXJjaGl2ZQ==

is Base64 encoded.

Decoding it gives:

archive

Therefore:

routes.e = archive

This gives us an important clue about the hidden route.

3. Identify the Decoy Flags

The JavaScript also contains several flag-like values:

const fakeFlags=[
    "FLAG{nice_try}",
    "FLAG{almost}",
    "FLAG{fake_admin}"
];

These are decoy flags and are not the final answer.

For example, accessing the admin page displays:

FLAG{fake_admin}

This is intentionally designed to mislead the player.

4. Analyze the Login Function

Next, inspect the login() function:

function login(){


    const input=document.getElementById("key").value.trim();


    if(hash(input)==="4b3b7c6"){
        localStorage.setItem("employee","verified");
        document.getElementById("msg").innerHTML=
        "<span style='color:lightgreen'>Access cached.</span>";
    }else{
        document.getElementById("msg").innerHTML=
        "<span style='color:tomato'>Invalid key.</span>";
    }
}

The application hashes the supplied access key and compares the result against:

4b3b7c6

When the comparison succeeds, the application stores:

employee = verified

in LocalStorage.

This is an important client-side security observation because LocalStorage is controlled by the browser user.

5. Inspect the URL Parameters

The application reads two URL parameters:

const params=new URLSearchParams(location.search);


const page=params.get("page");
const token=params.get("token");

This means the application accepts:

page
token

as user-controlled URL parameters.

6. Find the Archive Conditions

The important part of the JavaScript is the final conditional:

if(
    page===routes.e &&
    localStorage.getItem("employee")==="verified" &&
    token==="alpha-42"
){
    ...
}

From the previous investigation:

routes.e = archive

Therefore, the required conditions are:

page = archive
employee = verified
token = alpha-42

All three conditions must be satisfied.

7. Access the Archive

After satisfying the required client-side conditions, the application displays the Archive page:

Archive
Access Granted.

The application then reveals the final flag.

🏁 Final Flag
FLAG{browser_exploration_master}
🔐 Security Lessons

This CTF demonstrates several important Web Security concepts.

Client-Side Authentication

Authentication logic implemented entirely in client-side JavaScript can be inspected by the user.

Sensitive authentication and authorization decisions should be enforced server-side in real applications.

LocalStorage

The challenge stores an authentication state in LocalStorage:

employee = verified

LocalStorage is controlled by the browser user and should not be treated as a trusted security boundary.

URL Parameters

The application uses URL parameters to control application behavior:

page
token

URL parameters are user-controlled input and should never be blindly trusted for sensitive authorization decisions.

Base64 Encoding

The route value:

YXJjaGl2ZQ==

is Base64 encoded.

Base64 is encoding, not encryption.

Encoded values can easily be decoded and should not be used to protect sensitive information.

Client-Side Secrets

The challenge contains important values directly inside JavaScript.

Since frontend JavaScript is delivered to the browser, users can inspect it.

Real applications should never rely on hiding security-sensitive information inside client-side source code.

🧠 Intended Learning Path

The challenge can be solved by following this investigation flow:

Initial Reconnaissance
        ↓
Inspect JavaScript
        ↓
Identify Base64 value
        ↓
Decode "archive"
        ↓
Identify decoy flags
        ↓
Analyze login logic
        ↓
Inspect LocalStorage
        ↓
Inspect URL parameters
        ↓
Identify token requirement
        ↓
Access archive
        ↓
Capture the flag
🏆 Difficulty

Beginner / Intermediate

The challenge is primarily focused on browser-based reconnaissance and client-side Web Security concepts.

It is intended as a learning-focused CTF rather than an advanced exploitation challenge.

📌 Project Status

Version 1.0 — Completed

Future versions may introduce additional challenge stages and more advanced Web Security concepts.
    d:"console",
    e:atob("YXJjaGl2ZQ==")
};
