---
layout: post
title:  "Try Hack Me - Industrial Intrusion Osint Write-Up"
---
# THM Industrial Intrusion

## OSINT WriteUp

## Mission 1

*Hexline,
 we need your help investigating the phishing attack from 3 months ago. 
We believe the threat actor managed to hijack our domain 
virelia-water.it.com and used it to host some of their infrastructure at
 the time. Use your OSINT skills to find information about the 
infrastructure they used during their campaign*

Let’s scan for subdomains of [virelia-water.it.com](https://virelia-water.it.com/).

subfinder -d [virelia-water.it.com](https://virelia-water.it.com/) -silent

We find the subdomains:

[stage0.virelia-water.it.com](https://stage0.virelia-water.it.com/)

[54484d7b5375357373737d.virelia-water.it.com](https://54484d7b5375357373737d.virelia-water.it.com/)

The first one actually points to a web page.

https://stage0.virelia-water.it.com/

The second one doesn’t, and the 54484d7b5375357373737d part is a hex string which is the first flag 🎊

### THM{Su5sss}

## Mission 2

*Great work on uncovering that suspicious subdomain, Hexline. However, your work here isn’t done yet, we believe there is more.*

In the first subdomain [stage0.virelia-water.it.com](https://stage0.virelia-water.it.com/) we can see a website:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image.png)

In the source code of it we can see link to:

[https://raw.githubusercontent.com/SanTzu/uplink-config/refs/heads/main/init.js](https://raw.githubusercontent.com/SanTzu/uplink-config/refs/heads/main/init.js)

Which has the following content:

```
var beacon = {
  session_id: "O-TX-11-403",
  fallback_dns: "uplink-fallback.virelia-water.it.com",
  token: "JBSWY3DPEBLW64TMMQQQ=="
};
```

It is another subdomain, which doesn’t seem to point to a website.

We can dig into it:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%201.png)

The `dig` command was used to query the DNS TXT records for the subdomain `uplink-fallback.virelia-water.it.com`. 

TXT records are a type of DNS record that can store arbitrary text data. They're often used for verification purposes, but in this case, it contained a base64 encoded string that revealed the flag when decoded.

`eyJzZXNzaW9uIjoiVC1DTjEtMTcyIiwiZmxhZyI6IlRITXt1cGxpbmtfY2hhbm5lbF9jb25maXJtZWR9In0=`

### THM{uplink_channel_confirmed}

## Mission 3

*After the initial breach, a single OT-Alert appeared in Virelia’s monthly digest—an otherwise unremarkable maintenance notice, mysteriously signed with PGP. Corporate auditors quietly removed the report days later, fearing it might be malicious. Your mission is to uncover more information about this mysterious signed PGP maintenance message.*

So in [https://virelia-water.it.com](https://virelia-water.it.com/) we can see:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%202.png)

And in archives:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%203.png)

Searching in the web archive didn't yield any results.

Digging, however:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%204.png)

CNAME is a type of DNS record that maps one domain name (an alias) to another.

In this case, the CNAME record for [virelia-water.it.com](https://virelia-water.it.com) points to [virelia-water.github.io](https://virelia-water.github.io), indicating that the website content is actually hosted on GitHub Pages.

This is significant for our investigation because it reveals that the company's web presence is connected to a GitHub account, potentially giving us access to more information through that platform.

If a website is hosted at [virelia-water.github.io](https://virelia-water.github.io), it typically indicates that the GitHub user "virelia-water" is the owner of that repository. GitHub Pages allows users to host websites directly from their GitHub repositories, and the URL pattern is usually [username].github.io.

And there is a user with that name! Joined 5 days ago!

https://github.com/virelia-water

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%205.png)

And compliance is the site!!!!!!

Some funny issues here

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%206.png)

Now we can look for interesting commits, to see what was deleted from the site.

[https://github.com/virelia-water/compliance/commits/main/](https://github.com/virelia-water/compliance/commits/main/)

Looks good:

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%207.png)

And here we have a pgp signature, a name “DarkPulse” and an email alerts@virelia-water.it.com.

![image.png](/assets/img/imgs-2025-06-30-thm-industrial-intrusion-osint/image%208.png)

The pgp message:

```python
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

Please confirm system integrity at 03:00 UTC.

-----BEGIN PGP SIGNATURE-----

iQFQBAEBCgA6FiEEiN7ee3MFE71e3W2fpPD+sISjEeUFAmhZTEQcHGFsZXJ0c0B2
aXJlbGlhLXdhdGVyLml0LmNvbQAKCRCk8P6whKMR5ZIUCADM7F0WpKWWyj4WUdoL
6yrJfJfmUKgJD+8K1neFosG7yaz+MspYxIlbKUek/VFhHZnaG2NRjn6BpfPSxfEk
uvWNIP8rMVEv32vpqhCJ26pwrkAaUHlcPWqM4KYoAn4eEOeHCvxHNJBFnmWI5PBF
pXbj7s6DhyZEHUmTo4JK2OZmiISP3OsHW8O8iz5JLUrA/qw9LCjY8PK79UoceRwW
tJj9pVsE+TKPcFb/EDzqGmBH8GB1ki532/1/GDU+iivYSiRjxWks/ZYPu/bhktT
NNcOzgEfuSekkQAz+CiclXwEcLQb219TqcS3plnaO672kCV4t5MUCLvkXL5/kHms
Sh5H
=jdL7
-----END PGP SIGNATURE-----
```

So I saved the signature in `msg.asc` file, and ran:

`$ gpg --verify msg.asc`

`gpg: Signature made Mon 23 Jun 2025 08:44:52 AM EDT
gpg:                using RSA key 88DEDE7B730513BD5EDD6D9FA4F0FEB084A311E5
gpg:                issuer "[alerts@virelia-water.it.com](mailto:alerts@virelia-water.it.com)"
gpg: Can't check signature: No public key`

Than I tried to import the public key:

```bash
gpg  --recv-keys 88DEDE7B730513BD5EDD6D9FA4F0FEB084A311E5
```

But it didn’t work because there was no uid in the public key.

`gpg: key F8ED5BC28874364F: new key but contains no user ID - skipped`

So I searched and found the solution here:

[https://superuser.com/questions/1485213/gpg-cant-import-key-new-key-but-contains-no-user-id-skipped](https://superuser.com/questions/1485213/gpg-cant-import-key-new-key-but-contains-no-user-id-skipped)

We need to use a different keyserver that doesn’t enforce having a uid.

And finally:

```bash
gpg --keyserver hkps://keyserver.ubuntu.com  --recv-keys 88DEDE7B730513BD5EDD6D9FA4F0FEB084A311E5
```

### THM{h0pe_th1s_k3y_doesnt_le4d_t0_m3}