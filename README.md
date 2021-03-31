
<p align="center">
  <img src="https://i.imgur.com/frzDe3U.png" alt="Sublime's custom image" width="300"/>
</p>

# Medial ID
Medical ID is our capstone project created during the Winter 2021 semester. Medical ID's purpose is to make it easier and secure to transfer confidential patient information to doctors offices and other medical institutions through the use of Near Field Communication (NFC). 

Users will simply go near a Raspberry Pi and activate NFC to have their information quickly and securely collected rather than having to film out a large form.

# Technologies Used
To demonstrate sufficient knowledge in Computer Science and Information Technology, this project utilized many technologies and programming languages. Below is a list of all technologies and programming languages used.

 - **Front End Development**: HTML, CSS, JavaScript, Bootstrap, React
 - **Back End Development**: PHP
 - **Mobile Application Development**: iOS (Swift) and Android (Java)
  - **Databases**: SQL (mySQL software)
  - **Operating Systems**: Debian and Raspbian (Linux distributions)
  - **Other Programming Languages**: Python
  - **Other Knowledge**: Deploying a virtual private server, running a web server behind CloudFlare, configuring a [LAMP](https://www.ibm.com/cloud/learn/lamp-stack-explained) stack,AES encryption and good security practices

# Mobile Applications

A large part of this project centers around mobile applications for the iOS and Android platforms. The purpose of the applications is to securely collect the users' information and store it securely. 

Users are able to fill out basic personal information such as name, gender, address, phone number, etc. as well as other items such as marital status, weight, height, ethnicity and insurance information. In addition to this, users can fill out emergency contact information and enter as much information as needed about their medical conditions and any injury information.

Because all user information is housed in the mobile applications, we designed the apps in such a way that maximum security is enforced. We go over this more in the next section below. 


# Mobile Application Security

When users save their information, it is saved in encrypted form in SharedPreferences (Android) with the key residing in the KeyStore or in a plist file on iOS (UserDefaults) with the key residing in the iOS keychain. In addition to this, all encrypted information is saved on a web server without the key. All encryption is performed with **AES-256-CBC**

Perhaps one of our biggest achievements is the fact that these mobile applications can successfully operate without the need of a password column in our web server's mySQL database. 

The way this works is that when a user first registers, we take the 
**SHA512** of their password and use that as the key to encrypt the email. From here, this encrypted email string is used as the primary key in the database and serves as an efficient primary key at that. It is not possible for another user to  generate the same string unless the email and password are the same. 

![Flow chart of login and register](https://i.imgur.com/huzJ0To.jpg)

Below is some psuedocode of the register functionality:

    function register(email, password){
	    pass_hash = sha512(password);
	    //AES.encrypt(string, key);
	    enc_email = AES.encrypt(email, pass_hash);
	    MedicalID.register(enc_email); 
	} 


When a user wants to login, the process is fairly similar. First the entered password is hashed with **SHA512** and is used as the key to encrypt the email. From there an API request is sent that fetches all of the users encrypted information and restores them to the application. From there, the applications attempts to decrypt all of the information that was fetched by utilizing the same sha512-hashed password used to perform the login. This should be the right key as the only way to fetch the correct record was through having a correctly encrypted email that relies on the user's password. 

**In a highly unlikely event where another users' encrypted information is returned by the server, it will be completely useless as such information could never be decrypted without the right key.** 

There is no way for anyone to decrypt the information without the right password. **Furthermore, in the worst case scenario where the database is breached, all of the encrypted data will be useless to a hacker.  The beauty of using an implementation where we don't even store the password in any shape or form is that hackers cannot try to crack a user's password through iterative brute-force or through rainbow tables.**

Below is a sample row of what our mySQL database looks like:
|email|MID_Name|MID_Birthday|
|--|--|--|
|30AE621F292899FFD791DE2F8F8A3DE6AEFC83BA047DE5388CDD6D1238546FB9|E5D3A942AA1647F8DD613A9164D16865:21EA1E2164C60BD0|5E166387DD674F6FAAAB791BC6891F05:7F9CB5C60D6E4B9C|

Aside from the email, all other rows are in the following format :
> `encryptedString:randomIV`


# HIPAA compliance

It is important for any application that deals with sensitive medical information to be HIPAA compliant. Below are some items we implemented to get towards HIPAA complaince:

 - All user information is encrypted with **AES-256-CBC**
 - All username/password combinations guaranteed to be unique.
 - Strong password requirements 
 - Preventing use of common passwords like 12345
 - Web server forces HTTPS and uses [HSTS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)  through CloudFlare
 - Records maintained of when information is accessed.
 - Records of last activity for each user.
 - A server daemon to delete accounts of users with months of inactivity.
 - Secondary authentication on apps through 2nd password or biometrics.


 

# NFC Setup



The Near Field Communication (NFC) functionality was an integral part of this project and could be considered the most difficult part of the project. Our choice for an NFC reader was the PN-532 NFC module. This module is able to write to NFC cards as well as NFC tags in various formats.

One major setback that we came across is that it would not be easy to implement proper functionality with iOS devices. The reason for this is Apple does not provide an API for Host Card Emulation (HCE). Typically, NFC readers including the ones at many retail stores can only detect compatible tags and cards. For a phone to be recognized, it needs to pretend to be a NFC tag or card which is the essence of HCE. It is possible to utilize HCE with Android devices, but not with Apple devices. The functionality exists, but no API is made available for iOS. One speculative reason for this is that Apple does not want 3rd party payment applications competing with Apple Pay. HCE functionality exists in iOS and is specifically utilized by Apple Pay. Because Android has an API for HCE, this has allowed for other payment applications such as Samsung Pay and LG pay. 

**Because of this hurdle with iOS, we developed an innovative and reliable workaround that is compatible with both iOS and Android platforms. Placed on top of the PN-532 module is a small NFC tag that is what users actually write to.** (Note that iOS supports writing to actual tags, not not simulating them which is what HCE would do.) Once users write to the tag, the PN-532 reads the NFC tag that's on top of it and clears it right after to prevent anyone else from attempting to read from it. In addition to the PN-532 NFC module, we have an SR-04 ultrasonic distance sensor that is actively used with the PN-532. When the SR-04 detects that an object (a phone) is nearby, it temporarily disables reading for the PN-532 so that there is no interference when a user tries to write to the NFC tag that is on top of the PN-532. Without this feature, the tag would not be writeable. As soon as the SR-04 sees that there is no longer an object in front of it, it will read the tag and clear it right after. 

Both the iOS application and Android application encode the NFC tags with NFC Data Exchange Format (NDEF) format and creates a single record with the string `[[encryptedEmail:sha512OfPassword]]`. The `encryptedEmail` portion  is the user's AES-encrypted email address that serves as the primary key in the database, and the sha512OfPassword portion is the user's password hashed with SHA-512 that serves as the key for AES encryption and decryption. Below is an example of a NDEF record that the mobile application try to write to the NFC tag:

    [[30AE621F292899FFD791DE2F8F8A3DE6AEFC83BA047DE5388CDD6D1238546FB9:049FE98190302258F92FE7091AEC6B2C3EE7943CF004D2E69ADDB1BDF008A104B321988D910B3A05DE1D861ECCC600ED7D3E631B02A5829A4B29728C7B57339C]]


This is the only time that the user's hashed password ever leaves their smartphone as it is needed for the raspberry pi to decrypt their medical information after pulling it without the encrypted email. Although SHA512 is a strong hashing algorithm, the raspberry pi will never save it for security purposes and as mentioned previously, the data on the NFC tag is quickly wiped after it is successfully read.

With a successful scan, our GUI application would greet the user and proceed to save securely save their information locally on the raspberry pi for the medical staff to look at and/or export. They are considered to be checked in at the medical institution. Once a user's smartphone is placed near the NFC module and it writes to the NFC tag, the rest of process of takes less than a few seconds.


<p align="center">
  <img src="https://i.imgur.com/dMFJNk5.jpg" alt="PN532" width="500"/>
  
<p style="font-size: 6px !important;">Pictured to the left is an PN-532 module. To the right is an SR-04 ultrasonic sensor.</p>

# Raspberry Pi Web Server
To be done later...

# Ways To Improve and Future Ideas 

We believe that we have made some great achievements with this project which includes NFC writing compatibility with iOS devices as well as writing mobile applications for iOS and Android and successfully publishing them to the app store. Nevertheless, there were some more features that we would have liked to implement had we not had any time constraints as well as an ongoing pandemic.

Below are some of the features and/or ideas:
* Disk encryption for the raspberry pi
* Allowing medical organizations to customize the information that the users would need to fill out.
* A two-factor-authentication (2FA) feature with Google Authenticator or through SMS verification with the Twilio API.
* SMS  notifications upon new login activity.
