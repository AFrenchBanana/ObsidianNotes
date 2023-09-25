Abuses Sanitation of `write` user input.
### Types of Attacks
|Type|Description|
|---|---|
|`Stored (Persistent) XSS`|The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)|
|`Reflected (Non-Persistent) XSS`|Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)|
|`DOM-based XSS`|Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)|


### Stored XSS
Stored in back-end database. 
The attack is persistent no matter the user.
#### Testing Payloads 
```html
<script>alert(window.origin)</script>
```
Sometimes alert  is blocked. 
Other examples:
`<plaintext>`
`<script>print()</script>`
### Reflected XSS
Non persistent and never reach the backend
Do not occur on page refreshes.

To reach a target, send a payload containg the data.
Look at the GET request on dev settings. 
#### Test 
`<script>alert(window.origin)</script>`
