 Web Security CTF — Write-up
nn SPOILER WARNING
This write-up contains the complete solution and final flag.
Try solving the challenge yourself before reading further.
1. Initial Reconnaissance
The challenge starts with a simple Internal Portal containing an access-key input and a Continue button.
The first step is to inspect the application using the browser's Developer Tools.
Useful areas to investigate include:
• Page Source
• JavaScript
• Browser Console
• URL Parameters
• LocalStorage
2. Analyze the JavaScript
const routes = {
a: "home",
b: "dashboard",
c: "portal",
d: "console",
e: atob("YXJjaGl2ZQ==")
};
The value YXJjaGl2ZQ== is Base64 encoded. Decoding it gives archive. Therefore, routes.e =
archive.
3. Identify the Decoy Flags
const fakeFlags = [
"FLAG{nice_try}",
"FLAG{almost}",
"FLAG{fake_admin}"
];
These are decoy flags and are not the final answer. For example, accessing the admin page displays
FLAG{fake_admin}.
4. Analyze the Login Function
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
The application does not compare the access key directly. Instead, the entered value is passed through a
custom hash() function. The target hash is 4b3b7c6.
5. Analyze the Hash Function
function hash(str){
let h = 0;
for(let i = 0; i < str.length; i++){
h = ((h << 5) - h) + str.charCodeAt(i);
h |= 0;
}
return Math.abs(h).toString(16);
}
The important operation is h = ((h << 5) - h) + str.charCodeAt(i). Since h << 5 is equivalent
to h × 32, subtracting h gives h × 31. Therefore, the hash can be understood as h = h × 31 +
character_code, with the final result converted to hexadecimal.
6. Reproduce the Hash
Because the hashing function is exposed in the client-side JavaScript, it can be copied into the browser
Console.
function hash(str){
let h = 0;
for(let i = 0; i < str.length; i++){
h = ((h << 5) - h) + str.charCodeAt(i);
h |= 0;
}
return Math.abs(h).toString(16);
}
hash("SGycd");
The result is:
"4b3b7c6"
This matches the target value. Therefore, the access key is SGycd.
7. Important Hash Observation
The custom hash is not a cryptographically secure hash. Different inputs can potentially produce the same
hash value because the algorithm allows collisions. Therefore, the objective is to reproduce the function and
find an input that produces the required target hash.
8. Inspect LocalStorage
localStorage.setItem("employee","verified");
After successful authentication, the application stores employee = verified in LocalStorage and displays
Access cached.
9. Inspect the URL Parameters
const params = new URLSearchParams(location.search);
const page = params.get("page");
const token = params.get("token");
The application accepts page and token as user-controlled URL parameters.
10. Find the Archive Conditions
if(
page === routes.e &&
localStorage.getItem("employee") === "verified" &&
token === "alpha-42"
){
// Archive access
}
Since routes.e = archive, the required conditions are:
• page = archive
• employee = verified
• token = alpha-42
11. Construct the Required URL
?page=archive&token=alpha-42
The correct access key has already created employee = verified in LocalStorage. With all three
conditions satisfied, the Archive page becomes accessible.
12. Access the Archive
Archive
Access Granted.
The application then reveals the final flag.
n Final Flag
FLAG{browser_exploration_master}
n Security Lessons
Client-Side Authentication
Authentication logic implemented entirely in client-side JavaScript can be inspected by the user. In a
real-world application, sensitive authentication and authorization decisions should be enforced server-side.
Client-Side Hashing
The challenge exposes both the hashing algorithm and target hash in the JavaScript source. Because the
algorithm is available to the user, it can be reproduced and analyzed.
Hash Collisions
The custom hash is not cryptographically secure and can have collisions. Matching a custom client-side hash
is not equivalent to securely authenticating a user.
LocalStorage
The challenge stores an authentication state in LocalStorage. LocalStorage is controlled by the browser user
and should not be treated as a trusted security boundary.
URL Parameters
URL parameters are user-controlled input and should never be blindly trusted for sensitive authorization
decisions.
Base64 Encoding
The route value YXJjaGl2ZQ== is Base64 encoded and decodes to archive. Base64 is encoding, not
encryption.
Client-Side Secrets
The challenge contains important values directly inside JavaScript. Since frontend JavaScript is delivered to
the browser, users can inspect it. Real applications should never rely on hiding security-sensitive information
inside client-side source code.
n Intended Learning Path
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
n Key Findings
Finding Value
Hidden Route archive
Encoded Route YXJjaGl2ZQ==
Target Hash 4b3b7c6
Access Key SGycd
LocalStorage Key employee
LocalStorage Value verified
Required Page archive
Required Token alpha-42
Final Flag FLAG{browser_exploration_master}
 Difficulty
Beginner / Intermediate
The challenge is primarily focused on browser-based reconnaissance and client-side Web Security concepts.
It is intended as a learning-focused CTF rather than an advanced exploitation challenge.
 Project Status
Version 1.0 — Completed
Future versions may introduce additional challenge stages and m
n Project Status
Version 1.0 — Completed
Future versions may introduce additional challenge stages and m
