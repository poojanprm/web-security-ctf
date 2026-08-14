
# 🚩 Web Security CTF — Write-up


> ⚠️ **SPOILER WARNING**
>
> This write-up contains the complete solution and final flag.
>
> Try solving the challenge yourself before reading further.


---


## 1. Initial Reconnaissance


The challenge starts with a simple **Internal Portal** containing an access-key input and a **Continue** button.


The first step is to inspect the application using the browser's Developer Tools.


Useful areas to investigate include:


- Page Source
- JavaScript
- Browser Console
- URL Parameters
- LocalStorage


---


## 2. Analyze the JavaScript


Inspecting the JavaScript reveals an object containing several route names:


```javascript
const routes = {
    a: "home",
    b: "dashboard",
    c: "portal",
    d: "console",
    e: atob("YXJjaGl2ZQ==")
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

const fakeFlags = [
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


    const input = document.getElementById("key").value.trim();


    if(hash(input) === "4b3b7c6"){
        localStorage.setItem("employee","verified");


        document.getElementById("msg").innerHTML =
            "<span style='color:lightgreen'>Access cached.</span>";
    }else{
        document.getElementById("msg").innerHTML =
            "<span style='color:tomato'>Invalid key.</span>";
    }
}

The application does not compare the access key directly.

Instead, the entered value is passed through a custom hash() function.

The target hash is:

4b3b7c6
5. Analyze the Hash Function

Inspecting the JavaScript reveals the custom hashing function:

function hash(str){
    let h = 0;


    for(let i = 0; i < str.length; i++){
        h = ((h << 5) - h) + str.charCodeAt(i);
        h |= 0;
    }


    return Math.abs(h).toString(16);
}

The important operation is:

h = ((h << 5) - h) + str.charCodeAt(i);

Since:

h << 5

is equivalent to:

h × 32

subtracting h gives:

h × 31

Therefore, the hash can be understood as:

h = h × 31 + character_code

with the final result converted to hexadecimal.

6. Reproduce the Hash

Because the hashing function is exposed in the client-side JavaScript, it can be copied into the browser Console.

Run:

function hash(str){
    let h = 0;


    for(let i = 0; i < str.length; i++){
        h = ((h << 5) - h) + str.charCodeAt(i);
        h |= 0;
    }


    return Math.abs(h).toString(16);
}

Now test the access key:

hash("SGycd");

The result is:

"4b3b7c6"

This matches the target value.

Therefore, the access key is:

SGycd

Enter this value into the Access Key field.

7. Important Hash Observation

The custom hash is not a cryptographically secure hash.

Different inputs can potentially produce the same hash value because the algorithm allows collisions.

Therefore, the objective is to reproduce the function and find an input that produces the required target hash.

This demonstrates why a simple client-side custom hash should not be used as a real authentication mechanism.

8. Inspect LocalStorage

After successful authentication, the application executes:

localStorage.setItem("employee","verified");

The application therefore stores:

employee = verified

in LocalStorage.

The application also displays:

Access cached.

This is an important client-side security observation because LocalStorage is controlled by the browser user.

9. Inspect the URL Parameters

The application reads two URL parameters:

const params = new URLSearchParams(location.search);


const page = params.get("page");
const token = params.get("token");

The application therefore accepts:

page
token

as user-controlled URL parameters.

These values become important for accessing the hidden Archive page.

10. Find the Archive Conditions

The important part of the JavaScript is:

if(
    page === routes.e &&
    localStorage.getItem("employee") === "verified" &&
    token === "alpha-42"
){
    // Archive access
}

From the earlier investigation:

routes.e = archive

Therefore, the required conditions are:

page = archive
employee = verified
token = alpha-42

All three conditions must be satisfied.

11. Construct the Required URL

The required page is:

archive

The required token is:

alpha-42

Therefore, the URL parameters need to contain:

?page=archive&token=alpha-42

The correct access key has already created:

employee = verified

in LocalStorage.

With all three conditions satisfied, the Archive page becomes accessible.

12. Access the Archive

With the required conditions satisfied:

page = archive
employee = verified
token = alpha-42

the application displays:

Archive
Access Granted.

The application then reveals the final flag.

🏁 Final Flag
FLAG{browser_exploration_master}
🔐 Security Lessons
Client-Side Authentication

Authentication logic implemented entirely in client-side JavaScript can be inspected by the user.

In a real-world application, sensitive authentication and authorization decisions should be enforced server-side.

Client-Side Hashing

The challenge exposes both the hashing algorithm and target hash in the JavaScript source.

Because the algorithm is available to the user, it can be reproduced and analyzed.

Client-side hashing should not be treated as a secure authentication mechanism.

Hash Collisions

The custom hash is not cryptographically secure and can have collisions.

Matching a custom client-side hash is not equivalent to securely authenticating a user.

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

is Base64 encoded and decodes to:

archive

Base64 is encoding, not encryption.

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
Decode "YXJjaGl2ZQ==" → "archive"
        ↓
Identify decoy flags
        ↓
Analyze login function
        ↓
Find hash() function
        ↓
Identify target hash: 4b3b7c6
        ↓
Reproduce the hash in Browser Console
        ↓
Test SGycd
        ↓
hash("SGycd") = 4b3b7c6
        ↓
Enter access key
        ↓
employee = verified
        ↓
Inspect URL parameters
        ↓
page = archive
        ↓
token = alpha-42
        ↓
Access Archive
        ↓
Capture the flag
🎯 Key Findings
Finding	Value
Hidden Route	archive
Encoded Route	YXJjaGl2ZQ==
Target Hash	4b3b7c6
Access Key	SGycd
LocalStorage Key	employee
LocalStorage Value	verified
Required Page	archive
Required Token	alpha-42
Final Flag	FLAG{browser_exploration_master}
🏆 Difficulty

Beginner / Intermediate

The challenge is primarily focused on browser-based reconnaissance and client-side Web Security concepts.

It is intended as a learning-focused CTF rather than an advanced exploitation challenge.

📌 Project Status
Version 1.0 — Completed




Future versions may introduce additional challenge stages and more advanced Web
