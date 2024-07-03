---
title: Fare Evasion
author: bytebrew
layout: writeup
category: UIUCTF-2024
chall_description: 
points: 370
solves: 173
tags: web
date: 2024-06-29
comments: false
---
SIGPwny Transit Authority needs your fares, but the system is acting a tad odd. We'll let you sign your tickets this time!

[https://fare-evasion.chal.uiuc.tf/](https://fare-evasion.chal.uiuc.tf/)
---
# First looks
When pressing the "I'm a Passenger" button, we get this alert: ```
```
Sorry passenger, only conductors are allowed right now. Please sign your own tickets.  
hashed _RòsÜxÉÄÅ´\ä secret: a_boring_passenger_signing_key_?
```
Examining the JavaScript function triggered by the button reveals the following:
```js
async function pay() {
	// i could not get sqlite to work on the frontend :(
	/*
		db.each(`SELECT * FROM keys WHERE kid = '${md5(headerKid)}'`, (err, row) => {
		???????
	*/
	const r = await fetch("/pay", { method: "POST" });
	const j = await r.json();
	document.getElementById("alert").classList.add("opacity-100");
	// todo: convert md5 to hex string instead of latin1??
	document.getElementById("alert").innerText = j["message"];
	setTimeout(() => { document.getElementById("alert").classList.remove("opacity-100") }, 5000);
}
```
This indicates that the hash we're getting in our error message is encoded in Latin1.  
# Creating a Malicious JWT
Looking at [this write up](https://cvk.posthaven.com/sql-injection-with-raw-md5-hashes) we see that the password `129581926211651571912466741651878684928` would trigger an SQL injection when hashed in md5 and encoded in Latin1. Now we have to figure out how to sign this password. 

Opening this in Burp Suite we can intercept the `/pay` post request triggered when you press the passenger button and get our current JWT. All we have to do is plug it and the secret (given to us in the error message) into https://jwt.io/, change the kid to "129581926211651571912466741651878684928", and change the JWT in the request we intercepted via Burp Suite. We then get the response:
```
Sorry passenger, only conductors are allowed right now. Please sign your own tickets. \nhashed \u00f4\u008c\u00f7u\u009e\u00deIB\u0090\u0005\u0084\u009fB\u00e7\u00d9+ secret: conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e\nhashed _\bR\u00f2\u001es\u00dcx\u00c9\u00c4\u0002\u00c5\u00b4\u0012\\\u00e4 secret: a_boring_passenger_signing_key_?"
```
# Crafting A Conductor JWT
All we have to do is make a JWT signed with:
```
conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e
```
Once we do, we get the flag:

	uiuctf{sigpwny_does_not_condone_turnstile_hopping!}
