
The days of remembering long, complex lists of passwords, maintaining our favorite password manager, or keeping a secret notebook with all of our credentials may be behind us within the next handful of years. 

From a security perspective, we are quite simply not interested in having users manage their own passwords - when credentials are created in bulk, the tendency is to follow a template system. While this allows for ease-of-creation for IT professionals, this opens the doors to spraying attacks, birthday vulnerabilities, and a host of other complications. 

![[00tsksIfrizD7YyMuljCu12-3.webp]]

While one solution for this would be the implementation and use of a password randomizer, this still leaves us with the prescient issue of end users having to manage their own passwords after they are generated. For MSP's, this can lead to problems beyond our ability to anticipate.  For instance, complex passwords with strict requirements can be circumnavigated during mandatory resets - if hardened credentials are replaced with one that could be easily brute-forced, for instance, we end up creating problems that could be otherwise avoided. 

Thankfully, there is a solution that we can look towards that could remedy all of these issues and more. To solve these problems, legendary open industry association FIDO (Fast Identity Online) has created passkeys, which are an entirely different kind of authentication technology. 

![[Gemini_Generated_Image_8yguvg8yguvg8ygu.png]]

Kim Key, writing for PC Mag, illustrates perfectly the effect that passkeys are already having on not only on cybersecurity, but on the wider Information Technology industry:

"Passkeys have plenty of benefits; for example, they cannot be guessed or shared. Also, passkeys resist some phishing attempts because they're unique to the sites they're created for, so they won't work on fraudulent lookalikes. Most importantly, in the age of [near-constant data breaches](https://www.pcmag.com/how-to/how-could-a-data-breach-affect-me), your passkeys cannot be stolen by hacking into a company's server or database, making the stolen data far less valuable to criminals."

![[Gemini_Generated_Image_t3owkht3owkht3ow.png]]

## What Actually Is a Passkey? 

Passkeys essentially utilize asymmetric encryption in order to safeguard login information and authenticate user identity. What does that mean for us? 

For every set of login credentials, a public and private key are generated - we can use the example of a padlock with two keys, with one which stays permanently slotted into the lock. Our private key travels with us, and can be used to completely open the lock whenever we should need to access whatever resource is being secured. 

Asymmetric encryption follows this principle almost exactly, with the only major difference being that cryptographic controls exist purely in cyberspace. Because everything is virtual, this leads us to wonder how exactly unique keys are generated, and what they look like. 

A great example for us to look at would be something like hashing, which is the process by which data is irreversibly scrambled. Take a look at the image below, in which a series of phrases are scrambled by a hash function.

![[Cryptographic_Hash_Function.svg]]

In the case of hashing, we could generate hashes in order to verify the legitimacy of files that are sent to us - we could generate a hash for a file, then verify it against the file's original checksum. Provided that those values match, we can verify that the file has not been altered in transit. 

In the case of asymmetric keys, we can more or less accomplish similar functionality. Instead of creating two values which match, our keys are combined and hashed behind the scenes in order to verify our identity when logging in. The created hash is then verified against a pre-loaded checksum, and we are authenticated! 

![[Gemini_Generated_Image_ad9eclad9eclad9e.png]]
## Security Concerns

Can we be sure that passkeys will absolve us of all of our greatest security woes? Well, this is where professionals at the apex of this change must act fast in order to ensure that websites and browsers are prepared for the incoming widespread adoption of passkeys as a form of authentication. 

According to Trevor Hilligoss, security researcher and vice  president of SpyCloud Labs, in a conversation with Kim Key, observes that website owners must concentrate efforts on addressing security holes in their architecture in order to make the transition away from traditional authentication as smooth as possible. For instance, offensive security professionals and criminals have already managed to breach website security and compromise accounts by stealing users' validated browser cookies. This can be done through traditional malware being loaded in users' browsers, through session hijacking, or through cross-site-scripting (XSS) attacks. 

This threat CAN actually be mitigated on our end as users, however the knowledge that would disrupt this threat is conspicuously absent from mainstream discourse.

For those of us who are not in the know, please DO NOT click "Accept" when pop-up banners appear asking us to accept session tracking defaults and blanket cookie policies. These tokens can be stolen and used by attackers to authenticate themselves - as such, we should set our cookie configurations to being as short-term as possible. By setting those terms to being only a few minutes (as opposed to hours, days, or years), the likelihood that our sessions can be hijacked is dramatically reduced by forcing sign-ins every time we attempt to access an online service. 

![[Gemini_Generated_Image_twcxfatwcxfatwcx(1).png]]
## Sources:
**Key, Kim.** “Passwordless Authentication: What It Is and Why You Need It ASAP.” _PCMag_, 21 June 2023, [www.pcmag.com/explainers/passwordless-authentication-what-it-is-and-why-you-need-it-asap](http://www.pcmag.com/explainers/passwordless-authentication-what-it-is-and-why-you-need-it-asap).