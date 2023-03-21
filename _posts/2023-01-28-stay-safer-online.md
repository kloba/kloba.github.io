---
layout: post
title: "Stay Safer Online: How to Generate Unique Passwords and Usernames on MacOS"
---

In today's digital age, data breaches are becoming increasingly common. A recent report found that 85% of companies experience at least one ransomware attack yearly [<a href="#link1">1</a>]. This means that it's more important than ever to take steps to protect your personal information online. One of the potential ways to do this is to use unique passwords and usernames for every website and system you use. This guide will show you how to use the Shortcuts app on MacOS to quickly generate random passwords and usernames.

<p align="center">
  <img src="/imgs/stay-safer-online/picture1.gif" />
</p>
 
**Why Use Unique Passwords and Usernames?**

Using the same password for multiple websites and services is a recipe for disaster. If one website or service you use experiences a data breach, your personal information and login credentials can be compromised. This can put all of your online accounts at risk, as many people use the same email and password combination across multiple sites.

Using unique and random passwords for each website and service is one of the best ways to protect yourself online. Additionally, turning on two-factor authentication (2FA) provides an extra layer of security for your accounts.

**Creating Shortcuts for Unique Passwords and Usernames on MacOS**
<ol>
<li>Open the Shortcuts app on your Mac.</li>
<li>Click on the "+" button in the top right corner to create a new shortcut.</li>
<li>Give your shortcut a name, such as "Generate a random password."</li>
<li>In the "Actions" section, add a "Run Script" action and enter the following command:<br/></li>
</ol>
<code>password=$(/usr/local/bin/pwgen -y -B 16 1) && echo $password | pbcopy<br/>osascript -e 'tell application "System Events" to keystroke "v" using {command down}'</code>
<br/>
<div class='bordered'>
This code snippet generates a random password using the <b>pwgen</b> tool. The "<b>-y</b>" flag is used to include at least one special character in the password, and the "<b>-B</b>" flag is used to exclude ambiguous characters such as "l" and "1" that could be confused when printed. The password is set to have a length of 16 characters.
The generated password is saved to the variable $password and is then echoed and piped to the <b>pbcopy</b> command which copies the generated password to the clipboard.
The <b>osascript</b> command is used to simulate the keystroke <b>command+v</b>, which pastes the copied password.
This allows the user to quickly generate a random password and paste it into the desired location without having to manually type it out or copy and paste it.
<br/>
</div>
<br/>
<ol>
<li value="5">In the "Name" field, enter a name for the shortcut, such as "Generate random password".</li>
<li>Click "Add" to save the shortcut.</li>
<li>Repeat the steps for "Generate a random username":</li>
</ol>
<code>username=$(/usr/local/bin/pwgen -0 -A -B 8 1) && echo $username | pbcopy<br/>osascript -e 'tell application "System Events" to keystroke "v" using {command down}'<code>

<p align="center">
  <img src="/imgs/stay-safer-online/picture2.png" />
</p>
<ol> 
<li value="8">In order for this command to work, you also need to allow both the Shortcuts app and siriactionsd in System Preferences > Security & Privacy > Privacy > Accessibility.</li>
</ol>
<p align="center">
  <img src="/imgs/stay-safer-online/picture3.png" />
</p>
<ol>
<li value="9">Now you have two new shortcuts in the Shortcuts app, one for generating a random password and one for generating a random username. But in order to trigger them quickly, you can add global keyboard shortcuts for them.</li>
<li>To do this, go to System Preferences > Keyboard > Shortcuts > App Shortcuts.</li>
<li>Click the "+" button to add a new shortcut.</li>
<li>In the "Application" field, select "All Applications."</li>
<li>In the "Menu Title" field, enter the exact name of the shortcut you created earlier, such as "Generate a random password" or "Generate a random username."</li>
<li>In the "Keyboard Shortcut" field, enter the combination you want to use to trigger the shortcut, such as Control + Shift + Command + P for the password and Control + Shift + Command + U for the username.</li>
<li>Click "Add" to save the shortcut.</li>
</ol>
 <p align="center">
  <img src="/imgs/stay-safer-online/picture4.png" />
</p>
<ol>
<li value="16">Now you can use the keyboard shortcuts you set up to quickly generate a random password or username and paste it into the desired field.</li>
</ol>
It's important to note that while generating unique passwords and usernames can help protect your personal information, it's not a guarantee of security. Additionally, it's a good idea to check if your email or password has been compromised in a data breach by using a service like https://haveibeenpwned.com/. Taking these steps and being cautious about the information you share online can increase your chances of staying safer online.

It is important to note that storing passwords in a browser can also be dangerous, as it is one of the first places that hackers and malware will check. To ensure the security of your passwords, it is recommended to use more sophisticated tools like 1Password to store and manage them.

[<a name="link1">1</a>] Report: 85% of companies experience at least one ransomware attack per year (https://venturebeat.com/security/report-85-of-companies-experience-at-least-one-ransomware-attack-per-year/)
