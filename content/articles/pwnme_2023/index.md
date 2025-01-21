+++
template = "article.html"
title = "Writeup pwnme 2023"
date = 2023-05-09

[taxonomies]
tags = ["writeups", "ctf", "pwnme"]
+++
This year, I participated with my team securinsa. We didn't scored well but it was a good CTF, with fun challenges.
# OSINT
This category was mainly socmint related challenges. This writeup contains every challenges but I only flagged the first 2 ones. I wanted to make a full walkthrough of these challenges to keep the methodology in my mind. 
## Social Media Goes Brrrrr

<div class="rule">
[EN] Context explanation:
John Droper is a Franco-British individual who leaves an enormous digital footprint.
(Note: He speaks both English and French, and some information can only be found through one of these languages.)

First Step :
    - You have to find one of his main social media
</div>

For this first challenge, we need to find a social account of `John Droper`. By searching facebook, we find his account:
{{ resize_image(path="images/facebook_profil.png", height=400, width=500, op="fit") }}

There are many information to gather:
- He is a bad developper
- He registered his website on the AFNIC
- His ex girlfriend is the owner of his favourite bar
- He loves to travel and has an account on the train forum

At this point, I didn't found the flag. But by poking around his account, in the about section, I found the golden nugget!

{{ resize_image(path="images/about_droper.png", height=400, width=500, op="fit") }}

This page is rich in information. We can find the flag for the first challenge. But more important, we have a pseudo: `jdthetraveller`, we know that he exposes his life on the internet and that he has a website.

Step one completed.

## Newbie Dev
<div class="rule">
[EN] For the context of the challenge, please check the introduction challenge's descriptionJohn is a newbie dev, please find informations about this new passion not very developed
</div>

For the second challenge, my idea was to levarage the information gahtered in the previous part. First, be searching on the [AFNIC](https://www.afnic.fr/en/domain-names-and-support/everything-there-is-to-know-about-domain-names/find-a-domain-name-or-a-holder-using-whois/) with his name and his pseudo, we find a hit on jdthetraveller.fr.

{{ resize_image(path="images/droper_site.png", height=400, width=500, op="fit") }}
By going on the site, we don't see many information. But, he is talking about the new version of his site. So i try to access `jdthetraveller.fr/.git` and bingo! I found a git repo inside the side. Let's investigate.

We could use tools such [git-dumper](https://github.com/arthaud/git-dumper) but because It's a mess, I rather chcked the config file in the `.git` directory, and I found a github repo: https://github.com/droperkingjohn/myOwnWebsite.git.

A commun strategy is to check the commits in order to find deleted information. On one of the commit, we can find the second flag in a comment.

{{ resize_image(path="images/github_flag.png", height=400, width=500, op="fit") }}

Moreover, by scanning the other commits, we can extract much more information:
- a new pseudo: droperkingjohn 
- an email address: johndroperdroperjohn@gmail.com
- his passion about the Basketball team of **Tarbes**, in France
- A forum: https://www.forum-train.com/forum/index.php where he spoke about his trip.

Let's exploit these in the next part
## Europe
<div class="rule">
[EN] For the context of the challenge, please check the introduction challenge's description
Description: John loves adventure and travel can you give me the 3 cities he visited during his trip to Europe.
    Flag Format: PWNME{city1_city2_city3} Cities in lower case and in alphabetical order and separated by a "_" .

</div>

For this part, we need to find the 3 cities he traveled to in Europe. The first thing to do is to exploit the train forum. On the presentation thread, by searching a bit, we can find a thread by `jdthetraveller`:
{{ resize_image(path="images/jdthetraveller_train.png", height=400, width=500, op="fit") }}

New page, new information!  We know that he went by Bratislava and his train trip last 10h58.

At this point, we exploited all the previous gathered information. We need fresh info.

I used the [Blackbird](https://github.com/p1ngul1n0/blackbird) tool, because I find it more accurate that sherlock or holehe.

I ran the tools on the 3 gathered tools. And the results were promising:
{{ resize_image(path="images/neat_output.png", height=400, width=500, op="fit") }}

- with the jdthetraveller pseudo:
    - a jeuxvideo forum account
- with the droperkingjohn pseudo:
    - an Instagram account 
    - an Twitter account and its web archive

Wow, so much new information to exploit!
### JVC account
Let's start with jvc account. I searched by author and I found a topic

{{ resize_image(path="images/jvc_search.png", height=400, width=500, op="fit") }}

On the topic, I found a strange message...

{{ resize_image(path="images/strange_msg.png", height=400, width=500, op="fit") }}

I couldn't exploit this information during the ctf, but after someone talked about a tool, [what3word](https://what3words.com). This tool can be used to describe a place with only 3 words. By using these strange words, we can find **Kausas**, a city in Lithuania.

{{ resize_image(path="images/what3words.png", height=400, width=500, op="fit") }}

### Instagram account

Now, let's go on the instagram account. Among the post, one struck my attention:

{{ resize_image(path="images/billet_insta.jpg", height=400, width=500, op="fit") }}
He took a picture of a 10euro ticket and said that he wants to track it. I discovered a passion of people tracking their euro through Europe thanks their serial number and the [Eurobilltracker website](https://en.eurobilltracker.com/). I went on the website and by searching for jdthetraveller, with: `user:jdthetraveller`, I found a hit on a note of 10 euro. The city was **Gols**, in Austria

### Train forum

Let's go back to the train forum. During the ctf, I only managed to find Gols, but didn't successfully managed to exploit the travel duration to find the city. I tried different tools, with no success. But someone on the discord mentioned a very convinient tool, https://direkt.bahn.guru/. This tools allows to search for destination from a specific train station and the travel duration. 
{{ resize_image(path="images/bahn_guru.png", height=400, width=500, op="fit") }}

By searching around the farest destinations, we can find the city of **Terespol** that has exactly the right travel duration.

The flag is then: PWNME{gols_kausas_terespol}. GG to the people who flagged during the ctf, I did not but I learned a lot.

## French Dream
<div class="rule">
[EN] For the context of the challenge, please check the introduction challenge's description
Description: John is Franco-English but he lives in France and his life is almost entirely available on the internet. Find the city where he lives, the username of his current girlfriend and the birth name of his ex. The OSINT must remain passive and any interaction is strongly prohibited. No need to get in touch with anyone to solve the challenge.
        Flag Format: PWNME{lille_elonmusk_halliday} All lowercase with "_" to separate information.
</div>

For this one, I only found the place where he lived. I will go through this challenge, even I was far for solving it.

### The place John lives
For the place John lives, we can find a post on his instagram, where he talks about a barbecue. There is a receipt attach to the post, with his adress on the top. The tricky part here is that on instgram, the addres is cropped, but as I don't have an instagram account, I used Picuki, a service allowing to see instagram accounts without being connected. And Picuki does not crop images, so straightforward.

{{ resize_image(path="images/payment.jpg", height=400, width=500, op="fit") }}

### The ex-girlfriend's name
**For this part, I was near but I did not found the name.** 
For this part, we had to exploit facebook and instagram. On facebook, we learnt that his ex is the owner of his favourite bar.
On Instagram, he posted a picture talking about this bar with a encoded location:

{{ resize_image(path="images/bar.png", height=400, width=500, op="fit") }}

By using google lens, we can extract the text. It is then a simple encoding: It uses a old-fashion phone keyboard to encode each character.

{{ resize_image(path="images/old-phone.jpg", height=400, width=500, op="fit") }}
We extract the code:
```
3444 777 33 222 8 444 666 66 0
66 666 777 3 0 555 33 0 555 666 66 4
0333 0 555 20 777 444 888 444 33
777 33 0 7 777 33 6 444 33 777 0 222
777
```
and then with the help of the [dcode's t9 cyper](https://www.dcode.fr/t9-cipher), we extract a setence:
`DIRECTION NORD LE/LDD LONG F LA RIVIERE PREMIER BAR/CR/ABR`, in french and `NORTH ALONG THE FIRST BAR RIVER` in english.

On the post, He's also mentionning that he will watching his favourite basket team, Tarbes Gespe Bigorre, match. So we know that he is in Tarbes.

By searching bar that face north, near the river and on the way from the Tarbe Gespe Bigorre, we can find the bar "Le landais"

{{ resize_image(path="images/first-bar-north.png", height=400, width=500, op="fit") }}

We need the name of the owner. One widely used tool, for french companies is: [societe.com](https://societe.com). By searching for the bar, we found it, with the birthname of the owner, and it's win!

{{ resize_image(path="images/ex-name.png", height=400, width=500, op="fit") }}

### Name of his actual girlfriend
**I did not sole this part because I don't have sockpuppet, silly me**.

I struggled a lot on this, searching on his instagram, without success. I learnt afterwards that his girlfriend was following him on twitter, but I dont have an account, so I can't access this information.

The flag is PWNME{lannemezan_blanchelovejd_ghestem}. GG for those who get it, and thanks to the discord community for the help on this.

## American dream
<div class="rule">
[EN] For the context of the challenge, please check the introduction challenge's description
Description: John has wonderful memories of his trip to the United States but he lost his memory on certain points, with all the information you have on him, you can help him, can't you?
        Give me the city of the assembly plant of the car he rented. The email service used by the company where he saw the dog of his dreams. The price (in dollars rounded up) of what he bought when he went to the bar that was robbed on 01/15/2023.
        Flag Format: PWNME{miami_sendinblue_2600} All lowercase with "_" to separate information.
    
</div>

This was the hardest challenge of this category. I only managed to find his calendar, everything else was found after the ctf.

### The price in the bar

I exploit his email address for the first time, using the great [Ghunt tool](https://github.com/mxrch/GHunt). This can extract a ton of information on a google account. This gave me a neat json with all of his meeting:
```Json
{  
   "PROFILE_CONTAINER": {  
       "profile": {  
           "coverPhotos": {  
               "PROFILE": {  
                   "flathash": null,  
                   "url": "https://lh3.googleusercontent.com/c5dqxl-2uHZ82ah9p7yxrVF1ZssrJNSV_15Nu0TUZwzCWqmtoLxCUJgEzLGtxsrJ6-v6R6rK  
U_-FYm881TTiMCJ_",  
                   "isDefault": true  
               }  
           },  
           "sourceIds": {  
               "PROFILE": {  
                   "lastUpdated": "2023/03/29 20:39:53 (UTC)"  
               }  
           },  
           "profileInfos": {  
               "PROFILE": {  
                   "userTypes": [  
                       "GOOGLE_USER"  
                   ]  
               }  
           },  
           "profilePhotos": {  
               "PROFILE": {  
                   "flathash": "00000c0c3c180000",  
                   "url": "https://lh3.googleusercontent.com/a/AGNmyxYX3psPK83g_YqZJ2pFJL9i1kB8uFkT1fe57VdB=mo",  
                   "isDefault": true  
               }  
           },  
           "personId": "107243276973758576449",  
           "extendedData": {  
               "dynamiteData": {  
                   "entityType": "PERSON",  
                   "presence": "UNKNOWN",  
                   "customerId": "",  
                   "dndState": "AVAILABLE"  
               },  
               "gplusData": {  
                   "isEntrepriseUser": false,  
                   "contentRestriction": "WALLED_GARDEN"  
               }  
           },  
           "emails": {  
               "PROFILE": {  
                   "value": "johndroperdroperjohn@gmail.com"  
               }  
           },  
           "inAppReachability": {},  
           "names": {  
               "PROFILE": {  
                   "lastName": "Droper",  
                   "firstName": "John",  
                   "fullname": "John Droper"  
               }  
           }  
       },  
       "play_games": null,  
       "maps": {  
           "photos": [],  
           "reviews": [],  
           "stats": {}  
       },  
       "calendar": {  
           "details": {  
               "time_zone": "Europe/Paris",  
               "conference_properties": {  
                   "allowed_conference_solution_types": [  
                       "hangoutsMeet"  
                   ]  
               },  
               "id": "johndroperdroperjohn@gmail.com",  
               "summary": "johndroperdroperjohn@gmail.com"  
           },  
           "events": {  
               "time_zone": "Europe/Paris",  
               "updated": "2023/05/04 22:06:57 (UTC)",  
               "access_role": "reader",  
               "next_page_token": null,  
               "items": [  
                   {  
                       "original_start_time": {  
                           "time_zone": "",  
                           "date_time": null  
                       },  
                       "guest_can_invite_others": null,  
                       "html_link": "https://www.google.com/calendar/event?eid=NTRxajR2OTYzM2h2cGQwam51Y2RvMDdobnMgam9obmRyb3BlcmRyb3  
BlcmpvaG5AbQ",  
                       "status": "confirmed",  
                       "id": "54qj4v9633hvpd0jnucdo07hns",  
                       "location": "the Skellig, 2409 N Henderson Ave, Dallas, TX 75206, \u00c9tats-Unis",  
                       "end": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "updated": "2023/05/04 22:01:27 (UTC)",  
                       "recurring_event_id": null,  
                       "description": "2 Soft Baked Pretzel\n1 Grilled chicken",  
                       "visibility": null,  
                       "sequence": 0,  
                       "ical_uid": "54qj4v9633hvpd0jnucdo07hns@google.com",  
                       "start": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "organizer": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null
                       },  
                       "summary": "Bar",  
                       "reminders": {  
                           "overrides": [],  
                           "use_default": false  
                       },  
                       "event_type": "default",  
                       "created": "2023/05/04 22:01:27 (UTC)",  
                       "creator": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       }  
                   },  
                   {  
                       "original_start_time": {  
                           "time_zone": "",  
                           "date_time": null  
                       },  
                       "guest_can_invite_others": null,  
                       "html_link": "https://www.google.com/calendar/event?eid=MWlnMXQzOGQxbjQ4Zmo5NzhpZmNuamsyMTggam9obmRyb3BlcmRyb3  
BlcmpvaG5AbQ",  
                       "status": "confirmed",  
                       "id": "1ig1t38d1n48fj978ifcnjk218",  
                       "location": "The Cottage Lounge, 3006 W Northwest Hwy, Dallas, TX 75220, \u00c9tats-Unis",  
                       "end": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "updated": "2023/05/04 22:03:27 (UTC)",  
                       "recurring_event_id": null,  
                       "description": "1 french fries\n1 texas burger",  
                       "visibility": null,  
                       "sequence": 0,  
                       "ical_uid": "1ig1t38d1n48fj978ifcnjk218@google.com",  
                       "start": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "organizer": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       },  
                       "summary": "Bar",  
                       "reminders": {  
                           "overrides": [],  
                           "use_default": false  
                       },  
                       "event_type": "default",  
                       "created": "2023/05/04 22:03:27 (UTC)",  
                       "creator": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       }  
                   },  
                   {  
                       "original_start_time": {  
                           "time_zone": "",  
                           "date_time": null  
                       },  
                       "guest_can_invite_others": null,  
                       "html_link": "https://www.google.com/calendar/event?eid=NmEydWhsNjBnZ3FsNGJsMmhxdmZqZDdoN2Egam9obmRyb3BlcmRyb3  
BlcmpvaG5AbQ",  
                       "status": "confirmed",  
                       "id": "6a2uhl60ggql4bl2hqvfjd7h7a",  
                       "location": "The Royal Pour Bar and Grill, 9909 Garland Rd, Dallas, TX 75218, \u00c9tats-Unis",  
                       "end": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "updated": "2023/05/04 22:04:49 (UTC)",  
                       "recurring_event_id": null,  
                       "description": "3 Brisket Grilled Cheese\n1 Royal Dogs",  
                       "visibility": null,  
                       "sequence": 0,  
                       "ical_uid": "6a2uhl60ggql4bl2hqvfjd7h7a@google.com",  
                       "start": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "organizer": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       },  
                       "summary": "Bar",  
                       "reminders": {  
                           "overrides": [],  
                           "use_default": false  
                       },  
                       "event_type": "default",  
                       "created": "2023/05/04 22:04:49 (UTC)",  
                       "creator": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       }  
                   },  
                   {  
                       "original_start_time": {  
                           "time_zone": "",  
                           "date_time": null  
                       },  
                       "guest_can_invite_others": null,  
                       "html_link": "https://www.google.com/calendar/event?eid=NzBhYmpybm52M29sNWdpMW51cHM4ZWRtMDggam9obmRyb3BlcmRyb3  
BlcmpvaG5AbQ",  
                       "status": "confirmed",  
                       "id": "70abjrnnv3ol5gi1nups8edm08",  
                       "location": "Monkey Bar, 77 Highland Park Village, Dallas, TX 75205, \u00c9tats-Unis",  
                       "end": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "updated": "2023/05/04 22:05:41 (UTC)",  
                       "recurring_event_id": null,  
                       "description": "7 Cocktails of the Day",  
                       "visibility": null,  
                       "sequence": 0,  
                       "ical_uid": "70abjrnnv3ol5gi1nups8edm08@google.com",  
                       "start": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "organizer": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       },  
                       "summary": "Bar",  
                       "reminders": {  
                           "overrides": [],  
                           "use_default": false  
                       },  
                       "event_type": "default",  
                       "created": "2023/05/04 22:05:41 (UTC)",  
                       "creator": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       }  
                   },  
                   {  
                       "original_start_time": {  
                           "time_zone": "",  
                           "date_time": null  
                       },  
                       "guest_can_invite_others": null,  
                       "html_link": "https://www.google.com/calendar/event?eid=MHJwczJuamxiM3BiMnRlbjNvOWtnMGM1cWIgam9obmRyb3BlcmRyb3  
BlcmpvaG5AbQ",  
                       "status": "confirmed",  
                       "id": "0rps2njlb3pb2ten3o9kg0c5qb",  
                       "location": "The Hard Shake Bar, 211 S Akard St, Dallas, TX 75202, \u00c9tats-Unis",  
                       "end": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "updated": "2023/05/04 22:06:57 (UTC)",  
                       "recurring_event_id": null,  
                       "description": "2 Supurrito\n1 Swipe Right",  
                       "visibility": null,  
                       "sequence": 0,  
                       "ical_uid": "0rps2njlb3pb2ten3o9kg0c5qb@google.com",  
                       "start": {  
                           "time_zone": null,  
                           "date_time": null  
                       },  
                       "organizer": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       },  
                       "summary": "Bar",  
                       "reminders": {  
                           "overrides": [],  
                           "use_default": false  
                       },  
                       "event_type": "default",  
                       "created": "2023/05/04 22:06:57 (UTC)",  
                       "creator": {  
                           "email": "johndroperdroperjohn@gmail.com",  
                           "self": true,  
                           "display_name": null  
                       }  
                   }  
               ],  
               "default_reminders": [],  
               "summary": "johndroperdroperjohn@gmail.com"  
           }  
       }  
   }  
}⏎
```

Because we have many Texan place, we can guess that he is in Dallas, Texas, in the US. His twitter archived tweet confirms this.
There is open data on crime for this city on this [website](https://www.dallasopendata.com/Public-Safety/New-Crime-Watch/9c2r-acu8)
We can filter on date and address and search only for the bar John went.

We got an hit on `the Skellig`: that the bar. We can retrive the menu and with the google calendar, we have what he ordered.

There is the menu on the website but Its prices have changed since John came. But With Wayback machine, and going in February, we find the correct prices: 21$.

### Email provided for the dog 

On his twitter, there is a picture of his dreamed dog.  By using google lens, we can find the company: www.dognkittycity.org. There no obvious information on the mail provider. But we could think of dig on the website to get more information. One of the configured DNS records for mail servers are TXT one. Running `dig txt dognkittycity.org` gave us response, and one of the field is a SPF record, which is used in email authentication.

{{ resize_image(path="images/dig-ans.png", height=400, width=500, op="fit") }}

By going on eoidentity.com, we have the mail of the provider: **EmailOctopus**

### Car plant

For this one, by searching through the archived tweets of John, we could find a car license plate: 

{{ resize_image(path="images/car.png", height=400, width=500, op="fit") }}

Using a tool I discovered after the ctf, https://www.lookupaplate.com/, we can find the right car with this license plate and the State of Texas (found in the previous steps)

{{ resize_image(path="images/plant-city.png", height=400, width=500, op="fit") }}

So this car has been manufactured in HWASUNG.

Finally the flag is PWNME{hwasung_emailoctopus_21}

# Conclusion

The OSINT part of the CTF, even if I did successed well (2 out of 5 challenges flagged), was really fun and well made. I learned a lot and discovered new tools. GG to whom flagged every challenges and the organizers for such a great OSINT challenges.
