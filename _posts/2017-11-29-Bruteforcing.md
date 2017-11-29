---
title: "Pentest: Bruteforcing passwords and password spraying"
categories:
  - Penetration-Testing
tags:
  - Pentest
  - Bruteforce
---

Bruteforcing passwords is easily detectable and easly fixed by restricting the number of login attempts before locking out an account.
Bruteforcing is done by attempting a login with a certain user and trying different permutations of passwords usually with the help of an automated tool and a good wordlist.

Password spraying reverses this method, and instead of permutating different passwords, it enumerates through users while trying to login with the same password. This way you can effectivly avoid an account lockout. The only drawback in this method is that you'd have to find (through recon or social engineering) valid usernames, or you effectivly doubled the unknown variable which makes it exponentially difficult to succesfully crack a username/login combination.
