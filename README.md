Alice, Bobby and Charlie want to share information privately.


ALICE
---
Generate a key.

		 gpg --gen-key

BOB
---
Generate a key.

		gpg --gen-key


ALICE
---
Export public key for sharing with Bob.

        gpg --armor --output /files/alicekey.asc --export "ALICE"
        
BOB
---
Export public key for sharing with Alice.

        gpg --armor --output /files/bobkey.asc --export "BOBBY"
        

ALICE
---
Import BOB's key so she can encrypt files for him to decrypt

        gpg --import /files/bobkey.asc

      
BOB
---
Import ALICE's key so he can encrypt files for her to decrypt

        gpg --import /files/alicekey.asc

CHARLIE
---
Import ALICE's and BOBBY's keys so he can transfer encrypted messages
which both ALICE and BOBBY can read.

        gpg --import /files/alicekey.asc
        gpg --import /files/bobkey.asc

Charlie trusts both ALICE and BOBBY and so can tell gpg to set their keys
trust level to a high level.

        gpg --edit-key ALICE
        
GPG will then show a mini-terminal command terminal.
        
        gpg> trust

The `trust` command will show a list of trust levels to choose from:

    gpg> trust
    pub  rsa2048/BEC092243EF4FB07
         created: 2017-02-28  expires: 2019-02-28  usage: SC  
         trust: unknown       validity: unknown
    sub  rsa2048/011DDAAD4EBD4949
         created: 2017-02-28  expires: 2019-02-28  usage: E   
    [ unknown] (1). ALICE <alice@local>
    
    Please decide how far you trust this user to correctly verify other users' keys
    (by looking at passports, checking fingerprints from different sources, etc.)
    
      1 = I don't know or won't say
      2 = I do NOT trust
      3 = I trust marginally
      4 = I trust fully
      5 = I trust ultimately
      m = back to the main menu
    
    Your decision? 5
    
    ...
    
    gpg> save

Setting the trust level to "ultimate" may sound scary and you might like to
set it to a lower level, but if you do you will be constantly prompted to
"use this key anyway?" whenever you are encrypting a file ...

Now ALICE's key is trusted and when we encrypt a file GPG will not warn CHARLIE
about the 'untrusted' nature of ALICE's key. 

    gpg -er ALICE sample.txt 
    gpg: checking the trustdb
    gpg: marginals needed: 3  completes needed: 1  trust model: pgp
    gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
    gpg: next trustdb check due at 2019-02-28

We can experiment with BOBBY's key (which we haven't set up trust for) thus:

    gpg -er BOBBY sample.txt
    gpg: checking the trustdb
    gpg: marginals needed: 3  completes needed: 1  trust model: pgp
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    gpg: D08BAB48518E2D52: There is no assurance this key belongs to the named user
    sub  rsa2048/D08BAB48518E2D52 2017-02-28 BOBBY <bob@local>
     Primary key fingerprint: 9E25 8869 DC37 0DD5 381A  7FCD F4C2 22DA 90A4 D985
          Subkey fingerprint: 01F7 A0B2 36F0 BAB0 8730  BC4D D08B AB48 518E 2D52
    
    It is NOT certain that the key belongs to the person named
    in the user ID.  If you *really* know what you are doing,
    you may answer the next question with yes.
    
    Use this key anyway? (y/N) 

Following the steps above to set the trust level for BOBBY's key:

    gpg --edit-key BOBBY
    gpg> trust
    ...
    gpg> save

Now CHARLIE can encrypt a file that both ALICE and BOBBY can decrypt.

To see the results of CHARLIE's key management efforts, he can ask gpg to
list his keys.

    gpg --list-keys
    /home/user/.gnupg/pubring.kbx
    -----------------------------
    
    pub   rsa2048 2017-02-28 [SC] [expires: 2019-02-28]
          9E258869DC370DD5381A7FCDF4C222DA90A4D985
    uid           [ultimate] BOBBY <bob@local>
    sub   rsa2048 2017-02-28 [E] [expires: 2019-02-28]
    
    pub   rsa2048 2017-02-28 [SC] [expires: 2019-02-28]
          7E57FA2BBD5C3C0E1E26C779BEC092243EF4FB07
    uid           [ultimate] ALICE <alice@local>
    sub   rsa2048 2017-02-28 [E] [expires: 2019-02-28]
    

CHARLIE
---

     gpg -ae -r ALICE -r BOBBY sample.txt
     gpg: checking the trustdb
     gpg: marginals needed: 3  completes needed: 1  trust model: pgp
     gpg: depth: 0  valid:   3  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 3u
     gpg: next trustdb check due at 2019-02-28

Now there is a file called `sample.txt.asc` which is an "ascii-armoured"
encrypted version of the file, suitable for pasting into an email,
a chat program or whatever.

ALICE
---

    root@alice:/files# gpg -d sample.txt.asc
    gpg: encrypted with RSA key, ID D08BAB48518E2D52
    gpg: encrypted with 2048-bit RSA key, ID 011DDAAD4EBD4949, created 2017-02-28
          "ALICE <alice@local>"
    this is a sample text

    there are some lines


    here is another line


    this is the last line.
    root@alice:/files#     


The decryption report does not show BOBBY's key when we decrypt on ALICE's
machine - ALICE never imported BOBBY's key.

BOBBY
---

    root@bob:/files# gpg -d sample.txt.asc
    gpg: encrypted with 2048-bit RSA key, ID 011DDAAD4EBD4949, created 2017-02-28
          "ALICE <alice@local>"
    gpg: encrypted with 2048-bit RSA key, ID D08BAB48518E2D52, created 2017-02-28
          "BOBBY <bob@local>"
    this is a sample text

    there are some lines


    here is another line


    this is the last line.
    root@bob:/files#

The decryption report on BOBBY's machine shows ALICE's key in the list,
because BOBBY has ALICE's key on his list.


