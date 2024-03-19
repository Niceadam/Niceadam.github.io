+++
title = 'Hacking my Gym'
date = 2023-11-16
+++

Note: To avoid legal issues, I'll refer to this gym company as "Jim".

"Jim" is a gym franchise in Sweden and they have
a local gym that I go to. It has a 24/7 unmanned section with automatic
entrance gates that you can use when the main section closes. There's QR codes
on the gates that you scan using their official app to open them. There's 2
gates with a one-person buffer zone to stop you from letting all your friends
in. The app is flimsy and the scanning feature feels slow.

One day I curiously scanned the code on the first gate using a QR reader
and got a long random string. Now, I have had attempts in the past to decode public
transport tickets, but those curiosities quickly faded after realizing the
complexity of those systems. At this point, I saw an opportunity to actually
dissect what was going on. Given the amateurish quality of the app, I assumed
the developers didn't put too much effort on security.

## Decompilation

First things first, get the latest APK and decompile it using
[`apktool`](https://apktool.org/). the output directory contains
`assets/index.android.bundle` so the app was probably made in React Native,
[`ripgrep`](https://github.com/BurntSushi/ripgrep) confirms it all over the
place. The bundle file contains Hermes bytecode, which is what React Native
compiles to nowadays making this a bit harder. Luckily I found
[`hermes-dec`](https://github.com/P1sec/hermes-dec) which decompiles the
bytecode to something that is somewhat readable. Searching the decompiled
output for some keywords (`password`, `https://`... whatever you fancy) spits
out..

```yml
jimApiDomain: 'https://webapi.jim.se'
jimApiDomainUsername: 'apiuser557644'
jimApiDomainPassword: 'ZZgfwIY0xiGOPDzD2...'
```

you can almost smell what's coming, surely they wouldn't embed global
credentials for the API in the app, right?

## _"Excuse me waiter, there is a man in my middle"_

I wanted get the API calls for opening the gates and I was too lazy to scavenge
the code for it, so I used [`mitmproxy`](http://mitm.it/) to catch the
traffic from my phone instead. Starting from Android 7.0, you have to recompile
the APK with custom certificates so it can trust your HTTPS proxy (this step is easier
using [`apk-mitm`](https://github.com/shroudedcode/apk-mitm)). I slapped the APK back
on my phone, started the proxy, scanned a picture of the
gate code from a friends phone and _voilà_.

`Request:`

```http
POST https://webapi.jim.se/persons/{my_user_id}/qr-checkin
Authorization: "..."
```

```json
{
    "lat": {Gym Latitude},
    "lng": {Gym Longitude},
    "qr": {QR code}
}
```

Every call contains basic authentication so I pulled up Postman and
tried the API calls myself with the username and password I got from the app
and..

`Response:`

```json
{
    "success": true,
    "qrData": {
        "centerId": "{my_gym_id}",
        "resourceType": "GATE",
        "resourceId": "GYM_GATE_OUTDOOR"
    }
}
```

_Open sesame_ the first gate opened, turns out that those were indeed global
credentials. It's the same call for the second indoor gate as well. I can now
make a shortcut on my phone homescreen that makes the call, so I don't have to
scan anything to get in. Looks like they use the gym location to make sure you're
not scanning the code on the other side of the country, not very useful, I
can let my friends in now too. But if the credentials are global, it doesn't
matter what the user ID, so whats the point?

## Going further

I played around in the app to see what other calls I can catch...

```http
GET https://webapi.jim.se/persons/{my_user_id}
```

This one returns

- Email
- Personnummer (Social security number)
- Full name, gender, date of birth
- Address
- .. and some other membership info, such as if I am blacklisted

This is where it gets spicy.. `{my_user_id}` looks like a very normal integer,
so I made the same call with `{my_user_id} + 1`.

Low and behold, it gives the information of some other account! They didn't
use `uuid`, they just assigned sequential ID's to people, yikes! I scribbled up
a python script that iteratively does calls with `user_id = [0, 1, ..]`. I got
a dozen accounts in a few seconds before I stopped it. I even got accounts from
Norway, super yikes!

Now that I know people's email and and personnumer, those form the default
login details to your account (unless you changed your default password, which
many people don't.. I know I didn't), so I can now login as that user and
control their account. Because of the global credentials, I can probably use
any one of these ID's to actually enter the gym (although I didn't test this
out)

## Conclusion

This was a quite basic attack and I was suprised at just how much damage was
done. I am no security expert but if you only need a few script kiddie tools to
do a complete data breach then things can probably be done a bit better.

There is details missing here that I didn't explore myself but overall its
evident that members data, including mine, were not secure. I emailed them about it
and they swiftly fixed the vulnerability
